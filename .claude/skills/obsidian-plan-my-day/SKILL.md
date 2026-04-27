---
name: obsidian-plan-my-day
description: "Creates a structured daily work plan in Obsidian (journal/YYYY-MM-DD.md) by combining Outlook calendar, emails, and active projects from the kanban board. Always use this skill when the user says \"plan my day\", \"what's on my agenda\", \"daily plan\", \"give me an overview of tomorrow\", \"help me prepare for tomorrow\", \"what do I have today\", \"what's tomorrow look like\", or when they want to combine calendar + inbox + projects into one actionable plan. Trigger even for loose phrasings like \"what do I have today?\" or \"sort out my day\". Output is always a .md file saved to vault/journal/."
---

# Plan My Day

Creates a structured daily plan **for work matters only** and saves it to the Obsidian vault.

> ⚠️ Scope: `work/` system only. Never pull data from `my/` — that PARA system covers personal matters and does not belong in the daily plan.

**Vault:** `/Users/lsosnicki/obsidian/vault`
**Journal:** `/Users/lsosnicki/obsidian/vault/journal/`
**Template:** `/Users/lsosnicki/obsidian/vault/work/resources/tools/obsidian/templates/daily.md`
**Projects:** `/Users/lsosnicki/obsidian/vault/work/projects/`

---

## Step 1 — Determine the date

- Default: **today**. If "tomorrow" → use tomorrow's date.
- Format: `YYYY-MM-DD` → target file: `/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`

---

## Step 2 — Check or create the journal file

Use `Filesystem:read_text_file` on the target path:

- **File exists** → load and preserve existing notes. Only fill in empty placeholders.
- **File does not exist** → continue; the template will be loaded in Step 6a.

---

## Step 3 — Load all active projects

Use `Filesystem:list_directory` on `/Users/lsosnicki/obsidian/vault/work/projects/` to get all `.md` files.

Load all of them using `Filesystem:read_multiple_files`.

Extract from each project:
- Open checkboxes `- [ ]` (tasks to do)
- Assigned people `[[@...]]`
- Frontmatter tags

If a project file does not exist — skip it, do not abort.

---

## Step 4 — Detect overlooked tasks from projects

Based on the loaded project files (Step 3), look for tasks that are easy to miss:
- `- [ ]` checkboxes buried deep in sections (not at the top of the file)
- Checkboxes with a date in their text that has already passed

Build a list of max **3–5 suggestions** — only those **not already** in the "My Day" section. Add them to "My Day" as:

```
- [ ] 📌 *[SUGGESTION]* Task title ← project: project-name
```

---

## Step 5 — Fetch calendar and emails (M365 MCP)


### Calendar
```
outlook_calendar_search
- date: the full target day
- max_results: 20
- timezone: Europe/Warsaw (CET/CEST)
```
Save: title, time (HH:MM–HH:MM), whether the meeting is recurring (series), meeting description (if present). Do **not** save attendees.

### Emails
```
outlook_email_search
- last 24h
- max_results: 10
```
Look only for emails requiring a reply or action. Ignore FYI/CC.

---

## Step 6 — Build the journal file

### 6a — Load template from vault

**Always** load the current template before building the file, and use it to structure the day:

```
Filesystem:read_text_file
→ /Users/lsosnicki/obsidian/vault/work/resources/tools/obsidian/templates/daily.md
```

- Use the loaded template as the base structure of the journal file.
- If the template contains `{{date}}` or other Obsidian variables — substitute them with the correct value (e.g. date in `YYYY-MM-DD` format).
- If the template file does not exist → abort and inform the user.

### 6b — Populate the template with data

Fill in sections with data from steps 3–5. If the journal file already existed (Step 2) — preserve the user's existing notes (Notes, Reflections, Summary sections, manual entries). Only fill in empty placeholders or sections with no content yet. Do not overwrite anything that looks like a manual entry.

### Rules for the "My Day" section (`### My Day`)

- **3–7 tasks** — choose the most important ones, do not copy everything
- **Task sources (in priority order):**
  1. Action items from emails (requiring reply / action)
  2. Additional tasks from other sources
- **Do not insert** tasks that are already an open checkbox `- [ ]` in any project file — those are visible directly from the project
- Prefer tasks assigned to `[[@lsosnicki]]` or without any assignment

### Rules for "Notes", "Reflections", "Summary" sections

- **Always leave empty** — do not insert any content, placeholders, or `?`
- The user fills these in manually

### Rules for the "Meetings" section (`### Meetings`)

- Each calendar meeting = its own subsection `#### Title (HH:MM–HH:MM)`
- No meetings → insert `No meetings`
- Skip All Day / OOO / private blocks
- **Do not include attendees** — always omit this information
- **Recurring meetings (series)** → title and time only, no description
- **One-off meetings (non-recurring)** → title, time + short description from the meeting's `description` field (if present); skip if description is empty

---

## Step 7 — Save the file

`Filesystem:write_file` → `/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`

---

## Step 8 — Confirm and summarize

```
✅ Daily plan saved:
📅 /Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md
📆 Meetings: <N>
🚀 Active projects loaded: <N>
📬 Email action items: <N>
```

Also display a **brief chat summary**: meetings + top tasks for today.

---

## Edge Cases

- **No M365 access** → create file with empty Meetings section, add `⚠️ Could not fetch calendar`
- **Project file does not exist** → skip without error
- **Journal file already has content** → preserve it, fill only placeholders
- **Planning for tomorrow** → use tomorrow's calendar, projects unchanged
- **Filesystem unavailable** → display plan in chat, inform user that saving failed
