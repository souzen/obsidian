---
name: obsidian-save-to-inbox
description: Saves uploaded files (PDF, DOCX, image) or web article URLs as structured Markdown summary notes to the user's Obsidian vault inbox. Trigger whenever the user wants to "add to Obsidian", "save to vault", "zapisz do vaulta", "wrzuć do Obsidian", or store any file/link in their notes system.
---

# Obsidian: Save to Inbox

Saves uploaded files or web articles as structured Markdown notes into the user's Obsidian vault inbox.

**Vault path:** `/Users/lsosnicki/vault`
**Target folder:** `/Users/lsosnicki/vault/inbox/`

---

## Inputs

Accept any of the following:
- **Uploaded file** — PDF, DOCX, image, or any document
- **URL / article link** — paste of a web address pointing to an article, blog post, documentation, etc.
- **Both** — a file + a URL together (treat as separate notes or combined, based on context)

---

## Step-by-Step Workflow

### Input: File

1. **Read the file** using the appropriate tool (pdf, docx, image reader, etc.)
2. **Extract core content**: title, author (if present), date (if present), main topics
3. **Generate note** using the Note Template below
4. **Derive filename** from the document title or content (slugified, max 60 chars)
5. **Write summary note** using Filesystem `write_file` to `/Users/lsosnicki/vault/inbox/<filename>.md`
6. **Copy original file** using Filesystem `copy_file` from its upload path to `/Users/lsosnicki/vault/inbox/<filename>.<original-extension>` — use the same base name as the summary note
7. **Confirm** to the user with both file paths

### Input: URL / Article Link

1. **Fetch the URL** using `web_fetch` to retrieve the article content
2. **Extract core content**: title, author (if present), publication date (if present), main topics
3. **Generate note** using the Note Template below — **always include the source URL**
4. **Derive filename** from the article title (slugified, max 60 chars)
5. **Write summary note** using Filesystem `write_file` to `/Users/lsosnicki/vault/inbox/<filename>.md`
6. **Confirm** to the user with the file path

---

## Note Template

The note must start with YAML frontmatter (matching the style in the vault), followed by content:

```
---
tags:
  - inbox
  - <tag2>
  - <tag3>
source: <URL if from web / filename if from file upload>
date-added: <YYYY-MM-DD>
---

# <Title>

## Summary

<3–5 sentence summary. Capture: what this is about, key arguments or findings, and why it matters.>

## Key Points

- <point 1>
- <point 2>
- <point 3>
- <point 4 — aim for 4–7 bullets total>

## Details

<Optional: longer notes, quotes, technical details worth preserving. Omit if not needed.>
```

---

## Tagging Rules

Tags go in the YAML frontmatter at the top. Always include `inbox` as the first tag. Add 2–4 additional tags based on content:

- Use lowercase, hyphenated tags: `engineering`, `product-management`, `ai`, `security`, `leadership`, `architecture`, `process`, `tools`, `research`, etc.
- For InPost/GoKart-related content: add `inpost` or `gokart` where relevant
- Keep tags specific and useful for future retrieval

---

## Filename Rules

- Derive from the title: lowercase, spaces → hyphens, remove special chars
- Max 60 characters
- No `.md` extension for the base slug — append `.md` for the summary, keep original extension for the source file copy
- Examples (summary + original file pairs):
  - `inpost-api-security-guidelines-2024.md` + `inpost-api-security-guidelines-2024.pdf`
  - `team-topologies-key-concepts.md` + `team-topologies-key-concepts.docx`

---

## Writing Guidelines

- **Summary**: 3–5 sentences. What is this? What does it argue or show? Why does it matter?
- **Key Points**: 4–7 bullets. Each = one idea. Start with a noun or verb. No full paragraphs.
- **Source link**: Always present — URL for web articles, original filename for uploads
- **Language**: Match the source language (Polish doc → Polish summary; English article → English summary). If mixed, default to English.
- **Tone**: Neutral and informative. Not opinionated unless the source is.

---

## Edge Cases

- **Paywalled or inaccessible URL**: Inform the user the page couldn't be fetched. Ask them to paste the text directly.
- **Very short content** (< 3 paragraphs): Still create the note, but skip the Details section and note it was brief.
- **No clear title**: Derive a descriptive filename from the main topic.
- **Multiple files or links at once**: Create one note per item, saving each to inbox separately.
- **Filesystem tool unavailable**: Inform the user the vault can't be accessed and offer to show the note content so they can paste it manually.
- **File copy fails**: Still save the summary note and inform the user the original file couldn't be copied (they can do it manually).

---

## Confirmation Message

After writing the files, always confirm:

```
✅ Zapisano do Obsidian:
📄 /Users/lsosnicki/vault/inbox/<filename>.md  ← summary
📎 /Users/lsosnicki/vault/inbox/<filename>.<ext>  ← original file
🏷️ Tags: #inbox #<tag1> #<tag2>
```

For URL input (no file to copy):
```
✅ Zapisano do Obsidian:
📄 /Users/lsosnicki/vault/inbox/<filename>.md
🏷️ Tags: #inbox #<tag1> #<tag2>
```
