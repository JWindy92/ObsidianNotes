---
title: Future Automations
date: 2026-06-12
type: project
status: backlog
tags: [automation, ideas, vault]
---

# Future Automations

Ideas for automating and extending the vault workflow. Organized roughly by complexity and impact. This is a living backlog — add ideas here as they come up.

---

## Core Infrastructure (Build First)

### Git Auto-Commit on Save
**What:** Every time a file is saved, automatically commit and push to GitHub.
**Why:** Eliminates the need to ever think about syncing. The vault just stays current.
**How:** A VS Code extension (`vscode-git-autocommit`) or a file watcher script using `fswatch` + a shell script that runs `git add -A && git commit -m "auto: $(date)" && git push`.
**Effort:** Low

---

### iOS Shortcut → Inbox
**What:** A single tap on the phone creates a new timestamped file in `Inbox/` via the GitHub API.
**Why:** Removes the need to open any app — works from the home screen or lock screen.
**How:** iOS Shortcut using the GitHub REST API (`POST /repos/{owner}/{repo}/contents/{path}`) to create a new `.md` file with dictated or typed content.
**Effort:** Medium
**Note:** Requires a GitHub personal access token stored in iOS Keychain.

---

### Paper → Inbox via OCR
**What:** Photograph a page of the paper notebook, extract text via OCR, drop it into `Inbox/` as a new note.
**Why:** Eliminates manual transcription from paper — the biggest current friction point.
**How:** iOS Shortcut using the built-in Live Text / Vision framework, or a Shortcut that sends an image to an OCR API (e.g. Google Vision, Tesseract). Output gets committed to `Inbox/` via the GitHub API shortcut above.
**Effort:** Medium–High

---

## Intelligence Layer (Build Later)

### AI-Assisted Inbox Triage
**What:** A script that reads all files in `Inbox/`, suggests a destination folder and relevant tags for each, and optionally auto-moves them.
**Why:** The weekly processing session is the highest-friction part of the system. AI can do most of the routing decision.
**How:** A Python or Node script that calls an LLM API (OpenAI, Claude) with the note content + folder structure as context. Output is a suggested path and tags. Could run as a CLI tool or a scheduled task.
**Effort:** Medium
**Ideas:**
- Interactive mode: show suggestion, confirm/override per note
- Auto mode: moves with high-confidence items, flags uncertain ones for review

---

### Semantic Link Suggestions
**What:** When creating or editing a note, surface existing notes that are semantically related — even if not explicitly linked.
**Why:** You can't link to notes you've forgotten exist. This surfaces forgotten connections.
**How:** Embed all notes as vectors (using a local model like `nomic-embed-text` via Ollama, or an API). On note open/save, query for nearest neighbors and display in a sidebar or status bar.
**Effort:** High
**Note:** Foam has an MCP server (`foam mcp`) that may be a foundation for this.

---

### Weekly Digest / Resurface
**What:** Every Sunday, generate a digest that includes: unprocessed Inbox count, open project next actions, and 3–5 notes you wrote a while ago that you haven't visited recently.
**Why:** Forces a review cadence without requiring discipline — the system reminds you.
**How:** A script that reads the vault, computes last-modified dates, picks "forgotten" notes by some criteria, and formats a digest into a new note or sends it via email/notification.
**Effort:** Medium

---

### Morning Brief Note
**What:** Each morning, auto-generate a note that pulls together: today's daily note (pre-created), open tasks across all Projects, and a random past note to revisit.
**Why:** Gives a structured start to the day without manually checking multiple places.
**How:** A script triggered by a cron job or iOS Shortcut that reads project notes for open `- [ ]` tasks, picks a random archived note, and writes a summary into today's `Journal/` note before you open it.
**Effort:** Medium

---

### Auto-Tagging Pipeline
**What:** When a new note is created, automatically suggest or apply tags based on content.
**Why:** Tags are only useful if consistently applied. Manual tagging is inconsistent.
**How:** LLM call with the note content + existing tag vocabulary. Could run as a git pre-commit hook or a file watcher.
**Effort:** Medium

---

## Integrations (Explore Later)

| Idea | Description |
|---|---|
| **Readwise → Resources** | Auto-import highlights from books/articles into `Resources/` as literature notes |
| **Calendar → Meeting Notes** | Pull today's calendar events and pre-create meeting note stubs |
| **GitHub Issues → Projects** | Sync open issues from a repo into a project note's task list |
| **Drafts App (iOS)** | Use Drafts as a mobile capture layer that syncs to Inbox via action scripts |
| **Obsidian Dataview** | Query notes like a database — surfaces all open tasks, all notes by tag, etc. |

---

## Notes
- Start with **git auto-commit** and **iOS Shortcut → Inbox** — these two solve the biggest daily friction points
- The AI/semantic features are high-value but depend on having enough notes to make them useful (give it 3–6 months of real usage first)
- Revisit this list during quarterly reviews of the system
