---
name: obsidian-project-outcomes-review
description: >
  Reviews an Obsidian project file across three dimensions: language quality,
  outcomes quality, and next actions relevance. Use this skill whenever the user
  wants to review a project file, says things like "review my project", "check project file",
  "sprawdź projekt", "przejrzyj projekt", "review project in obsidian", "audit project file",
  "check outcomes", "check my next actions", "wygeneruj outcome dla projektu",
  "wygeneruj outcome", "generate outcome for project", or provides a path to a project
  .md file and asks for review or improvement. Also trigger when the user asks to
  improve, validate, upgrade, or generate/write an outcome for a project note in
  Obsidian. Always use this skill — not generic file-reading — when the request
  involves reviewing or generating outcomes for a project file in the vault.
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

Use Bash:
```
ls /Users/lsosnicki/obsidian/vault/work/projects/
ls /Users/lsosnicki/obsidian/vault/my/projects/
```

Match by filename (case-insensitive, ignore dashes/spaces). If ambiguous → ask the user to pick.

Read the file with the Read tool: `<resolved_path>`

Extract, for use in Phase 3:
- Any `[[...]]` backlinks in the file **body**
- Any `[[...]]` links in **frontmatter** fields (`owner`, `co-owner`, `EM`, `PM`, `SEM`, `members`, `team`)

### Step 0b — Quick check for a project-level status signal

Before evaluating anything, do a cheap check for whether this project has been explicitly paused/stopped/deprioritized:

`grep -il "<project-name>\|\[\[<project-name>\]\]" journal/*.md`

Skim any matches for stop/pause language (e.g. "stopujemy", "wstrzymujemy", "zawieszamy", "on hold", "pausing", "deprioritized"). This is a fast pre-check, not the full journal read (that happens in Phase 3b-1) — its only purpose is to carry a `paused: yes/no + source` flag into Phase 2, since Phase 2 runs before Phase 3.

If found → carry this forward and surface it before Phase 2's evaluation (see Step 2a) and again as the headline of Phase 3 (see Step 3c).

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

Then **auto-save** the corrected file using the Write tool.

Confirm: `✅ Language corrections saved to <filename>`

---

## Phase 2 — Outcomes Review

**Goal:** Ensure the `### Outcomes` section exists and follows the Outcomes over Outputs principle.

Read the full reference before this phase:
→ See `references/outcomes-over-outputs.md`

### Step 2a — Check section existence

Look for a `### Outcomes` section (also accept `### Cele`, `### Wyniki`, `### Rezultaty`).

**If section is missing, OR exists but contains only placeholder text (`todo`, `TBD`, `outcomes`, or empty):**
> "This project has no Outcomes section yet. I'll help you create one — see the questions below."
Jump to Step 2c.

**If section exists with real content:**
→ Read its contents and proceed to Step 2b.

**If Step 0b found a project-level pause/stop signal**, say so before evaluating anything: "Note: the journal indicates this project may be paused (<source>) — evaluating the Outcomes below anyway, but flag if this project should be archived instead of reviewed." Do not silently validate outcomes for a project the journal says is on hold.

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

On "yes" → update file with the Write tool and confirm.  
On "edit first" → iterate on the draft in chat.  
On "skip" → move to Phase 3 without saving.

---

## Phase 3 — Next Actions Review

**Goal:** Cross-reference tasks against journal and related files to surface stale, duplicate, or missing actions.

### Step 3a — Read current next actions, decisions, and questions

Find `### Next Actions` (also accept `### Następne kroki`, `### TODO`, `### Tasks`), `### Decisions`, and `### Questions`.

Extract:
- `- [ ]` open tasks, `- [x]` completed tasks (for context only)
- Open decisions/questions from those sections

**Intra-file consistency check:** compare `### Next Actions`, `### Questions`, and `### Decisions` against each other for content duplicated across sections within this same file (e.g. the same question listed both as an open Next Action and under Questions). Flag any duplication found — this is separate from the journal/backlink cross-referencing below.

### Step 3b — Gather context from vault

Run these lookups in parallel:

**3b-1 — Recent journal entries (last 7 calendar days)**

Use Bash `ls /Users/lsosnicki/obsidian/vault/journal/`. Read whichever journal files fall in the last 7 calendar days (the vault may not have a file for every day — read what exists in that window, don't assume 7 contiguous files). Search for:
- Mentions of this project (by name, filename, or `[[wikilink]]`)
- Any `- [ ]` tasks that reference this project
- Any decisions or blockers logged in meetings
- **A project-level status signal**: an explicit decision to pause, stop, deprioritize, or archive the whole project (distinct from a single task update) — if Step 0b already found one, confirm/expand on it here

**3b-2 — Backlinked files**

For each `[[link]]` found in the project file body **and** each `[[@handle]]`/`[[~team]]` link found in frontmatter (Phase 0), use the Read tool on `/Users/lsosnicki/obsidian/vault/<resolved_link>.md`.
Look for tasks or updates that relate back to this project.

### Step 3c — Analyze and propose upgrades

**If a project-level pause/stop/deprioritize signal was found (Step 0b or 3b-1), lead with it** — before the per-task diff, as its own headline, e.g.:
```
🛑 Project-level signal: journal/2026-08-19.md indicates work on this project has been stopped
   (message from [[@mochrymowicz]]). Consider archiving this project instead of reviewing
   its Next Actions below.
```
Do not bury this as a "Priority signal" bullet inside the task diff — it changes the meaning of everything below it.

Then compare gathered context against current `### Next Actions`. Identify:

| Finding | Example |
|---|---|
| **Stale task** | Task was already discussed and resolved in journal, but still open |
| **Duplicate task (cross-file)** | Same task appears in a related/backlinked file |
| **Duplicate content (same file)** | Same question/task appears in both Next Actions and Questions/Decisions (from Step 3a) |
| **Missing task** | Journal mentions a follow-up that hasn't been added to the project |
| **Outdated task** | Decision in journal changed the scope or made the task irrelevant |
| **Priority signal** | Journal mentions a blocker or deadline that should be reflected (task-level, not project-level) |

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

On "yes" → apply all changes and save with the Write tool.  
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
| No `### Outcomes` section, or section has only placeholder text (`todo`, `TBD`, empty) | Skip directly to interview (Step 2c) |
| No `### Next Actions` section | Note it, offer to create the section |
| User says "generate outcome for X" / "wygeneruj outcome dla projektu X" | Treat as "only check outcomes" — Phase 2 only |
| Project is a personal project (`my/projects/`, tags like `firearms`, `mind-n-body`, hobby-related) | In Step 2c, swap the "business impact" question for "Why does this matter to you personally — safety, skill, health, relationships?"; don't require a business framing |
| Journal dir empty / no recent entries | Skip journal lookup, continue with backlinks |
| Backlinked file not found on disk | Skip with a note: "[[X]] not found in vault" |
| File is in English only | Skip Polish diacritics check |
| User says "only check language" | Run Phase 0, then Phase 1 only |
| User says "only check outcomes" | Run Phase 0, then Phase 2 only |
| User says "only check tasks" | Run Phase 0, then Phase 3 only |

*Phase 0 (including Step 0b's pause-signal check) always runs first regardless of which phase(s) the user scoped to — it's not one of the "three phases," it's the shared setup they all depend on.*
