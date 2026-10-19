---
name: obsidian-journal-review
description: >
  Processes journal entries marked with the tag #claude-review: copies the entry's Decisions into the
  Log of the project linked in the entry title, appends its Next Steps to that project's Next Actions,
  and changes the tag to #claude-reviewed. Use this skill whenever the user says "review journal",
  "przejrzyj dziennik", "process journal entries", "move decisions and next steps to projects",
  "przenieś decyzje i next steps do projektów", or asks to handle journal entries tagged #claude-review.
  Runs unattended every weekday morning (before obsidian-plan-my-day), so it never asks questions.
---

# obsidian-journal-review

For every journal entry tagged `#claude-review`, pushes its **Decisions** and **Next Steps** into the project linked in the entry title, then marks the entry `#claude-reviewed`. No language correction, no day summary.

**Journal dir:** `/Users/lsosnicki/obsidian/vault/journal/` (skip `archive/`)
**Projects:** `/Users/lsosnicki/obsidian/vault/work/projects/`, then `/Users/lsosnicki/obsidian/vault/my/projects/`

Unattended: never ask. If something can't be done, leave the tag as is and report it.

---

## Step 1 — Find marked entries

`grep -n -E '#claude-review($|[^a-zA-Z])' /Users/lsosnicki/obsidian/vault/journal/*.md` — `#claude-reviewed` does not match (a letter follows `review`).

An **entry** is the `###` heading line that carries the tag, plus everything below it up to the next heading of the same or higher level. Its sub-sections are `Overview`, `Next Steps` and `Decisions` (`####`, matched by keyword).

No marked entries → say so and stop.

---

## Step 2 — Process each entry

Read the journal file first. Then for each entry:

### 2a — Title, date, project

- **Title:** the heading text without any hour — remove time ranges and single times in any form (`(11:30–11:55)`, `11:30-11:55`, `(09:30)`, `[09:30]`) — and without the `[[links]]` and the tag; trim leftover spaces and dashes; first letter capitalised. Example: `magento / presta / woo plugins - status (11:30–11:55) [[integration-plugins]] #claude-review` → `Magento / presta / woo plugins - status`.
- **Date:** from the journal filename (`YYYY-MM-DD.md`).
- **Project:** the `[[...]]` links in the heading that match a project file (`work/projects/<name>.md`, else `my/projects/<name>.md`). People (`[[@…]]`) and teams (`[[~…]]`) are not projects. Exactly one project is required — none or several → skip the entry, leave the tag, report why.

### 2b — Decisions → project Log

If the entry has a `Decisions` sub-section, read the project file and add this block as the **first** entry in its `### Log` section (match the heading by keyword `Log`, directly under the section's intro line, above older entries):

```markdown
#### Decisions - <Title> [[<YYYY-MM-DD>]]

- <decision>
- <decision>

```

- Bullets are copied unchanged.
- Already there (same `#### Decisions - <Title> [[<YYYY-MM-DD>]]` heading) → skip, nothing to add.
- Project has no `Log` section → skip the entry, leave the tag, report.

### 2c — Next Steps → project Next Actions

If the entry has a `Next Steps` sub-section, append its items at the **bottom of the project's `### Next Actions` list** (after the last item, before the next heading), as top-level items exactly as written (`- [ ] … [[@assignee]]`).

- An item whose text is already in that section (compared after trimming) → skip it.
- Project has no `Next Actions` section → skip the entry, leave the tag, report.

### 2d — Flip the tag and drop the hour from the journal heading

Only when 2b and 2c finished (or had nothing to do), edit the journal heading line:
- replace `#claude-review` with `#claude-reviewed`;
- remove the hour from the heading (any form, same as in 2a: `(11:30–11:55)`, `11:30-11:55`, `(09:30)`, …), keeping the title, the `[[links]]` and the tag. Example: `### magento / presta (11:30–11:55) [[integration-plugins]] #claude-review` → `### magento / presta [[integration-plugins]] #claude-reviewed`.

The rest of the journal entry is left untouched — the journal keeps the full record.

---

## Rules

- Edit **in place** with the Edit tool; never rewrite a whole file (it can silently alter other lines).
- Only change: the tag and the hour in the journal heading, the project's `Log` and `Next Actions` sections.
- Never touch entries already tagged `#claude-reviewed`.

---

## Step 3 — Report

```
✅ Journal review: N entries processed, N skipped
- 2026-10-19 · <Title> → [[project]] — decisions: added / already there / none · next steps: N added, N already there / none
- 2026-10-19 · <Title> — skipped: <reason>
```
