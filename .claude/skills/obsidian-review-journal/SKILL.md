---
name: obsidian-review-journal
description: >
  Reviews and linguistically improves journal files in the Obsidian vault.
  Detects language in each section, corrects grammar/spelling/style accordingly
  (Polish or English), replaces the original text in-place, and marks the file
  as "claude-reviewed" so it is never processed twice.
  Use this skill whenever the user says "przejrzyj dziennik", "popraw journal",
  "review my journal", "улучши журнал", "review journal entries", "popraw wpisy",
  "sprawdź journal", "przejrzyj notatki", "review my notes", or asks to
  linguistically improve, proofread, or clean up daily notes in Obsidian.
  Also trigger when the user asks to "mark journal as reviewed" or wants to
  batch-process unreviewed vault journal files.
---

# obsidian-review-journal

Reviews journal files in the Obsidian vault, improves language (Polish or English — auto-detected), saves the improved version, and marks the file as `#claude-reviewed`.

**Vault:** `/Users/lsosnicki/vault`
**Journal dir:** `/Users/lsosnicki/vault/journal/`

---

## Step 1 — Find unprocessed files

Use `Filesystem:list_directory` on `/Users/lsosnicki/vault/journal/`.

For each `.md` file:
- Read it (`Filesystem:read_text_file`)
- Check whether it contains the tag `#claude-reviewed` **anywhere in the file** (in content or frontmatter)
- **Skip the file if the tag is present**
- If the filename matches the `YYYY-MM-DD.md` pattern, parse the date and **skip the file if the date is in the future** (after today)

Build a list of files to process (those without `#claude-reviewed`).

If the list is empty → stop and inform the user:
> `✅ All journal files are already marked as claude-reviewed. Nothing new to process.`

---

## Step 2 — Process each file

For each file in the list:

### 2a — Split into sections

Split the file content into sections by Markdown headings (`##`, `###`). Treat each section as an independent unit for review.

Sections to **skip** (do not modify their content):
- `📌 My Day` — task checkboxes, leave untouched
- Empty sections or sections containing only `---` separators
- YAML frontmatter (`---` at the top of the file)
- Inline tags (`#tag`) and wikilinks (`[[...]]`) — preserve as-is within text

Sections to **process** (improve language):
- `💡 Reflections` — skip if empty
- `✍️ Summary` — skip if empty (will be populated in Step 5)
- `🧩 Notes` — all narrative text and bullet lists
- `🌊 Meetings` — descriptive text under meeting subheadings (**not** the meeting titles or times in `### Title (HH:MM–HH:MM)` format)
- Any other sections containing narrative text

> **Note:** Section names may vary between files (e.g. `Meetings` without emoji, `Notes` vs `Other Notes`). Match sections by keyword, not exact string.

### 2b — Detect section language

For each section to process:
- **Polish**: sentences contain Polish diacritics (ą, ę, ó, ś, ź, ż, ć, ń, ł) or Polish function words (z, na, do, że, jest, nie)
- **English**: sentences contain English function words (the, is, are, and, with, for)
- Mixed section → use the dominant language
- Section too short to assess (1–2 words) → skip

### 2c — Correct the text

For each section with a detected language, apply the following corrections:

**Polish:**
- Restore missing Polish diacritics (e.g. `ze` → `że`, `cos` → `coś`, `sie` → `się`)
- Fix typos and spelling errors
- Fix grammar errors (declension, syntax)
- Fix punctuation
- Improve style: remove repetitions, awkward phrasing
- Preserve the original register (informal journal → stays informal)
- Preserve the author's meaning and intent — do not rephrase radically

**English:**
- Fix typos and spelling errors
- Fix grammar (verb forms, articles, number)
- Fix punctuation
- Light style improvement without changing tone

**Never change:**
- Proper nouns, names, company abbreviations (GoKart, InPost, JSM-XXXXX)
- Markdown formatting (bold, italic, headings)
- Wikilinks `[[...]]`
- Tags `#...`
- Times, dates, numbers

### 2d — Replace text in file

For each changed section: replace the original text with the corrected text, leaving the rest of the file unchanged.

Use `Filesystem:write_file` to save the updated version of the entire file.

---

## Step 3 — Mark file as `#claude-reviewed`

After saving corrections, add the `#claude-reviewed` tag to the file. The tag always goes at the **top of the file**.

### How to add the tag

**If the file has YAML frontmatter** (starts with `---`):

Add `claude-reviewed` to the `tags:` section in frontmatter:

```yaml
---
tags:
  - existing-tag
  - claude-reviewed
---
```

If `tags:` doesn't exist, add it:

```yaml
---
tags:
  - claude-reviewed
---
```

**If the file has NO frontmatter** (typical journal file — e.g. `journal/YYYY-MM-DD.md`):

Add YAML frontmatter with the tag at the very top of the file:

```yaml
---
tags:
  - claude-reviewed
---

## 📌 My Day
...
```

Save the file with the tag using `Filesystem:write_file`.

---

## Step 4 — Write day summary into `✍️ Summary` section

After applying language corrections, synthesize the content of the entire file into a concise summary and write it into the `✍️ Summary` section (or `Summary` / `Closing line` — match by keyword).

### What to include

Read all populated sections of the file — `📌 My Day`, `🌊 Meetings`, `🧩 Notes`, `💡 Reflections` — and extract the key information from that day.

Write the summary as a **bullet list of key takeaways**, for example:

```markdown

- Rollout 200k brief with Pepco held — workshop tomorrow 15.04
- Dynatrace observability dashboards status reviewed with Tczerko
- CAG upgrade: throttling issues identified, next steps defined (workshop + problem definition)
- JSM-426665 PagerDuty access request for Paweł Bonikowski pending approval
- Ticket JSM-409079 (Merchant Portal password reset) still unresolved
```

### Rules

- **3–7 bullet points** — only the most important facts, decisions, and open items
- Use the **dominant language of the file** (Polish or English)
- Be factual and brief — no filler phrases like "Today was a busy day"
- Include: decisions made, key outcomes from meetings, open action items, important tickets/blockers
- Omit: routine recurring meetings with no notable outcome, empty sections
- If `✍️ Summary` already has content → **preserve it**, do not overwrite

### Positioning the Summary section

After generating (or preserving) the summary content, **move the entire `✍️ Summary` section to be the first section in the file**, immediately after the YAML frontmatter closing `---` and before all other sections (`📌 My Day`, `🌊 Meetings`, etc.). Add a blank line after the last bullet of the summary text.

**Important:** Place `## ✍️ Summary` on the very next line after the closing `---` — **no blank line** between frontmatter and the Summary heading. There **is** a blank line between the `## ✍️ Summary` heading and the first bullet.

Example final file structure:
```markdown
---
tags:
  - claude-reviewed
---
## ✍️ Summary

- Key takeaway 1
- Key takeaway 2

## 📌 My Day
...

## 🌊 Meetings
...
```

Use `Filesystem:write_file` to save the file with the summary filled in and repositioned.

---

## Step 6 — Summary

After processing all files, display in chat:

```
✅ Journal review complete

📋 Processed files: N
  - 2026-04-12.md — improved N sections
  - 2026-04-13.md — no changes (sections empty)

⏭️ Skipped (already reviewed): N files
```

If any file had notable corrections, optionally show 1–2 before/after examples for transparency.

---

## Edge Cases

- **Archive files** (`journal/archive/`): skip all files in subdirectories unless the user explicitly asks to process the archive
- **Section contains only emojis or single words**: skip, nothing to correct
- **Empty file**: mark as `#claude-reviewed` without modifying content
- **Filesystem unavailable**: inform the user and exit with an error
- **Very long file (>300 lines)**: process section by section, save the full file in a single `write_file` call at the end
