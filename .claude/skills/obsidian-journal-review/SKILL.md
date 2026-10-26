---
name: obsidian-journal-review
description: >
  Use when journal entries are tagged #claude-review, or the user says "review journal",
  "przejrzyj dziennik", "process journal entries", "move decisions and next steps to projects",
  "przenieś decyzje i next steps do projektów". Runs unattended on weekdays (before
  obsidian-journal-plan-my-day and hourly), so it never asks questions.
---

# obsidian-journal-review

For every journal entry tagged `#claude-review`: copies its **Decisions** into the Log of the linked project or person, moves its **Next Steps** out of the entry into the project's Next Actions or the person's Talking points, then marks the entry `#claude-reviewed`. No language correction, no day summary. Unattended: never ask, report what can't be done.

**Journal dir:** `/Users/lsosnicki/obsidian/vault/journal/` (skip `archive/`)
**Projects:** `/Users/lsosnicki/obsidian/vault/work/projects/<name>.md`, then `/Users/lsosnicki/obsidian/vault/my/projects/<name>.md`
**People:** `/Users/lsosnicki/obsidian/vault/work/areas/_4_people/@<handle>.md`

**Rules**
- Edit **in place** with the Edit tool; never rewrite a whole file.
- Only change: the journal heading's tag and hour, the entry's `Next Steps` that were moved to the project or person (Steps 2c, 3c), the project's `Log` and `Next Actions`, the person's `Log` and `Talking points`, the f2f person's `[[@handle]]` links below the heading of their entry.
- Never touch entries tagged `#claude-reviewed`.

---

## Step 1 — Find marked entries

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

Delete from the entry's `Next Steps` every item that is now in the project — just appended or already there — together with its indented sub-items.

- Items that could not be moved stay in the entry.
- If no item is left, delete the whole `Next Steps` sub-section: its heading and the blank lines after it.
- `Decisions` and `Overview` are never removed.

---

## Step 3 — If one person is linked in the heading (f2f)

**Person:** a `[[@handle]]` in the heading. Applies only when Step 2 found no project and exactly one person is linked. Otherwise skip this step.

**Person file:** `@handle.md` in the People dir. Missing → skip 3a–3c, note for the report. **Title** and **Date:** as in Step 2.

### 3a — Decisions → person's Log

If the entry has `Decisions`, add this block as the **first** entry of the person file's `Log` section (heading matched by keyword, any level), directly under the heading and any intro line, above older entries:

```markdown
### Decisions - <Title> [[<YYYY-MM-DD>]]

- <decision>

```

- Bullets copied unchanged.
- Heading one level deeper than the `Log` heading (`###` under `## Log`).
- Same heading already there → skip. No `Log` section → skip 3a, note for the report.

### 3b — Next Steps → person's Talking points

If the entry has `Next Steps`, append its items at the **bottom of the person file's `Talking points` section** (heading matched by keyword, any level), as top-level items exactly as written, except that every `[[@handle]]` of this person is removed (trim the leftover spaces). Other links stay.

- Item already in the section (trimmed, after removing the handle) → skip it. No `Talking points` section → skip 3b and 3c, note for the report.

### 3c — Remove the moved Next Steps from the entry

Same as 2c, for the items now in the person file.

### 3d — Unlink the handle in the entry

In the entry's sub-sections replace each remaining `[[@handle]]` with `@handle`. The `[[@handle]]` in the heading stays linked, as do all other links.

---

## Step 4 — Flip the tag

For every entry, whatever Steps 2–3 did, edit the heading: `#claude-review` → `#claude-reviewed`, and remove the hour. `### magento / presta (11:30–11:55) [[integration-plugins]] #claude-review` → `### magento / presta [[integration-plugins]] #claude-reviewed`. The rest of the entry stays untouched.

---

## Step 5 — Report

```
✅ Journal review: N entries reviewed (N project, N f2f, N other)
- 2026-10-19 · <Title> → [[project]] — decisions: added / already there / none / no Log section · next steps: N moved (N added, N already there; removed from the entry) / none / no Next Actions section (kept in the entry)
- 2026-10-19 · <Title> — f2f with @person: decisions: added / already there / none / no Log section · next steps: N moved to Talking points (N added, N already there; removed from the entry) / none / no person file / no Talking points section (kept in the entry) · N links unlinked
- 2026-10-19 · <Title> — nothing moved: <no project or person linked / several linked>
```
