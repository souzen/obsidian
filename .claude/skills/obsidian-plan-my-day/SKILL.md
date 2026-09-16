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

This step only establishes the skeleton — actual data gets filled in during Step 6.

---

## Step 3 — Gather all inputs (in parallel)

- **Calendar** — `outlook_calendar_search` (date: the full target day, max_results: 20, timezone: Europe/Warsaw / CET/CEST).
  - Save: title, time (HH:MM–HH:MM), whether the meeting is recurring (series), meeting description (if present), and attendees.
  - Attendees are used only for the F2F detection in Step 4 — the Meetings section itself never persists either (see Step 6).

---

## Step 4 — Detect F2F/1:1 meetings and generate agendas

For each calendar meeting from Step 3 (skip All Day/OOO/private blocks, same as Step 6's Meetings rules):

- **Detect**: the meeting has exactly one attendee besides you (`lsosnicki@inpost.pl`).
- **Resolve the handle**: take that attendee's email local-part (before `@`). If it contains a `.` (e.g. a contractor suffix like `.sii`, `.b3`), also try the part before the first `.`. Check whether `work/areas/people/@<candidate>.md` exists for either form.
  - Match found → confirmed F2F, handle = `@<candidate>`.
  - No match for either form → not a detected F2F. Don't guess a handle, and don't check tags/aliases/other fields as a fallback — carry the attendee's raw email into Step 5 instead, so the user can rename/create the matching person file manually.
- **For each confirmed F2F**: run `prepare-f2f-agenda`'s Steps 2–4 (gather context, read the `przepis_na_f2f.md` template, build the agenda) for that handle. **Skip its Step 5 confirmation** — this skill runs unattended (including via the daily cron), so write the agenda directly rather than asking "should I save?".

Carry each built agenda (keyed by handle) — and each unresolved attendee email (keyed by meeting) — into Step 6 for insertion under that meeting's heading.

---

## Step 5 — Generate agendas for non-company-wide meetings

For each calendar meeting from Step 3 that Step 4 did **not** already claim as a confirmed F2F:

- **Detect company-wide meetings**: the title contains a company-wide keyword (case-insensitive) — `All Hands`, `Town Hall`, `Kickoff`, `Company`, `Ogólne`, `Cała firma`.
  - Match → company-wide meeting. Skip the rest of this step for it — treat it as a normal meeting in Step 6, even if its title also happens to match a project filename. Never run `prepare-meeting-agenda` for a company-wide meeting.
  - No match → continue below; run `prepare-meeting-agenda` for every one of these, whether or not it matches a project.
- **Resolve the project (`prepare-meeting-agenda` Step 1)**: `ls work/projects/` and match the meeting title case-insensitively against a filename, ignoring dashes/spaces — same rule as `obsidian-add-to-journal` Step 5a.
  - Match found → confirmed project meeting. Run `prepare-meeting-agenda`'s Step 2 (gather context) and Step 3 (build agenda) in full, project file included.
  - No match → no project file to ground the agenda in. Skip 2a (project file) and 2b (people/teams from project frontmatter); still run 2c (journal mentions, grep by meeting title instead of project name) and 2d (calendar). Build the agenda from whatever 2c/2d surface — if both come up empty, treat it as a normal meeting in Step 6 (no agenda, no breadcrumb; this is expected for one-off syncs with no history).
- **Skip `prepare-meeting-agenda`'s Step 1 user-confirmation prompt** for the no-match case too — this skill runs unattended, so never ask the user to confirm/pick a filename; a miss just means no project context, not a blocker.
- **Skip its Step 4 confirmation** — same reasoning as Step 4's F2F agendas: this skill runs unattended, so write the agenda directly rather than asking "should I save?".

Carry each built agenda (keyed by meeting) into Step 6 for insertion under that meeting's heading. When a project was matched, append `[[project-name]]` after the heading (as before); when none was matched, the heading gets no project link.

---

## Step 6 — Fill in the journal file

Fill in the skeleton from Step 2 with data gathered in Steps 3–5. If the journal file already existed, preserve the user's existing notes (Notes, Reflections, Summary sections, manual entries). Only fill in empty placeholders or sections with no content yet. Do not overwrite anything that looks like a manual entry.

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
- **Detected F2F meetings (Step 4)** → append `[[@handle]]` (and any project links the agenda references) after the heading, then insert the generated agenda bullets directly under it — same bullet format `prepare-f2f-agenda` produces
- **1:1-shaped meeting with no matching person file (Step 4)** → insert the attendee's raw email as a plain line directly below the heading (no formatting, no guessed handle), e.g.:
  ```
  #### jarek x łukasz (10:45–11:00)
  jaskoczylas@inpost.pl
  ```
  This is a breadcrumb for the user to create/fix the matching person file — not something later steps or skills should try to resolve automatically.
- **Meetings with a generated agenda (Step 5)** → insert the agenda bullets directly under the heading, same bullet format `prepare-meeting-agenda` produces; append `[[project-name]]` after the heading only if a project was matched, otherwise the heading gets no link

---

## Step 7 — Save the file

Write tool → `/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`

---

## Step 8 — Confirm and summarize

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

- **No M365 access** → create file with empty Meetings section, add `⚠️ Could not fetch calendar`; skip Steps 4–5 (no calendar data to detect F2Fs or project meetings from)
- **1:1-shaped meeting with no matching person file** → treat as a normal meeting, but still surface the unresolved email below the heading (Step 6); do not block the rest of the plan on it
- **Meeting title doesn't match any project** → still attempt a reduced agenda from journal mentions + calendar description (Step 5); only fall back to a normal meeting (Step 6, no breadcrumb) if that yields nothing either — this is expected for one-off syncs with no history, not an error
- **Company-wide meeting title also happens to match a project filename** → still treat as a normal meeting (Step 6); company-wide detection wins, no `prepare-meeting-agenda` run
- **Journal file already has content** → preserve it, fill only placeholders
- **Planning for tomorrow** → use tomorrow's calendar
- **Vault unavailable** → print the full plan content to chat/stdout (not just an error message) and inform the user that saving failed, so the plan isn't lost even if nothing gets written to disk
