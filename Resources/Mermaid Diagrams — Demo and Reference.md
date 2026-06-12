# Mermaid Diagrams — Demo & Reference

Open this file with **Markdown Preview Enhanced** (`Cmd+Shift+V`) to see all diagrams rendered.

---

## 1. Flowchart
The most common type. Great for decision trees, processes, and workflows.

```mermaid
flowchart TD
    A[New Thought] --> B{Is it actionable?}
    B -->|Yes| C{Belongs to a project?}
    B -->|No| D{Worth keeping?}
    C -->|Yes| E[Add to Projects/]
    C -->|No| F[New project or Areas/]
    D -->|Yes — idea| G[Write permanent note in Notes/]
    D -->|Yes — reference| H[File in Resources/]
    D -->|Not sure| I[Archive/]
```

---

## 2. Sequence Diagram
Shows interactions between people or systems over time. Great for documenting workflows or API flows.

```mermaid
sequenceDiagram
    participant Phone
    participant GitHub
    participant Vault
    participant Me

    Me->>Phone: Capture a thought
    Phone->>GitHub: iOS Shortcut POST /contents/Inbox/note.md
    GitHub->>Vault: File appears in Inbox/
    Me->>Vault: Weekly processing session
    Vault->>Me: Processed, linked, organized
```

---

## 3. Mind Map
Good for brainstorming and showing how topics branch out from a central idea.

```mermaid
mindmap
  root((Second Brain))
    Capture
      Inbox
      Daily Note
      Paper Notebook
    Organize
      Projects
      Areas
      Resources
      Notes
    Connect
      Wikilinks
      Tags
      Backlinks
    Automate
      Git sync
      iOS Shortcuts
      AI triage
```

---

## 4. Timeline
For project planning, historical notes, or any sequence of dated events.

```mermaid
timeline
    title Vault Setup — June 2026
    June 12 : Installed Foam & extensions
            : Configured workspace settings
            : Built PARA + Zettelkasten structure
            : Created templates
            : Documented methodology
    July 2026 : Build git auto-commit
              : Build iOS Shortcut → Inbox
    Q3 2026   : Explore AI inbox triage
              : Semantic link suggestions
```

---

## 5. Gantt Chart
Project planning with durations and dependencies.

```mermaid
gantt
    title Automation Roadmap
    dateFormat  YYYY-MM-DD
    section Core
    Git auto-commit         :done,    a1, 2026-06-12, 1d
    iOS Shortcut to Inbox   :active,  a2, 2026-06-13, 3d
    OCR paper capture       :         a3, after a2,   5d
    section Intelligence
    AI Inbox triage         :         b1, 2026-08-01, 14d
    Semantic link suggest   :         b2, after b1,   21d
    Weekly digest script    :         b3, 2026-09-01, 7d
```

---

## 6. Entity Relationship Diagram
Good for documenting data models or how concepts relate to each other.

```mermaid
erDiagram
    PROJECT ||--o{ MEETING-NOTE : "has"
    PROJECT ||--o{ PERMANENT-NOTE : "links to"
    AREA    ||--o{ PROJECT : "contains"
    RESOURCE ||--o{ PERMANENT-NOTE : "inspires"
    DAILY-NOTE ||--o{ INBOX-ITEM : "captures"
    INBOX-ITEM }o--|| PROJECT : "becomes"
    INBOX-ITEM }o--|| PERMANENT-NOTE : "becomes"
    INBOX-ITEM }o--|| RESOURCE : "becomes"
```

---

## 7. Git Graph
Visualize a git branching history — handy for documenting dev workflows.

```mermaid
gitGraph
   commit id: "Initial vault setup"
   commit id: "Add folder structure"
   commit id: "Add templates"
   branch automations
   commit id: "Git auto-commit script"
   commit id: "iOS Shortcut v1"
   checkout main
   merge automations id: "Merge automation tools"
   commit id: "Add AI triage"
```

---

## Quick Reference

| Diagram Type | Best For |
|---|---|
| `flowchart` | Decision trees, processes, workflows |
| `sequenceDiagram` | Interactions between people/systems over time |
| `mindmap` | Brainstorming, topic exploration |
| `timeline` | Event history, project milestones |
| `gantt` | Project planning with dates and dependencies |
| `erDiagram` | Relationships between concepts or data models |
| `gitGraph` | Branching and merge history |

Full docs: [mermaid.js.org](https://mermaid.js.org/intro/)
