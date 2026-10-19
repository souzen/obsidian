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

## Step 1 — Date and target file

Default: **today**. If the user gave another date (including a past one) → use it, no confirmation needed.

---

## Step 2 — Read the journal file

Read the target file with the Read tool.

- **Exists** → preserve all existing content; the note is added in Step 6.
- **Does not exist** → it will be created in Step 6.

---

## Step 3 — Find the meeting (calendar)

Use `Microsoft 365:outlook_calendar_search` on the target day:

- **Title given** → search by title keywords; take the closest title match.
- **Only approximate time given** (e.g. "meeting at 2pm") → take the meeting starting or ending closest to that time (±30 min).
- **No hints** → take the meeting that ended latest before now.

Keep the meeting title and time (`HH:MM–HH:MM`). **Never save participants.**

No match (or no calendar access) → use the title/time the user gave, and add `<!-- not found in calendar -->` under the heading.

---

## Step 4 — Edit the note

Take the user's raw notes and:

1. Fix typos, spelling and grammar; restore missing Polish diacritics (ą, ę, ó, ś, ź, ż, ć, ń, ł) in Polish text.
2. Keep the original language (PL or EN — never translate) and register (formal / informal).
3. Structure the note like this:

```markdown
<1–2 sentence summary>

#### Overview
- <discussed topic>
  - <supporting detail if needed>

#### Next Steps
- [ ] <action point> <assignee_list>

#### Decisions
- <decision>
```

- **Overview** — always present; bullets summarising the discussed topics (group related points, one idea per bullet).
- **Next Steps** and **Decisions** — only if applicable; if there are none, skip the whole section including its heading. Next Steps are checkboxes in the journal note only — never written to project files.
- `<assignee_list>` — the people assigned to the action point, written per the people rule below.

**Be concise** — shorter is better; the note is a reminder, not a transcript:
- Summary: one sentence (two only if really needed).
- Overview: aim for 3–6 bullets, each one line (roughly ≤ 20 words). Merge overlapping points, drop background, filler, repetition and anything that doesn't change what happens next. No sub-bullets unless a detail is essential.
- Keep what matters: decisions, numbers, dates, names, owners, deadlines, risks. Never shorten or drop a Next Step or a Decision to save space.
- Prefer plain words over long phrases; no intro phrases like "the team discussed that…".

**People rule** (applies everywhere in the note, including assignee lists):
- First and last name known → `[[@flastname]]`: first letter of the first name + last name, lowercase, no special characters (Łukasz Sośnicki → `[[@lsosnicki]]`).
- First or last name unknown → keep what you have as `@person` or `@group`, lowercase, no wikilink.

No note content provided → ask: "What do you want to save from the meeting?"

---

## Step 5 — Link projects

If the user named a project, or exactly one project is an obvious match for the meeting, add `[[project-name]]` after the heading (name = project filename without `.md`, e.g. `rollout 200k.md` → `[[rollout 200k]]`). Find the project per "Journal meeting entries" in CLAUDE.md. No match or ambiguous → no link, do not ask.

---

## Step 6 — Write the journal file

Heading: `### <Meeting title> (HH:MM–HH:MM) [[project-name]]`

- Strip any time pattern from the calendar/user title first (`10:00 Weekly Sync`, `Weekly Sync 14:00–15:00`, `(09:30) Daily standup` → `Weekly Sync` / `Daily standup`), then trim leftover whitespace and dashes.
- Add `(HH:MM–HH:MM)` from Step 3; omit it if the time is unknown. Omit links if none (Step 5).
- Under the heading goes the note from Step 4 (summary, then `Overview`, and `Next Steps` / `Decisions` when applicable).
- **Source marker:** if the input includes a marker line such as `<!-- wispr:<id> -->`, put it on its own line directly under the heading, above the summary. If that exact marker is already in the file, the note was imported before — write nothing and say so.

Heading levels: in an existing file, match the levels it already uses — the meeting heading uses the level of the other meeting headings (normally `###`, one below `## Meetings`), and the note's `Overview` / `Next Steps` / `Decisions` go one level deeper (normally `####`). In a new file use the levels in the template below.

Placement (existing file):
1. A heading for this meeting already exists (compare titles ignoring the time and links) → insert the note under it, **without asking**. If it already has content, append the note below the existing content and keep that content as it is (the source marker, if any, goes directly under the heading).
2. `## Meetings` exists but no such heading → append the new block at the end of that section.
3. No `## Meetings` section → append the section at the end of the file.

New file:

```markdown
---
date: <YYYY-MM-DD>
---

# <YYYY-MM-DD>

## Meetings

### <Meeting title> (HH:MM–HH:MM) [[project-name]]

<note from Step 4>
```

Saving:
- **Existing file** → insert the block **in place** with the Edit tool (anchor on the meeting heading, or on the end of the `## Meetings` section). Never rewrite the whole file: that can silently alter other lines (invisible characters, trailing spaces).
- **New file** → create it with the Write tool.

---

## Step 7 — Confirm

```
✅ Note saved:
📅 /Users/lsosnicki/obsidian/vault/journal/<YYYY-MM-DD>.md
📌 Meeting: <Title>
```

Also show the formatted note in the chat for review.
