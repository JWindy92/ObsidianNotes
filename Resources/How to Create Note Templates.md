---
title: How to Create Note Templates
date: 2026-06-12
type: note
tags: [reference, foam, templates]
---

# How to Create Note Templates

Templates in Foam are plain markdown files stored in `.foam/templates/`. When you create a new note from a template, Foam copies the file and substitutes any variables in it automatically.

---

## Using Templates

**Create a note from a template:**
1. Open the Command Palette (`Cmd+Shift+P`)
2. Run `Foam: Create New Note From Template`
3. Pick a template and give the note a title

**Open today's daily note:**
- Command Palette → `Foam: Open Daily Note`
- Or assign a keybind in your keyboard shortcuts

---

## Template Variables

Variables are placeholders that get replaced when the note is created.

### Foam Variables
| Variable | Output |
|---|---|
| `$FOAM_TITLE` | The title you typed when creating the note |
| `$FOAM_DATE` | Today's date (`YYYY-MM-DD`) |
| `$FOAM_DATE_YEAR` | Current year |
| `$FOAM_DATE_MONTH` | Current month (numeric) |
| `$FOAM_DATE_MONTH_NAME` | Current month name |
| `$FOAM_DATE_DATE` | Day of the month |

### VS Code Native Variables
| Variable | Output |
|---|---|
| `$TM_FILENAME` | Full filename including extension |
| `$TM_FILENAME_BASE` | Filename without extension |
| `$CURRENT_YEAR` | Current year |
| `$CURRENT_MONTH` | Current month (01–12) |
| `$CURRENT_DATE` | Current day (01–31) |
| `$CURRENT_HOUR` | Current hour (24h) |
| `$CURRENT_MINUTE` | Current minute |

---

## Frontmatter

The block at the top between `---` markers is YAML frontmatter. Both Foam and Obsidian read it.

```markdown
---
title: My Note
date: 2026-06-12
type: project
status: active
tags: [work, planning]
---
```

Common fields to use:
- `title` — display name
- `date` — creation date
- `type` — note type (`daily`, `project`, `meeting`, `literature`, `note`)
- `status` — for projects: `active`, `paused`, `complete`
- `tags` — list of tags for filtering

---

## Creating Your Own Template

1. Create a new `.md` file in `.foam/templates/`
2. Add a frontmatter block with any fields you want pre-filled
3. Write out the sections and headings you always want
4. Drop variables wherever dynamic content should go
5. Save — it immediately appears in the template picker

### Example: a minimal template
```markdown
---
title: $FOAM_TITLE
date: $FOAM_DATE
tags: []
---

# $FOAM_TITLE

```

### Example: a template with a prompt
```markdown
---
title: $FOAM_TITLE
date: $FOAM_DATE
type: idea
---

# $FOAM_TITLE

## The Idea
<!-- Describe it in one sentence -->

## Why It Matters

## Next Step
- [ ] 
```

---

## Templates in This Vault

| Template | File | Use For |
|---|---|---|
| Daily Note | `daily-note.md` | Opened automatically each day |
| Project Note | `project-note.md` | New project index |
| Meeting Note | `meeting-note.md` | Any meeting worth capturing |
| Literature Note | `literature-note.md` | Reading notes from articles/books |
| Permanent Note | `permanent-note.md` | Atomic ideas in `Notes/` |

---

## Tips

- Keep permanent note templates **minimal** — the point is to write, not fill out forms
- Add a `<!-- comment -->` as a prompt inside sections so you know what to write there
- The daily note template is used automatically — Foam opens it without asking for a title
- You can have as many templates as you like — make one whenever you find yourself structuring notes the same way repeatedly
