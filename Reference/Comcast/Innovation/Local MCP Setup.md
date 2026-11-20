
---
tags:
  - setup-guide
  - qdrant
  - ollama
  - mcp
  - copilot
---

# Local Hybrid-RAG + MCP Setup Walkthrough

## 1. Local Embeddings (Ollama)
Install Ollama from [ollama.com](https://ollama.com), then pull the embedding model:
```bash
ollama pull nomic-embed-text
```

---

## 2. Project Directory & Qdrant Container
Create the workspace and start Qdrant:
```bash
mkdir -p ~/copilot-rag && cd ~/copilot-rag
```

Save as `~/copilot-rag/docker-compose.yml`:
```yaml
services:
  qdrant:
    image: qdrant/qdrant:latest
    container_name: copilot-qdrant
    restart: unless-stopped
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - ./qdrant_storage:/qdrant/storage
```

Start the container (Dashboard available at `http://localhost:6333/dashboard`):
```bash
docker compose up -d
```

---

## 3. Python Environment
```bash
python -m venv venv
source venv/bin/activate  # Windows PowerShell: .\venv\Scripts\Activate.ps1
pip install qdrant-client fastembed llama-index-core llama-index-embeddings-ollama tree-sitter-language-pack fastmcp watchdog
```

---

## 4. Incremental Indexer + File Watcher (`index_kb.py`)
Save as `~/copilot-rag/index_kb.py` and update `KB_FOLDER`:
```python
import os
import json
import time
import hashlib
from pathlib import Path
from qdrant_client import QdrantClient, models
from llama_index.core import Document
from llama_index.core.node_parser import MarkdownNodeParser, CodeSplitter, SentenceSplitter
from llama_index.embeddings.ollama import OllamaEmbedding
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

KB_FOLDER = "/path/to/your/knowledge_base"  # <-- UPDATE THIS PATH
COLLECTION = "knowledge_base"
MANIFEST_PATH = Path("./index_manifest.json")
IGNORE_DIRS = {".git", "node_modules", "__pycache__", ".venv", "dist", "build"}
CODE_EXTS = {".py": "python", ".ts": "typescript", ".js": "javascript", ".go": "go", ".java": "java", ".cs": "c_sharp"}
DATA_EXTS = {".json", ".csv", ".yaml", ".yml", ".xml", ".sql"}

client = QdrantClient(url="http://localhost:6333")
embed_model = OllamaEmbedding(model_name="nomic-embed-text")
md_parser = MarkdownNodeParser()
text_parser = SentenceSplitter(chunk_size=512, chunk_overlap=50)

def init_collection():
    if not client.collection_exists(COLLECTION):
        client.create_collection(
            collection_name=COLLECTION,
            vectors_config={"dense": models.VectorParams(size=768, distance=models.Distance.COSINE)},
            sparse_vectors_config={"bm25": models.SparseVectorParams(modifier=models.Modifier.IDF)}
        )

def file_hash(path: Path) -> str:
    return hashlib.sha256(path.read_bytes()).hexdigest()

def chunk_file(filepath: Path, rel_path: str):
    ext = filepath.suffix.lower()
    content = filepath.read_text(encoding="utf-8", errors="ignore")
    if not content.strip():
        return []

    if ext in DATA_EXTS:
        lines = content.splitlines()
        snippet = "\n".join(lines[:40])
        if len(lines) > 40:
            snippet += f"\n... [Truncated: {len(lines) - 40} more lines]"
        return [(f"File: {rel_path} (Sample Data)\n{snippet}", "sample_data")]

    doc = Document(text=content, metadata={"source": rel_path})
    if ext in {".md", ".mdx"}:
        nodes, role = md_parser.get_nodes_from_documents([doc]), "documentation"
    elif ext in CODE_EXTS:
        try:
            splitter = CodeSplitter(language=CODE_EXTS[ext], chunk_lines=60, chunk_lines_overlap=10)
            nodes = splitter.get_nodes_from_documents([doc])
        except Exception:
            nodes = text_parser.get_nodes_from_documents([doc])
        role = "code"
    else:
        nodes, role = text_parser.get_nodes_from_documents([doc]), "general_text"

    return [(f"File: {rel_path}\n{n.get_content()}", role) for n in nodes]

def delete_file_points(rel_path: str):
    client.delete(
        collection_name=COLLECTION,
        points_selector=models.FilterSelector(
            filter=models.Filter(must=[models.FieldCondition(key="source", match=models.MatchValue(value=rel_path))])
        )
    )

def index_single_file(filepath: Path, rel_path: str):
    delete_file_points(rel_path)
    chunks = chunk_file(filepath, rel_path)
    if not chunks:
        return

    points = []
    for idx, (text, role) in enumerate(chunks):
        point_id = hashlib.md5(f"{rel_path}:{idx}".encode()).hexdigest()
        dense_vec = embed_model.get_text_embedding(text)
        points.append(models.PointStruct(
            id=point_id,
            vector={
                "dense": dense_vec,
                "bm25": models.Document(text=text, model="Qdrant/bm25")
            },
            payload={"text": text, "source": rel_path, "role": role}
        ))
    client.upsert(collection_name=COLLECTION, points=points)
    print(f"Indexed: {rel_path} ({len(points)} chunks)")

def sync_all():
    init_collection()
    manifest = json.loads(MANIFEST_PATH.read_text()) if MANIFEST_PATH.exists() else {}
    current_files = {}

    for root, dirs, files in os.walk(KB_FOLDER):
        dirs[:] = [d for d in dirs if d not in IGNORE_DIRS]
        for f in files:
            fp = Path(root) / f
            rel = str(fp.relative_to(KB_FOLDER))
            current_files[rel] = fp

    for removed in set(manifest.keys()) - set(current_files.keys()):
        delete_file_points(removed)
        del manifest[removed]
        print(f"Removed: {removed}")

    for rel, fp in current_files.items():
        try:
            h = file_hash(fp)
            if manifest.get(rel) != h:
                index_single_file(fp, rel)
                manifest[rel] = h
        except Exception as e:
            print(f"Error {rel}: {e}")

    MANIFEST_PATH.write_text(json.dumps(manifest, indent=2))
    print("Sync complete.")

class KBWatcher(FileSystemEventHandler):
    def on_any_event(self, event):
        if event.is_directory:
            return
        if any(part in IGNORE_DIRS for part in Path(event.src_path).parts):
            return
        sync_all()

if __name__ == "__main__":
    sync_all()
    observer = Observer()
    observer.schedule(KBWatcher(), KB_FOLDER, recursive=True)
    observer.start()
    print(f"Watching {KB_FOLDER} for changes...")
    try:
        while True:
            time.sleep(2)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()
```

Run initial sync + keep watcher active:
```bash
python index_kb.py
```

---

## 5. MCP Server (`mcp_server.py`)
Save as `~/copilot-rag/mcp_server.py`:
```python
from fastmcp import FastMCP
from qdrant_client import QdrantClient, models
from llama_index.embeddings.ollama import OllamaEmbedding

mcp = FastMCP("LocalFeatureKB")
client = QdrantClient(url="http://localhost:6333")
embed_model = OllamaEmbedding(model_name="nomic-embed-text")
COLLECTION = "knowledge_base"

@mcp.tool()
def search_knowledge_base(query: str, role_filter: str = "") -> str:
    """
    Hybrid semantic + keyword search over local feature specs, code, runbooks, and sample data.
    Args:
        query: Search concepts, function names, or exact error strings.
        role_filter: Optional ('documentation', 'code', 'sample_data', 'general_text').
    """
    dense_vec = embed_model.get_text_embedding(query)
    q_filter = None
    if role_filter in {"documentation", "code", "sample_data", "general_text"}:
        q_filter = models.Filter(must=[models.FieldCondition(key="role", match=models.MatchValue(value=role_filter))])

    results = client.query_points(
        collection_name=COLLECTION,
        prefetch=[
            models.Prefetch(query=dense_vec, using="dense", limit=10),
            models.Prefetch(query=models.Document(text=query, model="Qdrant/bm25"), using="bm25", limit=10),
        ],
        query=models.FusionQuery(fusion=models.Fusion.RRF),
        query_filter=q_filter,
        limit=6
    ).points

    if not results:
        return "No relevant context found."

    return "\n\n".join(
        f"--- SOURCE: {p.payload['source']} [{p.payload['role']}] ---\n{p.payload['text']}"
        for p in results
    )

@mcp.tool()
def get_kb_stats() -> str:
    """Audit total indexed chunks and collection status."""
    info = client.get_collection(COLLECTION)
    return f"Status: {info.status} | Total Indexed Chunks: {info.points_count}"

if __name__ == "__main__":
    mcp.run()
```

---

## 6. VS Code MCP Configuration
Add to `.vscode/mcp.json` in your workspace (update `YOUR_USERNAME`; on Windows use `venv\\Scripts\\python.exe`):
```json
{
  "servers": {
    "local-feature-kb": {
      "type": "stdio",
      "command": "/Users/YOUR_USERNAME/copilot-rag/venv/bin/python",
      "args": [
        "/Users/YOUR_USERNAME/copilot-rag/mcp_server.py"
      ]
    }
  }
}
```

---

## 7. Copilot Agent Instructions
Save as `.github/copilot-instructions.md` in your code repository:
```markdown
# Feature Development & Troubleshooting Rules
- Always call `search_knowledge_base` before planning a feature, changing architecture, or debugging an error.
- Query `role_filter="documentation"` first for specs/requirements, then `role_filter="code"` for existing patterns.
- Only query `role_filter="sample_data"` when building unit tests, mocks, or payload parsers.
- Cite the `SOURCE` file paths returned by the tool when explaining architectural decisions.
```
