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
**Teams:** `/Users/lsosnicki/obsidian/vault/work/areas/gokart/teams/`
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
`grep -rl "@handle" work/projects/ work/areas/gokart/teams/` (also check without the `@` in case of plain-text mentions).

For each matching file, read it and note:
- Their role (owner, co-owner, EM, PM, SEM, member — from frontmatter)
- Open `### Next Actions` items, especially any already tagged to this person
- Anything in `### Decisions` / `### Issues` / `### Questions` that looks unresolved or recent

### 2c — Recent journal mentions
`grep -rl "@handle" journal/*.md` (or the person's plain name if the handle doesn't appear).

Read matching files — look for:
- Past meeting notes headed with this person's name (e.g. `### Name / Łukasz [[@handle]]`)
- Action items assigned to or about this person
- Any decisions/blockers logged that involve them

### 2d — Today's calendar entry (optional context)
If a meeting with this person is already in today's journal (`### <title> [[@handle]]`) or can be found via `Microsoft 365:outlook_calendar_search`, note its title/time — this becomes the meeting to attach the saved note to in Step 5.

---

## Step 3 — Read the F2F template

Read `work/resources/people-management/feedback/przepis_na_f2f.md`. Use its numbered structure (operational topics → wins/what went well → current challenges → feedback both ways) as the skeleton for the agenda. Preserve the time allocations if present; adapt them proportionally if the meeting is shorter/longer than the template assumes.

---

## Step 4 — Build and present the proposed agenda

Map gathered context onto the template skeleton. Populate:

1. **Tematy operacyjne** — open items from linked projects/teams that need a decision or status check (from 2b)
2. **Sukcesy / co idzie dobrze** — leave as an open prompt for the person unless the vault clearly shows a recent win
3. **Aktualne wyzwania** — unresolved issues/blockers/questions found in 2b/2c; open items from the person's own note (2a)
4. **Feedback dla Ciebie / dla mnie** — leave as an open prompt (this is inherently conversational, not derivable from notes)

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
| `przepis_na_f2f.md` missing or restructured | Fall back to the generic shape: operational topics → wins → challenges → feedback both ways |
| No meeting with this person found today (calendar or journal) | Still offer to save — create a new `### <name>` heading without a time |
| Multiple projects link to the person | Include all of them, grouped under "Tematy operacyjne" |
