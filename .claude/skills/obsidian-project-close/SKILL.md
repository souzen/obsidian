---
name: obsidian-project-close
description: >
  Closes (archives) an Obsidian project: stamps date-resolved in the project file, moves it to
  the archive/<current year>/ folder, removes its card from the kanban board and adds a card to
  the yearly archive board. Use this skill whenever the user says "close project", "archive project",
  "zamknij projekt", "zarchiwizuj projekt", "project is done", "projekt skończony", or gives a project
  name and asks to move it to the archive. Works for both personal (my/) and work projects.
  Always use this skill — not ad-hoc file moves — when archiving a project note in the vault.
---

# Obsidian Close Project

Archives a finished project and keeps the kanban boards in sync with the files.

**Vault root:** `/Users/lsosnicki/obsidian/vault`

| Scope | Project dir | Kanban board | Archive dir |
|---|---|---|---|
| personal (`my`) | `my/projects/` | `my/personal-kanban.md` | `my/archive/<year>/` |
| work (`work`) | `work/projects/` | `work/inpost-kanban.md` | `work/archive/<year>/` |

**Archive board** for the year: `my/archive/<year>/personal-projects-archive-<year>.md` or `work/archive/<year>/inpost-projects-archive-<year>.md`.

Today's date and `<year>` come from the environment's current date — never guess.

---

## Phase 1 — Resolve the project

Input: project name from the user.

```
ls /Users/lsosnicki/obsidian/vault/my/projects/
ls /Users/lsosnicki/obsidian/vault/work/projects/
```

Match by filename (case-insensitive, ignore dashes/spaces/underscores).

- No match → stop and tell the user; suggest the closest names.
- Several matches (including the same name in both `my` and `work`) → ask the user to pick.
- One match → remember `scope` (`my` or `work`), `name` (filename without `.md`) and `path`.

Read the file.

## Phase 2 — Pre-flight checks (nothing is written yet)

1. **Target free?** `<scope>/archive/<year>/<name>.md` must not exist. If it does → stop, never overwrite.
2. **Already closed?** If the frontmatter already has `date-resolved` → stop and ask the user.
3. **Unchecked next actions.** Find `- [ ]` items in the file. If any → show them and warn:
   "Project has N unchecked next actions: … Resolve them in the file, then tell me to continue — or confirm to close as is."
   Do **not** edit them — the user fixes them manually. Wait for the answer.
4. **Kanban card.** Find `[[<name>]]` in the scope's kanban board; note the lane (`## ...`) it is in. Not found → continue, but mention it in the report.
5. **Archive board.** Check whether the year's archive board exists.
   - `my`: use the single existing lane.
   - `work`: list the board's lanes (`## ...`) and **ask the user which lane** gets the card, offering "new lane" as an option.
   - Board missing → it will be created (see Phase 3, step 4).

Present the plan and wait for an explicit yes:

```
Closing: <name> (<scope>)
1. <path>: add date-resolved: YYYY-MM-DD
2. move → <scope>/archive/<year>/<name>.md
3. remove card from <board> (lane: <lane>)
4. add card to <archive board> (lane: <lane>)
```

## Phase 3 — Execute (after approval)

1. **Stamp the date.** Add `date-resolved: YYYY-MM-DD` to the frontmatter, right after `date-added` if it exists, otherwise as the last key. If the file has no frontmatter, create one. Use Edit — don't rewrite the file.
2. **Move.** `mkdir -p <scope>/archive/<year>` then `mv` the file there (plain `mv`, no `-f`).
3. **Remove the card** from the kanban board: delete exactly the line containing `[[<name>]]` (Edit). Keep the blank-line spacing between lanes intact. Don't touch other cards or the `%% kanban:settings` block.
4. **Add the card** `- [ ] [[<name>]]` at the end of the chosen lane in the archive board (Edit).
   - Board missing → create it by copying the header and the `%% kanban:settings %%` block from the previous year's board (or the current board if no previous one exists), with the chosen lane and no other cards.
   - New lane → add `## <lane name>` after the last lane, before the `%% kanban:settings` block.

## Phase 4 — Verify and report

Check with `ls` and `grep`:
- file exists in the archive dir and is gone from the project dir
- `date-resolved` is present in the moved file
- `[[<name>]]` is gone from the kanban board and present exactly once in the archive board

Report in chat: old → new path, date-resolved value, removed card (+ lane), added card (+ lane), plus any warnings (unchecked actions, card not found on board).

## Rules

- Never overwrite an existing file; never delete a project file.
- Don't change the project's content beyond the `date-resolved` property.
- Don't touch journal entries or links in other notes — `[[name]]` links keep working because the filename doesn't change.
- Write nothing before the user approves the plan in Phase 2.
