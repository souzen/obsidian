---
name: obsidian-journal-add
description: >
  Adds a meeting note to the Obsidian journal file (journal/YYYY-MM-DD.md).
  Use this skill whenever the user wants to save meeting notes, write a meeting summary,
  add notes to the journal, write a follow-up after a meeting, or says anything like:
  "save from the meeting", "add a note", "I was in a meeting about X", "notes from daily",
  "write to journal", "meeting notes", "add to journal", "save in Obsidian", "journal entry".
  The skill automatically detects the meeting from the calendar (by title, date, time) if the
  user hasn't provided full details. Corrects language and saves to vault.
---

# Obsidian Journal Add — Meeting Notes

Adds a cleaned-up meeting note to the daily journal. Writes only to the journal file — never to project files.

**Journal:** `/Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md`

---

## Step 1 — Date and journal file

Default: **today**. Another date (including a past one) → use it, no confirmation needed.

Read the target file with the Read tool. If it exists, preserve all its content; if not, it is created in Step 5.

---

## Step 2 — Find the meeting (calendar)

Use `Microsoft 365:outlook_calendar_search` on the target day:

- **Title given** → closest title match by keywords.
- **Approximate time given** (e.g. "meeting at 2pm") → meeting starting or ending closest to it (±30 min).
- **No hints** → meeting that ended latest before now.

Keep the title and time (`HH:MM–HH:MM`). **Never save participants.**

No match (or no calendar access) → use the title/time the user gave and add `<!-- not found in calendar -->` under the heading.

---

## Step 3 — Edit the note

No note content provided → ask: "What do you want to save from the meeting?"

Take the user's raw notes and:

1. Fix typos, spelling and grammar; restore missing Polish diacritics (ą, ę, ó, ś, ź, ż, ć, ń, ł).
2. Keep the original language (PL or EN — never translate) and register (formal / informal).
3. Structure the note like this:

```markdown
<1–2 sentence summary>

#### Overview
- <discussed topic>

#### Next Steps
- [ ] <action point> <assignee_list>

#### Decisions
- <decision>
```

- **Overview** — always present.
- **Next Steps** and **Decisions** — only if applicable; otherwise skip the whole section including its heading. Next Steps are checkboxes in the journal note only — never written to project files.

**Be concise** — the note is a reminder, not a transcript:
- Summary: one sentence (two only if really needed).
- Overview: 3–6 one-line bullets (roughly ≤ 20 words each), one idea per bullet. Merge overlapping points; drop background, filler, repetition and anything that doesn't change what happens next. No sub-bullets unless a detail is essential.
- Keep decisions, numbers, dates, names, owners, deadlines, risks. Never shorten or drop a Next Step or a Decision.
- Plain words; no intro phrases like "the team discussed that…".

**People rule** — replace every person name, everywhere in the note, with a handle:
- First and last name known → `@flastname`: first letter of the first name + last name, lowercase, no special characters (Łukasz Sośnicki → `@lsosnicki`).
- First or last name unknown → use what you have as `@person` or `@group`, lowercase.
- Only in `Next Steps` is the handle a wikilink to the person file (`[[@lsosnicki]]`); handles with an unknown first or last name are never linked, and neither is `@lsosnicki` (the user) — it stays `**@lsosnicki**`.
- Every handle that is not a wikilink is written in bold (`**@lsosnicki**`, `**@person**`, `**@group**`).
- Put the assignees at the **end** of each Next Step, after the action text.

---

## Step 4 — Link project

If the user named a project, or exactly one project obviously matches the meeting, add `[[project-name]]` after the heading (filename without `.md`, e.g. `rollout 200k.md` → `[[rollout 200k]]`). Find it per "Journal meeting entries" in CLAUDE.md. No match or ambiguous → no link, do not ask.

---

## Step 5 — Write the journal file

Heading: `### <Meeting title> (HH:MM–HH:MM) [[project-name]]`

- Strip any time pattern from the title first (`10:00 Weekly Sync`, `Weekly Sync 14:00–15:00`, `(09:30) Daily standup` → `Weekly Sync` / `Daily standup`), then trim leftover whitespace and dashes.
- Omit `(HH:MM–HH:MM)` if the time is unknown; omit the link if none.
- The Step 3 note goes under the heading.

Heading levels: in an existing file, match the levels it already uses — the meeting heading at the level of the other meeting headings (normally `###`), `Overview` / `Next Steps` / `Decisions` one level deeper (normally `####`). In a new file use the template below.

**Existing file** — insert the block **in place** with the Edit tool; never rewrite the whole file (that can silently alter other lines, e.g. invisible characters or trailing spaces):
1. A heading for this meeting exists (compare titles ignoring time and links) → insert the note under it **without asking**; if it already has content, append below it and leave that content untouched.
2. `## Meetings` exists, no such heading → append the block at the end of that section.
3. No `## Meetings` section → append the section at the end of the file.

**New file** — create it with the Write tool:

```markdown
---
date: <YYYY-MM-DD>
---

# <YYYY-MM-DD>

## Meetings

### <Meeting title> (HH:MM–HH:MM) [[project-name]]

<note from Step 3>
```

---

## Step 6 — Confirm

```
✅ Note saved:
📅 /Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md
📌 Meeting: <Title>
```

Also show the formatted note in the chat for review.
