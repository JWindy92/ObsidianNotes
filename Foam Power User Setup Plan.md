# Foam Power User Setup Plan

> Created: 2026-06-12  
> Purpose: Guided setup of Foam + VS Code as a power-user Obsidian-compatible note-taking environment.

---

## Phase 1: VS Code Extensions

Install Foam plus curated companion extensions:

| Extension | Purpose |
|---|---|
| `foam.foam-vscode` | Core wikilinks, backlinks, graph, daily notes |
| `yzhang.markdown-all-in-one` | TOC, keyboard shortcuts, formatting helpers |
| `shd101wyy.markdown-preview-enhanced` | Rich preview: Mermaid diagrams, math, code |
| `mushan.vscode-paste-image` | Paste clipboard images directly into markdown |
| `streetsidesoftware.code-spell-checker` | Spell checking in markdown |
| `eamodio.gitlens` | Git history and blame on notes |

---

## Phase 2: VS Code Workspace Settings

Configure `.vscode/settings.json` for the vault:
- Wikilink autocomplete
- Daily note location and template path
- Word wrap, font, ruler
- Auto-save
- File associations

---

## Phase 3: Folder Structure

```
📁 Inbox/          ← capture fast, process later
📁 Notes/          ← permanent evergreen notes
📁 Projects/       ← active project workspaces
📁 Journal/        ← daily notes land here
📁 Resources/      ← reference material, clippings
📁 Assets/         ← images, attachments
📁 Templates/      ← note templates
📁 Archive/        ← dead projects, old notes
```

---

## Phase 4: Templates

Starter templates to create:
- `Templates/daily-note.md`
- `Templates/meeting-note.md`
- `Templates/project-note.md`
- `Templates/quick-capture.md`

---

## Phase 5: `.gitignore`

Ensure the right Obsidian and Foam config files are committed vs. ignored:
- Commit: `.obsidian/app.json`, `appearance.json`, `core-plugins.json`
- Ignore: `.obsidian/workspace.json` (machine-specific window state)
- Ignore: `.foam/` cache files if generated

---

## Open Questions (to answer before starting)

- [ ] Use Obsidian on mobile?
- [ ] Primary note domains (work, personal, dev, research)?
- [ ] Daily journaling as part of setup?
- [ ] Any existing notes to migrate in?
