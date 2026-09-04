---
name: obsidian-plan-my-day
description: "Creates a structured daily work plan in Obsidian (journal/YYYY-MM-DD.md) from the Outlook calendar. Always use this skill when the user says \"plan my day\", \"what's on my agenda\", \"daily plan\", \"give me an overview of tomorrow\", \"help me prepare for tomorrow\", \"what do I have today\", \"what's tomorrow look like\", or when they want today's/tomorrow's meetings turned into a journal file. Trigger even for loose phrasings like \"what do I have today?\" or \"sort out my day\". Output is always a .md file saved to vault/journal/."
---

# Plan My Day

Creates a structured daily plan **for work matters only** and saves it to the Obsidian vault.

> ⚠️ Scope: `work/` system only. Never pull data from `my/` — that PARA system covers personal matters and does not belong in the daily plan.

**Vault:** `/Users/lsosnicki/obsidian/vault`
**Journal:** `/Users/lsosnicki/obsidian/vault/journal/`
**Template:** `/Users/lsosnicki/obsidian/vault/work/resources/tools/obsidian/templates/daily.md`

---

## Step 1 — Determine the date

- Default: **today**. If "tomorrow" → use tomorrow's date.
- Format: `YYYY-MM-DD` → target file: `/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`

---

## Step 2 — Gather all inputs (in parallel)

None of the following depend on each other's output — issue the reads/searches together in the same turn rather than one at a time:

- **Journal file** — Read tool on the target path (`/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`).
  - File exists → note to preserve its existing content in Step 3.
  - File does not exist → note that the template (below) will be used as the base structure in Step 3.
- **Template** — Read tool on `/Users/lsosnicki/obsidian/vault/work/resources/tools/obsidian/templates/daily.md`.
  - If it does not exist → abort and inform the user.
- **Calendar** — `outlook_calendar_search` (date: the full target day, max_results: 20, timezone: Europe/Warsaw / CET/CEST).
  - Save: title, time (HH:MM–HH:MM), whether the meeting is recurring (series), meeting description (if present). Do **not** save attendees.

---

## Step 3 — Build the journal file

Using the template and data gathered in Step 2:

- **New file**: use the loaded template as the base structure of the journal file.
- **Existing file**: keep the file's current structure as-is; use the template only as a reference for which sections/placeholders should exist, so you know what to fill in — do not reorder or replace existing headings.
- If the template contains `{{date}}` or other Obsidian variables — substitute them with the correct value (e.g. date in `YYYY-MM-DD` format).

Fill in sections with data gathered in Step 2. If the journal file already existed, preserve the user's existing notes (Notes, Reflections, Summary sections, manual entries). Only fill in empty placeholders or sections with no content yet. Do not overwrite anything that looks like a manual entry.

### Rules for "Notes", "Reflections", "Summary" sections

- **Always leave empty** — do not insert any content, placeholders, or `?`
- The user fills these in manually
- `Notes` and `Reflections` are nested as `#### Notes` / `#### Reflections` under a `### Journal %% fold %%` parent heading in the current template. Leave the parent heading and both subsections empty; preserve the `%% fold %%` marker verbatim (it's an Obsidian fold directive, not text to clean up).
- `Summary` is not part of this template — `obsidian-review-journal` adds it later, for already-elapsed days. If a journal file already has one, leave it alone same as the others.

### Rules for the "Meetings" section (`### Meetings`)

- Each calendar meeting = its own subsection `#### Title (HH:MM–HH:MM)`
- No meetings → insert `No meetings`
- Skip All Day / OOO / private blocks
- **Do not include attendees** — always omit this information
- **Recurring meetings (series)** → title and time only, no description
- **One-off meetings (non-recurring)** → title, time + short description from the meeting's `description` field (if present); skip if description is empty

---

## Step 4 — Save the file

Write tool → `/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`

---

## Step 5 — Confirm and summarize

```
✅ Daily plan saved:
📅 /Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md
📆 Meetings: <N>
```

Also display a **brief chat summary**: meetings for today.

---

## Edge Cases

- **No M365 access** → create file with empty Meetings section, add `⚠️ Could not fetch calendar`
- **Journal file already has content** → preserve it, fill only placeholders
- **Planning for tomorrow** → use tomorrow's calendar
- **Vault unavailable** → print the full plan content to chat/stdout (not just an error message) and inform the user that saving failed, so the plan isn't lost even if nothing gets written to disk
