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

## Step 2 — Prepare the journal file's structure

- **Journal file** — Read tool on the target path (`/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`).
- **Template** — Read tool on `/Users/lsosnicki/obsidian/vault/work/resources/tools/obsidian/templates/daily.md`.
  - If it does not exist → abort and inform the user.
- **File exists** → keep its current structure as-is; use the template only as a reference for which sections/placeholders should exist, so you know what to fill in later — do not reorder or replace existing headings.
- **File does not exist** → use the loaded template as the base structure for the new file.
- If the template contains `{{date}}` or other Obsidian variables — substitute them with the correct value (e.g. date in `YYYY-MM-DD` format).

This step only establishes the skeleton — actual data gets filled in during Step 5.

---

## Step 3 — Gather all inputs (in parallel)

- **Calendar** — `outlook_calendar_search` (date: the full target day, max_results: 20, timezone: Europe/Warsaw / CET/CEST).
  - Save: title, time (HH:MM–HH:MM), whether the meeting is recurring (series), meeting description (if present), and attendees.
  - Attendees are used only for the F2F detection in Step 4 — the Meetings section itself never persists either (see Step 5).

---

## Step 4 — Classify meetings and build agendas

Assign every calendar meeting from Step 3 to exactly one of the 6 categories below. Evaluate them **in this order — the first match wins**. For each meeting, build whatever the category calls for and carry it (keyed by meeting) into Step 5.

This skill runs unattended (including via the daily cron) — never ask the user to confirm a match or "should I save?"; write directly.

| # | Category | Detected when | Agenda | Heading / content in the note |
|---|---|---|---|---|
| 1 | **Skipped** | All Day / OOO / private block | none | not written to the note |
| 2A | **F2F — person file found** | exactly one attendee besides you (`lsosnicki@inpost.pl`) and a person file matches | `prepare-f2f-agenda` | `[[@handle]]` (plus any project links the agenda references) after the heading, then the agenda bullets directly under it — same checkbox format `prepare-f2f-agenda` produces |
| 2B | **F2F — no person file** | exactly one attendee besides you, no person file matches | none | the attendee's raw email as a plain line directly below the heading (no formatting, no guessed handle) |
| 3 | **Company-wide** | title contains (case-insensitive) `All Hands`, `Town Hall`, `Kickoff`, `Company`, `Ogólne`, `Cała firma` | none — never run `prepare-meeting-agenda` | plain heading, even if the title also matches a project filename |
| 4 | **Project meeting** | title matches a `work/projects/` filename | none — do not run `prepare-meeting-agenda` | `[[project-name]]` after the heading; no agenda |
| 5 | **Reduced agenda** | no project match, but journal mentions (grep by meeting title) or the calendar description yield something | `prepare-meeting-agenda` Steps 2c–2d only (skip 2a/2b) | agenda bullets directly under the heading; no project link |
| 6 | **Plain** | nothing above matched and 5 found nothing | none | plain heading (expected for one-off syncs with no history — not an error) |

### Category 2 — F2F resolution (A / B)

- **Detect**: exactly one attendee besides you.
- **Resolve the handle**: take that attendee's email local-part (before `@`). If it contains a `.` (e.g. a contractor suffix like `.sii`, `.b3`), also try the part before the first `.`. Check whether `work/areas/_4_people/@<candidate>.md` exists for either form.
  - Match found → **2A**, handle = `@<candidate>`. Run `prepare-f2f-agenda`'s Steps 2–4 (gather context, read the `przepis_na_f2f.md` template, build the agenda) for that handle. Skip its Step 5 confirmation.
  - No match for either form → **2B**. Don't guess a handle, and don't check tags/aliases/other fields as a fallback — carry the attendee's raw email into Step 5 so the user can rename/create the matching person file manually. Example:
    ```
    ### jarek x łukasz (10:45–11:00)
    jaskoczylas@inpost.pl
    ```
    This is a breadcrumb for the user — not something later steps or skills should try to resolve automatically.

### Categories 4–6 — project resolution

- `ls work/projects/` and match the meeting title case-insensitively against a filename, ignoring dashes/spaces — same rule as `obsidian-add-to-journal` Step 5a.
- Match → category 4. Just link the project; do not gather context or build an agenda.
- No match → no project file to ground the agenda in. Run only 2c (journal mentions, grep by meeting title instead of project name) and 2d (calendar). If either surfaces something → category 5; if both come up empty → category 6.
- Never run `prepare-meeting-agenda`'s Step 1 user-confirmation prompt or its Step 4 confirmation (category 5 runs only its Steps 2c–2d) — a miss just means no project context, not a blocker.

---

## Step 5 — Fill in the journal file

Fill in the skeleton from Step 2 with data gathered in Steps 3–4. If the journal file already existed, preserve the user's existing notes (Notes, Reflections, Summary sections, manual entries). Only fill in empty placeholders or sections with no content yet. Do not overwrite anything that looks like a manual entry.

### Rules for "Notes", "Reflections", "Summary" sections

- **Always leave empty** — do not insert any content, placeholders, or `?`
- The user fills these in manually
- `Notes` and `Reflections` are nested as `### Notes` / `### Reflections` under a `## Journal %% fold %%` parent heading in the current template. Leave the parent heading and both subsections empty; preserve the `%% fold %%` marker verbatim (it's an Obsidian fold directive, not text to clean up).
- `Summary` is not part of this template — `obsidian-review-journal` adds it later, for already-elapsed days. If a journal file already has one, leave it alone same as the others.

### Rules for the "Meetings" section (`## Meetings`)

- Each meeting (every category except 1) = its own subsection `### Title (HH:MM–HH:MM)`
- No meetings → insert `No meetings`
- **Do not include attendees** — always omit this information (the only exception is the 2B breadcrumb email)
- **Recurring meetings (series)** → title and time only, no description
- **One-off meetings (non-recurring)** → title, time + short description from the meeting's `description` field (if present); skip if description is empty
- Category-specific links and agenda content under the heading: as defined in the Step 4 table
- **Meeting with nothing under its heading** (no agenda, description or breadcrumb — e.g. a category 4 or 6 meeting) → leave two blank lines below the heading instead of one, so the user has room to type notes

---

## Step 6 — Save the file

Write tool → `/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`

---

## Step 7 — Confirm and summarize

```
✅ Daily plan saved:
📅 /Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md
📆 Meetings: <N>
🤝 F2F agendas generated: <N>
📋 Meeting agendas generated: <N>
```

Also display a **brief chat summary**: meetings for today.

---

## Edge Cases

- **No M365 access** → create file with empty Meetings section, add `⚠️ Could not fetch calendar`; skip Step 4 (no calendar data to classify)
- **Category 2B** → treat as a normal meeting, but still surface the unresolved email below the heading; do not block the rest of the plan on it
- **Category 3 title also matches a project filename** → company-wide wins (it is checked first); no `prepare-meeting-agenda` run
- **Journal file already has content** → preserve it, fill only placeholders
- **Planning for tomorrow** → use tomorrow's calendar
- **Vault unavailable** → print the full plan content to chat/stdout (not just an error message) and inform the user that saving failed, so the plan isn't lost even if nothing gets written to disk
