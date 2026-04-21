---
name: obsidian-add-to-journal
description: >
  Adds a meeting note to the Obsidian journal file (journal/YYYY-MM-DD.md).
  Use this skill whenever the user wants to save meeting notes, write a meeting summary,
  add notes to the journal, write a follow-up after a meeting, or says anything like:
  "save from the meeting", "add a note", "I was in a meeting about X", "notes from daily",
  "write to journal", "meeting notes", "add to journal", "save in Obsidian", "journal entry".
  The skill automatically detects the meeting from the calendar (by title, date, time) if the
  user hasn't provided full details. Corrects language and saves to vault.
---

# Obsidian Add to Journal — Meeting Notes

Adds a formatted meeting note to the Obsidian vault journal.

**Vault:** `/Users/lsosnicki/vault`  
**Journal:** `/Users/lsosnicki/vault/journal/`

---

## Step 1 — Determine date and target file

- Default: **today**. If the user provided a different date → use it.
- Format: `YYYY-MM-DD` → file: `/Users/lsosnicki/vault/journal/<YYYY-MM-DD>.md`

---

## Step 2 — Read the journal file

Use `Filesystem:read_text_file` on the target path.

- **File exists** → read full content; preserve existing content — the note will be **appended** to the `### Meetings` section or at the end of the file.
- **File does not exist** → create a new file with minimal structure (see Step 6b).

---

## Step 3 — Detect meeting from calendar

Use `Microsoft 365:outlook_calendar_search` to find the meeting.

### Search strategy

**If the user provided a meeting title:**
- Search by keywords from the title on the target day.
- Match the meeting with the highest title similarity.

**If the user provided only an approximate time (e.g. "meeting at 2pm"):**
- Retrieve all meetings for the target day.
- Find the meeting that starts or ends closest to the given time (±30 min tolerance).

**If the user provided no hints:**
- Retrieve all meetings for the target day.
- Select the one that ended latest before the current time (i.e. most likely the one that just happened).

### Data to save from the meeting (from calendar)
- Meeting title
- Time: `HH:MM–HH:MM`
- Description (if it exists and is not a recurring boilerplate)
- **Do NOT save participants**

If no match found → use the title/time provided by the user, without calendar data.

---

## Step 4 — Edit the note

Take the user's raw notes/bullet points and:

### 4a — Language correction
- Fix typos, spelling, and grammar errors.
- Restore missing Polish diacritic characters (ą, ę, ó, ś, ź, ż, ć, ń, ł) if the note is in Polish.
- Preserve the register (formal / informal) matching the original.
- Preserve the note's language (PL or EN) — do not translate.

### 4b — Structure as bullet points
Organise the note as a clean bullet-point list:

1. Convert all content into bullet points and sub-bullet points.
2. Group related points logically (decisions, blockers, open questions, context).
3. Use sub-bullets for supporting details or context under a main point.
4. Keep each bullet concise — one idea per bullet.

### Output format (for journal — NO action items)

```markdown
- <point 1>
  - <sub-point if needed>
- <point 2>
- <point 3>
...
```

> **Action items do NOT go into the journal note** — they are saved directly to project files (Step 5).  
> Exception: if no project match was found → action item goes at the end of the note as `#todo` (see Edge Cases).

---

## Step 5 — Detect project and save action items

Execute this step **before** saving the journal, if the user's notes contain clear action items / follow-ups.

### 5a — Detect project file

**Projects folder:** `/Users/lsosnicki/vault/work/projects/`

Get the file list: `Filesystem:list_directory` on `/Users/lsosnicki/vault/work/projects/`.

For each action item, try to match a project:

1. **User explicitly named the project** (e.g. "rollout 200k", "GOKPER", "bielik-ext") →
   - Search for a file whose name contains the given string (case-insensitive, ignore dashes/spaces).
   - If one file found → use it.
   - If >1 matching → **ask the user** which one they mean (show the list).

2. **User did not name the project** →
   - Try to infer from context (meeting title, keywords in the note).
   - If match is certain (one obvious candidate) → use it and inform the user.
   - If ambiguous (>1 candidate or none) → **ask the user**: "Which project should I save the action items to? (or type 'todo' to mark as #todo)"

3. **No matching file found** → mark as `#todo` (see Edge Cases).

### 5b — Append action items to project file

Read the project file: `Filesystem:read_text_file`.

Append action items at the end of the file as a block:

```markdown

#### Action items — <Meeting title> (<YYYY-MM-DD>)
- [ ] <action item 1> ← @person or unassigned
- [ ] <action item 2>
```

Save: `Filesystem:write_file` (full file with appended block).

---

## Step 6 — Append note to journal file

### If file exists and has a `## Meetings` section or `### <meeting title>` subsection

Find the right place:
1. If a `### <meeting title>` subsection already exists → insert the note directly under the heading (replace empty space or append).
2. If `## Meetings` section exists but no such subsection → append a new subsection at the end of the Meetings section.
3. If no `## Meetings` section → append the entire section at the end of the file.

### Format of the appended block

If projects were detected (Step 5) → append Obsidian links after the title:

```markdown
### <Meeting title> [[project-name1]] [[project-name2]]

- <point 1>
- <point 2>
```

If no project detected → heading without links:

```markdown
### <Meeting title>
```

> The name in `[[...]]` = project file name **without the `.md` extension**, exactly as it appears in the `work/projects/` folder.  
> Example: file `rollout 200k.md` → `[[rollout 200k]]`

---

## Step 7 — Save the journal file

`Filesystem:write_file` → `/Users/lsosnicki/vault/journal/<YYYY-MM-DD>.md`

Write the **full file** (not a patch) — the read content + appended note.

### Title cleanup before saving

When constructing the `### <Meeting title>` heading, **strip any time/hour pattern from the title**:
- Remove patterns like `HH:MM`, `HH:MM–HH:MM`, `HH:MM-HH:MM`, `(HH:MM)`, `[HH:MM]` from the meeting title.
- Also remove leading/trailing whitespace and dashes left after stripping.
- Examples:
  - `"10:00 Weekly Sync"` → `"Weekly Sync"`
  - `"Weekly Sync 14:00–15:00"` → `"Weekly Sync"`
  - `"(09:30) Daily standup"` → `"Daily standup"`

### 7b — If the file did not exist

Create a minimal structure:

```markdown
---
date: <YYYY-MM-DD>
---

# <YYYY-MM-DD>

## Meetings

### <Meeting title>

<note>
```

---

## Step 8 — Confirm

```
✅ Note saved:
📅 /Users/lsosnicki/vault/journal/<YYYY-MM-DD>.md
📌 Meeting: <Title>
📁 Action items → <project file name> (N items)
```

If there were action items marked `#todo` → list them separately.

Also **display the edited note in the chat** (the bullet-point version), so the user can review it.

---

## Edge Cases

| Situation | Behaviour |
|---|---|
| No M365 access / no calendar | Use user-provided data; save without calendar metadata |
| Meeting not found in calendar | Use title/time from user; add `⚠️ Not found in calendar` as a comment |
| User provided no note content | Ask: "What do you want to save from the meeting?" |
| Journal file does not exist | Create a new one (Step 7b) |
| Note is in English | Keep English; do not translate |
| User provided a past date | Use that date; do not ask for confirmation |
| Action item — project unknown / ambiguous | Append at the end of the journal note as: `- [ ] <content> #todo` |
| Action item — project file does not exist on disk | Same as above — append as `#todo` in journal and inform the user |
| No action items in notes | Skip Step 5 entirely |
