---
name: obsidian-journal-import-wispr
description: >
  Imports finalized meeting notes from Wispr Flow Notetaker into the Obsidian journal by running
  the obsidian-journal-add skill once per note. Use this skill whenever the user wants to pull,
  load, check, sync or import notes from Wispr Flow into the journal, or says things like
  "import wispr notes", "add new wispr notes to journal", "sync wispr", "load latest note from wispr",
  "zaimportuj notatki z wispr", "dodaj notatki z wispr do dziennika", "są nowe notatki we wispr?".
  Interactive — it asks only before creating a new journal entry; notes for meetings already in the journal are added without asking.
---

# Obsidian Journal Import — Wispr Flow

Fetches finalized meeting notes from Wispr Flow and adds each one to the journal by invoking the `obsidian-journal-add` skill. This skill never formats or writes journal content itself — `obsidian-journal-add` does that.

**Time zone:** Europe/Warsaw. Wispr returns UTC — convert every time before showing it or using it.

---

## Step 1 — Choose the window

Default: **today** (00:00 → tomorrow 00:00, local). The user may say:
- "yesterday" / a date → that day
- "since 14:00" → from that time until now
- "latest note" → only the single most recent finalized note (no window)

Build `since` / `until` as ISO timestamps with the local offset.

---

## Step 2 — List the notes

Call `mcp__wispr-flow__search_meetings` with `since` / `until` (or no dates for "latest note", `limit: 1`). Follow `next_cursor` while `has_more` is true. If a call fails with a connection error (e.g. `ECONNRESET`), retry once.

- Keep only `finalized: true`.
- Sort by start time, oldest first.
- Show times in local time.

---

## Step 3 — Skip notes that are already imported

`obsidian-journal-add` writes a marker `<!-- wispr:<meeting id> -->` under every note it saves from Wispr. For each note, read the journal of its start date (`vault/journal/<YYYY-MM-DD>.md`) and check whether the marker with that meeting id is present → **already imported**.

Show the user a table:

| Time | Wispr title | Status |
|---|---|---|
| 09:00–09:30 | … | new / already imported |

No new notes → say so and stop.

---

## Step 4 — Import each new note (oldest first, one at a time)

Notes are processed sequentially — they edit the same journal file.

For each new note:

1. **Fetch the note:** `mcp__wispr-flow__get_meeting` with the meeting id. Use the `summary` (it holds the Overview / Next Steps / Decisions) plus `content` if present. Use the transcript (`view_transcript`) only when the summary is empty or clearly insufficient. Retry once on a connection error.
2. **Match the calendar event:** search Outlook (`Microsoft 365:outlook_calendar_search`) for the note's day and pick the event whose time overlaps the note's start/end the most (it should cover at least half of the shorter of the two).
   - Match found → use the **calendar title and time** (Wispr titles are auto-generated and often differ). If the note's content clearly does not fit that event's title, flag the mismatch to the user instead of guessing.
   - No match → use the Wispr title and its local time, and tell the user it isn't in the calendar.
3. **Confirm — only if the journal has no entry for this meeting.** Compare the meeting title (the calendar title if step 2 matched, otherwise the Wispr title) with the `###` headings in that day's journal, ignoring the time and links.
   - **A matching heading exists** → import without asking.
   - **No matching heading** → ask: "Import '<title>' (HH:MM–HH:MM)? It will create a new entry." — options: import / skip / stop.
4. **Invoke `obsidian-journal-add`** with the Skill tool. Pass in the args:
   - the date (`YYYY-MM-DD`)
   - the meeting title and time (`HH:MM–HH:MM`) from step 2
   - the source marker line: add `<!-- wispr:<meeting id> -->`
   - the raw note text from step 2
   - a remark that speakers appear as `speaker1` / `speaker2` and that the note comes from Wispr Flow
5. Let `obsidian-journal-add` run to the end. If the heading already has content, journal-add appends below it (no question). Do not write to the journal yourself.

---

## Step 5 — Report

```
✅ Imported: N   ⏭ Skipped: N   ❗ Failed: N
- HH:MM–HH:MM <title> — imported / skipped / failed (<reason>)
```

List any title mismatches or notes not found in the calendar so the user can review them.

---

## Rules

- Never create a new journal entry without the user's go-ahead (Step 4.3); notes that go under an existing matching heading need no confirmation. This skill is not meant for unattended runs.
- Never write to project files or the journal directly — only through `obsidian-journal-add`.
- Don't re-import a note whose marker is already in the journal; if the user explicitly wants a re-import, append it below the existing content.
- Keep the Wispr share link out of the journal; only the meeting-id marker is stored.
