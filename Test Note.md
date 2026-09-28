# Test Note — Style Preview

This is a regular paragraph of text. It gives a sense of how body copy feels at this font size and line height. Long enough to see how lines wrap in the editor when the content exceeds a single line width.

---

## Headings

# H1 — Main Title
## H2 — Section
### H3 — Subsection
#### H4 — Detail

---

## Lists

### Unordered
- First item
- Second item
  - Nested item
  - Another nested item
- Third item

### Ordered
1. Step one
2. Step two
3. Step three

### Task List
- [x] Completed task
- [x] Another done item
- [ ] Still to do
- [ ] Also pending

---

## Blockquote

> "The mind is not a vessel to be filled, but a fire to be kindled."
> — Plutarch

---

## Code

Inline code: `const note = new Note("ideas");`

```javascript
function openDailyNote(date) {
    const filename = formatDate(date, "YYYY-MM-DD");
    return vault.open(`Journal/${filename}.md`);
}
```

---

## Table

| System    | Best For              | Weakness            |
|-----------|-----------------------|---------------------|
| PARA      | Actionable projects   | Ideas without home  |
| Zettelkasten | Connected thinking | No built-in structure |
| Hybrid    | Everything            | Requires discipline |

---

## Wikilinks & Tags

This note relates to [[Foam Power User Setup Plan]] and the overall note-taking methodology.

Tags: #setup #reference #foam

---

## Emphasis & Formatting

**Bold text** for key terms. *Italic* for emphasis. ~~Strikethrough~~ for outdated info. `inline code` for technical terms.

A sentence with a [hyperlink](https://foambubble.github.io/foam/) inline.