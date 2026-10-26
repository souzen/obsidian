---
name: obsidian-journal-review
description: >
  Processes journal entries marked with the tag #claude-review: copies the entry's Decisions into the
  Log of the project linked in the entry title, moves its Next Steps to that project's Next Actions
  (removing them from the entry), and changes the tag to #claude-reviewed. Use this skill whenever the user says "review journal",
  "przejrzyj dziennik", "process journal entries", "move decisions and next steps to projects",
  "przenieś decyzje i next steps do projektów", or asks to handle journal entries tagged #claude-review.
  Runs unattended every weekday morning (before obsidian-journal-plan-my-day), so it never asks questions.
---

# obsidian-journal-review

For every journal entry tagged `#claude-review`: with a linked project, copies its **Decisions** into the project Log and moves its **Next Steps** into the project's Next Actions (removing them from the entry); for an f2f entry, unlinks the person's handle; then marks the entry `#claude-reviewed`. No language correction, no day summary. Unattended: never ask, report what can't be done.

**Journal dir:** `/Users/lsosnicki/obsidian/vault/journal/` (skip `archive/`)
**Projects:** `/Users/lsosnicki/obsidian/vault/work/projects/`, then `/Users/lsosnicki/obsidian/vault/my/projects/`

**Rules**
- Edit **in place** with the Edit tool; never rewrite a whole file.
- Only change: the tag and hour in the journal heading, the entry's `Next Steps` that were moved to the project (Step 2c), the project's `Log` and `Next Actions`, the f2f person's `[[@handle]]` links below the heading of their entry.
- Never touch entries tagged `#claude-reviewed`.

---

## Step 1 — Find marked entries and iterate

`grep -n -E '#claude-review($|[^a-zA-Z])' /Users/lsosnicki/obsidian/vault/journal/*.md` (`#claude-reviewed` does not match).

An **entry** is the `###` heading carrying the tag plus everything below it up to the next heading of the same or higher level. Its sub-sections are `Overview`, `Next Steps`, `Decisions` (`####`, matched by keyword). **Hour** in a heading means any time form: `(11:30–11:55)`, `11:30-11:55`, `(09:30)`, `[09:30]`.

No marked entries → say so and stop. Otherwise read each journal file and run Steps 2–4 for every entry in it.

---

## Step 2 — If one project is linked in the heading

**Project:** a `[[...]]` in the heading matching `work/projects/<name>.md`, else `my/projects/<name>.md`. People (`[[@…]]`) and teams (`[[~…]]`) are not projects. None or several → skip this step.

- **Title:** the heading without hour, `[[links]]` and tag; trim leftover spaces and dashes; capitalise the first letter. `magento / presta (11:30–11:55) [[integration-plugins]] #claude-review` → `Magento / presta`.
- **Date:** from the journal filename (`YYYY-MM-DD.md`).

### 2a — Decisions → project Log

If the entry has `Decisions`, add this block as the **first** entry of the project's `Log` section (heading matched by keyword, directly under its intro line, above older entries):

```markdown
### Decisions - <Title> [[<YYYY-MM-DD>]]

- <decision>

```

- Bullets copied unchanged.
- Heading level `###` under `## Log`; `####` if the project's Log is `### Log`.
- Same heading already there → skip. No `Log` section → skip 2a, note for the report.

### 2b — Next Steps → project Next Actions

If the entry has `Next Steps`, append its items at the **bottom of the project's `## Next Actions` list**, as top-level items exactly as written (`- [ ] … [[@assignee]]`).

- Item already in the section (trimmed) → skip it. No `Next Actions` section → skip 2b and 2c, note for the report.

### 2c — Remove the moved Next Steps from the entry

After 2b, delete from the entry's `Next Steps` every item that is now in the project — both the ones just appended and the ones that were already there — together with its indented sub-items.

- Items that could not be moved stay in the entry.
- If no item is left, delete the whole `Next Steps` sub-section: its heading and the blank lines after it.
- `Decisions` and `Overview` are never removed — only copied (Decisions) or left as is.

---

## Step 3 — If one person is linked in the heading (f2f)

**Person:** a `[[@handle]]` in the heading. Applies only when Step 2 found no project and exactly one person is linked. Otherwise skip this step.

### 3a — Unlink the handle

In the entry's sub-sections (everything below the heading) replace each `[[@handle]]` with `@handle`. The `[[@handle]]` in the heading stays linked, as do all other links. Nothing is written to the person note.

---

## Step 4 — Flip the tag

For every entry, whatever Steps 2–3 did, edit the heading: `#claude-review` → `#claude-reviewed`, and remove the hour. `### magento / presta (11:30–11:55) [[integration-plugins]] #claude-review` → `### magento / presta [[integration-plugins]] #claude-reviewed`. The rest of the entry stays untouched.

---

## Step 5 — Report

```
✅ Journal review: N entries reviewed (N project, N f2f, N other)
- 2026-10-19 · <Title> → [[project]] — decisions: added / already there / none / no Log section · next steps: N moved (N added, N already there; removed from the entry) / none / no Next Actions section (kept in the entry)
- 2026-10-19 · <Title> — f2f with @person: N links unlinked
- 2026-10-19 · <Title> — nothing moved: <no project or person linked / several linked>
```
