---
name: obsidian-review-project
description: >
  Reviews an Obsidian project file across three dimensions: language quality,
  outcomes quality, and next actions relevance. Use this skill whenever the user
  wants to review a project file, says things like "review my project", "check project file",
  "sprawdź projekt", "przejrzyj projekt", "review project in obsidian", "audit project file",
  "check outcomes", "check my next actions", or provides a path to a project .md file and
  asks for review or improvement. Also trigger when the user asks to improve, validate,
  or upgrade a project note in Obsidian. Always use this skill — not generic file-reading —
  when the request involves reviewing a project file in the vault.
---

# Obsidian Review Project

Reviews a project `.md` file in the Obsidian vault across three sequential phases.

**Vault root:** `/Users/lsosnicki/obsidian/vault`  
**Project dirs:** `work/projects/` and `my/projects/`  
**Journal dir:** `journal/`

---

## Phase 0 — Resolve File Path

If the user provided a full path → use it directly.  
If the user provided a project name → search for a matching file:

```
Filesystem:list_directory /Users/lsosnicki/obsidian/vault/work/projects/
Filesystem:list_directory /Users/lsosnicki/obsidian/vault/my/projects/
```

Match by filename (case-insensitive, ignore dashes/spaces). If ambiguous → ask the user to pick.

Read the file:
```
Filesystem:read_text_file <resolved_path>
```

Extract from frontmatter:
- `tags` → used in Phase 3 for context search
- Any backlinks `[[...]]` in the file body → note them for Phase 3

---

## Phase 1 — Language & Style Review

**Goal:** Fix language errors and save automatically (no confirmation needed).

### What to check
- Typos, spelling, grammar (mixed PL/EN is expected — do not translate between languages)
- Missing Polish diacritics: `ą ę ó ś ź ż ć ń ł`
- Style: clarity, conciseness, sentence structure
- Consistency: terminology used consistently across sections

### What NOT to change
- Heading structure
- Task checkbox syntax `- [ ]` / `- [x]`
- Obsidian links `[[...]]`
- Frontmatter keys/values (except obvious typos in values)
- Intentional code-switching (a heading in PL with bullets in EN is fine)

### Output
Show the user a **diff-style summary** in chat:

```
## Phase 1 — Language Review

✅ Fixed N issues:
- Line 12: "managment" → "management"
- Line 34: "nastepny" → "następny"
- Line 41: Style: "we need to do X thing" → "Do X"
```

Then **auto-save** the corrected file using `Filesystem:write_file`.

Confirm: `✅ Language corrections saved to <filename>`

---

## Phase 2 — Outcomes Review

**Goal:** Ensure the `### Outcomes` section exists and follows the Outcomes over Outputs principle.

Read the full reference before this phase:
→ See `references/outcomes-over-outputs.md`

### Step 2a — Check section existence

Look for a `### Outcomes` section (also accept `### Cele`, `### Wyniki`, `### Rezultaty`).

**If section is missing:**
> "This project has no Outcomes section. I'll help you create one — see the questions below."
Jump to Step 2c.

**If section exists:**
→ Read its contents and proceed to Step 2b.

### Step 2b — Evaluate existing outcomes

For each outcome listed, check:

| Rule | Bad example | Good example |
|---|---|---|
| Behavioral change, not deliverable | "Deploy monitoring dashboard" | "On-call engineers detect incidents 30% faster" |
| Measurable or observable | "Better performance" | "P99 latency drops below 200ms" |
| User/customer/business focused | "Refactor auth module" | "Users no longer get logged out mid-session" |
| Not just a feature shipped | "Add export button" | "Finance team exports reports without dev help" |

Flag any outcome that looks like an **output** (deliverable, feature, task) rather than an outcome (behavioral change, measurable improvement).

Show the evaluation in chat:

```
## Phase 2 — Outcomes Review

⚠️ 2 of 3 outcomes look like outputs:
- "Deploy new CI pipeline" → this is a task, not an outcome
  Suggestion: "Developers merge PRs 2× faster with zero flaky builds"
- "Migrate DB to Postgres" → this is a deliverable
  Suggestion: "DB maintenance incidents drop to 0/month after migration"

✅ 1 outcome looks good:
- "Support team resolves L1 tickets without engineering help"
```

### Step 2c — Interview to create/improve outcomes

Ask targeted questions (max 3 at a time, conversationally):

1. **Who benefits?** "Who is the end user or stakeholder for this project?"
2. **What changes for them?** "What will they be able to do differently — or stop doing — when this project succeeds?"
3. **How would we know it worked?** "What would we observe or measure that tells us it worked?"
4. **What's the business impact?** "Does this reduce cost, increase revenue, reduce risk, or improve satisfaction?"

Based on answers, draft outcome candidates in this format:

> `[WHO] [can/will/no longer] [BEHAVIOR] [measurable signal]`

Example:
> `Engineers on-call can detect anomalies within 5 minutes instead of discovering them via customer complaints.`

### Step 2d — Propose changes

Present the proposed `### Outcomes` section in full. Ask:

> "Should I replace the Outcomes section with this? (yes / edit first / skip)"

On "yes" → update file with `Filesystem:write_file` and confirm.  
On "edit first" → iterate on the draft in chat.  
On "skip" → move to Phase 3 without saving.

---

## Phase 3 — Next Actions Review

**Goal:** Cross-reference tasks against journal and related files to surface stale, duplicate, or missing actions.

### Step 3a — Read current next actions

Find the `### Next Actions` section (also accept `### Następne kroki`, `### TODO`, `### Tasks`).

Extract:
- `- [ ]` open tasks
- `- [x]` completed tasks (for context only)

### Step 3b — Gather context from vault

Run these lookups in parallel:

**3b-1 — Recent journal entries (last 7 days)**

```
Filesystem:list_directory /Users/lsosnicki/obsidian/vault/journal/
```
Read the 7 most recent journal files. Search for:
- Mentions of this project (by name or filename)
- Any `- [ ]` tasks that reference this project
- Any decisions or blockers logged in meetings

**3b-2 — Files with same tags**

From frontmatter `tags` extracted in Phase 0:
```
Filesystem:search_files /Users/lsosnicki/obsidian/vault/work/ <tag>
Filesystem:search_files /Users/lsosnicki/obsidian/vault/my/ <tag>
```
Read matching files. Look for:
- Related tasks or decisions
- Duplicate tasks already captured elsewhere
- Context that changes the priority of existing tasks

**3b-3 — Backlinked files**

For each `[[link]]` found in the project file body:
```
Filesystem:read_text_file /Users/lsosnicki/obsidian/vault/<resolved_link>.md
```
Look for tasks or updates that relate back to this project.

### Step 3c — Analyze and propose upgrades

Compare gathered context against current `### Next Actions`. Identify:

| Finding | Example |
|---|---|
| **Stale task** | Task was already discussed and resolved in journal, but still open |
| **Duplicate task** | Same task appears in a related file |
| **Missing task** | Journal mentions a follow-up that hasn't been added to the project |
| **Outdated task** | Decision in journal changed the scope or made the task irrelevant |
| **Priority signal** | Journal mentions a blocker or deadline that should be reflected |

### Step 3d — Present proposed changes

Show a clear diff of the proposed `### Next Actions` section:

```
## Phase 3 — Next Actions Review

Context found in: journal/2026-04-23.md, rollout-200k.md

Proposed changes:
❌ Remove (resolved in journal 2026-04-22): "- [ ] Confirm deployment window with Rafał"
✅ Keep: "- [ ] Write runbook for rollback procedure"
➕ Add (from journal 2026-04-23): "- [ ] Follow up with Paweł N. on load test results"
🔄 Update: "- [ ] Review capacity" → "- [ ] Review capacity before 2026-05-01 (deadline from Łukasz)"
```

Ask:
> "Should I apply these changes to the Next Actions section? (yes / pick individually / skip)"

On "yes" → apply all changes and save with `Filesystem:write_file`.  
On "pick individually" → present each change as a y/n confirmation.  
On "skip" → do not modify, just summarize findings in chat.

---

## Final Summary

After all three phases, output a summary:

```
## ✅ Project Review Complete: <filename>

Phase 1 — Language:     Fixed N issues, auto-saved
Phase 2 — Outcomes:     Updated / No changes / Skipped
Phase 3 — Next Actions: N changes applied / Skipped

📁 File: <full path>
```

---

## Edge Cases

| Situation | Behaviour |
|---|---|
| File not found | Ask user to confirm path or pick from list |
| No `### Outcomes` section | Skip directly to interview (Step 2c) |
| No `### Next Actions` section | Note it, offer to create the section |
| Journal dir empty / no recent entries | Skip journal lookup, continue with tags + backlinks |
| Tag search returns 0 results | Skip, note in summary |
| Backlinked file not found on disk | Skip with a note: "[[X]] not found in vault" |
| File is in English only | Skip Polish diacritics check |
| User says "only check language" | Run Phase 1 only, skip 2 and 3 |
| User says "only check outcomes" | Run Phase 2 only |
| User says "only check tasks" | Run Phase 3 only |
