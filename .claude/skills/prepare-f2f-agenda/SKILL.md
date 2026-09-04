---
name: prepare-f2f-agenda
description: >
  Prepares a proposed F2F / 1:1 meeting agenda for a specific person by gathering
  context from the Obsidian vault (journal entries, project files, team files, past
  meeting notes) and structuring it using the przepis_na_f2f.md template. Use this
  skill when the user says things like "prepare f2f agenda", "przygotuj agendę na f2f",
  "agenda na spotkanie z @X", "co powiedzieć na f2f z X", "przygotuj mnie na 1:1 z X",
  or asks for topics/agenda for an upcoming 1:1 with a named person. After presenting
  the agenda, offers to save it as today's meeting note in the journal.
---

# Prepare F2F Agenda

Builds a proposed 1:1/F2F agenda for a person, grounded in what's actually in the vault
(not generic management advice), then optionally saves it as today's meeting note.

**Vault:** `/Users/lsosnicki/obsidian/vault`
**Template:** `/Users/lsosnicki/obsidian/vault/work/resources/people-management/feedback/przepis_na_f2f.md`
**People:** `/Users/lsosnicki/obsidian/vault/work/areas/people/`
**Projects:** `/Users/lsosnicki/obsidian/vault/work/projects/`
**Teams:** `/Users/lsosnicki/obsidian/vault/work/areas/people/teams/` (recurse into subfolders — team notes are named `~<slug>.md`)
**Journal:** `/Users/lsosnicki/obsidian/vault/journal/`

---

## Step 1 — Resolve the person

Input is a handle like `@jpintera` or a name.

- If given `@handle` → target file is `work/areas/people/@handle.md`. Read it with the Read tool.
- If given a name instead of a handle → `ls work/areas/people/` and match case-insensitively against filenames. If ambiguous (>1 match) or no match → ask the user to confirm the handle.
- If the person file is empty → that's fine, continue (not all person notes have content).

---

## Step 2 — Gather context from the vault

Run these lookups (in parallel where possible):

### 2a — Person's own note
Already read in Step 1. Extract any open (`- [ ]`) items — these are standing things to raise with this person.

### 2b — Projects and teams linked to this person
`grep -rl "@handle" work/projects/ work/areas/people/teams/` (also check without the `@` in case of plain-text mentions).

Team notes are named `~<slug>.md` — that `~` prefix marks a match as a team rather than a project. The slug (filename without `~`/`.md`) is reused in Step 2c and in `[[~slug]]` links.

For each matching file, read it and note:
- Their role (owner, co-owner, EM, PM, SEM, member — from frontmatter)
- Open `### Next Actions` items, especially any already tagged to this person
- Anything in Decisions / Issues / Questions that looks unresolved or recent — match by keyword, not a fixed heading level: the current project template nests these as `#### Decisions` / `#### Questions` / `#### Issues` under a `### Log %% fold %%` parent, but older project files still use them as flat `###` sections

### 2c — Recent journal mentions
`grep -rl "@handle" journal/*.md`. Also run `grep -rl "<FirstName>" journal/*.md` using the person's first name (from their person-note title, project frontmatter, or the meeting title convention `FirstName / Łukasz`) — older meeting notes may predate the `[[@handle]]` linking convention and won't match the handle grep alone.

If 2b found a team this person belongs to (EM/PM/SEM/member), also run `grep -ril "<team-slug>" journal/*.md` using that team's slug (from 2b) and its `[[~team-slug]]` wikilink form. Team-wide entries (reorgs, retros, incidents) often don't name the person directly and would otherwise be missed.

Read matching files — look for:
- Past meeting notes headed with this person's name (e.g. `### Name / Łukasz [[@handle]]`)
- Action items assigned to or about this person
- Any decisions/blockers logged that involve them or their team

### 2d — Today's calendar entry (optional context)
If a meeting with this person is already in today's journal (`### <title> [[@handle]]`) or can be found via `Microsoft 365:outlook_calendar_search`, note its title/time — this becomes the meeting to attach the saved note to in Step 5.

If that journal heading **already has content under it** (not just a bare title/time), treat that content as additional input for Step 2c rather than an empty slot — it may be earlier prep or partial notes for the same meeting. Do not silently overwrite it later in Step 5; see the Edge Cases table.

---

## Step 3 — Read the F2F template

Read `work/resources/people-management/feedback/przepis_na_f2f.md`. Its actual structure is 4 items with time allocations (for a 30-min f2f): (1) 5 min — Twoje sukcesy / co idzie dobrze w ostatnim tygodniu, (2) 15 min — aktualne wyzwania (co to jest, jakie podejście planujesz zastosować, moja perspektywa), (3) 5 min — feedback dla Ciebie, (4) 5 min — feedback dla mnie. There is no separate "operational topics" section — project/team status items belong inside item 2, as the substance of the "aktualne wyzwania" discussion. Use this exact 4-item skeleton; adapt the minute allocations proportionally if the meeting is shorter/longer than 30 min.

---

## Step 4 — Build and present the proposed agenda

Map gathered context onto the template's 4 items. Populate:

- **Sukcesy / co idzie dobrze (X min)** — leave as an open prompt for the person unless the vault clearly shows a recent win
- **Aktualne wyzwania (X min)** — open items from linked projects/teams that need a decision or status check (from 2b), plus unresolved issues/blockers/questions found in 2b/2c and open items from the person's own note (2a)
- **Feedback dla Ciebie (X min)** — leave as an open prompt
- **Feedback dla mnie (X min)** — leave as an open prompt (this is inherently conversational, not derivable from notes)

Output the agenda as a bullet list (`-`), not a numbered list — including when saving it into the journal (Step 5).

For each topic pulled from a project or journal entry, cite the source inline so the user can verify, e.g.:
```
- Status [[cag-upgrade]] — decyzja OneWeb BE nt. backlogu wciąż otwarta (Next Actions)
```

Present the agenda in chat as a bullet list grouped by template section. Do not save anything yet.

If Step 2 found nothing at all for this person (empty note, no project/journal matches) → say so explicitly and present a bare template-only agenda instead of inventing topics.

---

## Step 5 — Offer to save today's meeting note

After presenting the agenda, ask:

> "Zapisać tę agendę jako notatkę dzisiejszego spotkania w journalu? (tak / nie)"

**If yes:**
- Determine today's meeting title/time the same way as `obsidian-add-to-journal` Step 3 (match calendar entry with this person, or use the existing `### <title> [[@handle]]` heading already in today's journal if present).
- Insert the agenda bullets under that meeting heading (same insertion rules as `obsidian-add-to-journal` Step 6: existing heading → insert under it; `## Meetings` section exists but no heading → append one; no `## Meetings` section → append it).
- Link any projects referenced in the agenda after the heading, e.g. `### <title> [[@handle]] [[project-name]]`.
- Save with the Write tool (full file, not a patch) and confirm:
  ```
  ✅ Note saved:
  📅 journal/<YYYY-MM-DD>.md
  📌 Meeting: <title> [[@handle]]
  ```

**If no:** stop, don't write anything.

---

## Edge Cases

| Situation | Behaviour |
|---|---|
| Person handle not found in `work/areas/people/` | Ask the user to confirm the exact handle/filename |
| Person note is empty | Continue — rely on projects/journal instead |
| No projects/journal mentions found at all | Present a template-only agenda and say explicitly that no vault context was found |
| `przepis_na_f2f.md` missing or restructured | Fall back to the generic shape: wins → challenges → feedback both ways |
| No meeting with this person found today (calendar or journal) | Still offer to save — create a new `### <name>` heading without a time |
| Multiple projects link to the person | Include all of them, grouped under "Aktualne wyzwania" |
| Today's meeting heading already has content under it | Don't overwrite. Show the existing content to the user alongside the new agenda and ask: append the new agenda below the existing content, replace it, or skip saving |
