---
name: prepare-meeting-agenda
description: >
  Prepares a proposed agenda for any work meeting about a specific project by
  gathering context from the Obsidian vault (the project file's Outcomes, Next
  Actions, Decisions, Questions, Issues; recent journal mentions; linked people
  and teams; resource links) and the calendar. Use this skill when the user says
  things like "prepare meeting agenda", "przygotuj agendę na spotkanie", "agenda
  na spotkanie o <project>", "przygotuj mnie na spotkanie z <project>", or asks
  for topics/agenda for an upcoming meeting tied to a named project. Unlike
  prepare-f2f-agenda, this is not for 1:1s and does not use the przepis_na_f2f
  template — it derives the agenda structure from the project's own sections.
  After presenting the agenda, offers to save it as today's meeting note in the
  journal.
---

# Prepare Meeting Agenda

Builds a proposed agenda for a project-related meeting, grounded in what's actually
in the vault, then optionally saves it as today's meeting note.

**Vault:** `/Users/lsosnicki/obsidian/vault`
**Projects:** `/Users/lsosnicki/obsidian/vault/work/projects/`
**People:** `/Users/lsosnicki/obsidian/vault/work/areas/people/`
**Teams:** `/Users/lsosnicki/obsidian/vault/work/areas/people/teams/` (recurse into subfolders — team notes are named `~<slug>.md`)
**Journal:** `/Users/lsosnicki/obsidian/vault/journal/`

---

## Step 1 — Resolve the project

Input is a project name (e.g. `stage-env`, `product-feeds`, `cag-upgrade`).

- `ls work/projects/` and match case-insensitively against filenames (ignore dashes/spaces, same rule as `obsidian-add-to-journal` Step 5a).
- If ambiguous (>1 match) or no match → ask the user to pick / confirm the filename.
- Read the resolved file with the Read tool.

---

## Step 2 — Gather context from the vault

Run these lookups in parallel where possible:

### 2a — The project file itself (already read in Step 1)
Extract, section by section:
- `### Outcomes` — the goal, so the agenda stays anchored to it. If the section is missing entirely (not just placeholder text), fall back to the project's top-level description or a linked strategy doc from Resources, and say explicitly that no Outcomes section exists yet
- `### Next Actions` — open (`- [ ]`) items, especially any with an owner tag
- Decisions — anything unresolved (placeholder text like `todo`/`decisions` means none logged yet — skip it)
- Questions — open questions
- Issues — open issues/blockers
- Resources — links (Jira, Confluence, SharePoint). List them for reference; do not fetch external URLs unless the user explicitly asks

Decisions/Questions/Issues/Resources: match by keyword, not a fixed heading level. The current project template nests Decisions/Questions/Issues as `#### Decisions` / `#### Questions` / `#### Issues` under a `### Log %% fold %%` parent, with `### Resources` moved earlier — but older project files still use all of these as flat `###` sections. Handle both.

### 2b — People and teams linked to the project
From frontmatter (`owner`, `co-owner`, `EM`, `PM`, `SEM`, `members`, `team`) and any `[[@handle]]` mentions in the body. For each, note their role — this tells you who should be in the room / whose open items matter here.

Plain-text names in the body with no `[[@handle]]` link (e.g. an external contact mentioned only by first name) have no person note to check, but still surface them narratively next to the action item they're tied to — don't drop them just because they're unlinked.

### 2c — Recent journal mentions
`grep -rl "<project-name>" journal/*.md` (try both the exact filename and its `[[wikilink]]` form, e.g. `[[stage-env]]`).

Read matching files — look for:
- Past meeting notes about this project
- Decisions or blockers logged that haven't made it into the project file's Decisions/Issues section yet (a gap worth flagging)
- Action items about this project that aren't yet in `### Next Actions`

### 2d — Today's calendar entry (optional context)
Same approach as `obsidian-add-to-journal` Step 3: search `Microsoft 365:outlook_calendar_search` for the project name/keywords, or use an existing `### <title> [[project-name]]` heading already in today's journal if present. Note title/time — this becomes the meeting to attach the saved note to in Step 4.

If that journal heading **already has content under it**, treat it as additional input for 2c rather than an empty slot, and don't plan to silently overwrite it (see Edge Cases).

---

## Step 3 — Build and present the proposed agenda

There is no fixed template — derive the shape from what Step 2 actually found. A typical shape:

- **Status / recap** — current state vs. `### Outcomes`, recent progress from journal (2c)
- **Decyzje do podjęcia** — open Decisions and any Next Actions that are really decisions in disguise
- **Otwarte pytania** — Questions
- **Blokery** — Issues
- **Next steps** — remaining `### Next Actions`, with the linked people (2b) who'd own each

Output the agenda as a bullet list (`-`), not a numbered list — including when saving it into the journal (Step 4).

Skip any of these sections if Step 2 found nothing for it — don't invent content. Cite the source section inline, e.g.:
```
- Decyzja: czy przechodzimy na rozwiązanie off-the-shelf, czy zostaje CAG jako open source? (Decisions, [[cag-upgrade]])
```

If a journal mention (2c) surfaced something not yet reflected in the project file (e.g. a decision made verbally but not logged), flag it explicitly so the user can decide whether to update the project file too — don't fold it in silently.

Present the agenda in chat as a bullet list grouped by section. Do not save anything yet.

If Step 2 found nothing usable (empty project file, no journal mentions) → say so explicitly and present a bare skeleton (status / decisions / questions / issues / next steps) instead of inventing topics.

---

## Step 4 — Offer to save today's meeting note

After presenting the agenda, ask:

> "Zapisać tę agendę jako notatkę dzisiejszego spotkania w journalu? (tak / nie)"

**If yes:**
- Use the meeting title/time found in Step 2d, or ask the user for one if nothing matched.
- Insert the agenda bullets under that meeting heading (same insertion rules as `obsidian-add-to-journal` Step 6: existing heading → insert under it; `## Meetings` section exists but no heading → append one; no `## Meetings` section → append it).
- Link the project after the heading, e.g. `### <title> [[project-name]]`, plus any people/teams from 2b worth linking.
- Save with the Write tool (full file, not a patch) and confirm:
  ```
  ✅ Note saved:
  📅 journal/<YYYY-MM-DD>.md
  📌 Meeting: <title> [[project-name]]
  ```

**If no:** stop, don't write anything.

---

## Edge Cases

| Situation | Behaviour |
|---|---|
| Project name not found in `work/projects/` | Ask the user to confirm the exact filename |
| Project file has only placeholder content (`todo`, `outcomes`, `decisions`) | Note it explicitly; build agenda mostly from journal mentions and linked people instead |
| No journal mentions found at all | Continue with project-file-only agenda, say so explicitly |
| No meeting found today (calendar or journal) | Still offer to save — create a new heading without a time, ask user for a title if needed |
| Today's meeting heading already has content under it | Don't overwrite. Show the existing content alongside the new agenda and ask: append below, replace, or skip saving |
| Journal mentions reveal decisions/issues not yet in the project file | Flag the gap to the user; don't merge it into the project file automatically (out of scope for this skill) |
