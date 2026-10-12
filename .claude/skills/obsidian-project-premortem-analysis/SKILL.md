---
name: obsidian-project-premortem-analysis
description: >
  Runs a premortem for an Obsidian project: imagine the project has already failed,
  surface the reasons why, rank them by likelihood and impact, and turn the top risks
  into mitigations saved in the project file. Use this skill whenever the user says
  "premortem", "pre-mortem", "zrób premortem", "premortem dla projektu", "what could go wrong
  with project X", "co może pójść nie tak w projekcie", "analiza ryzyk projektu", "risk review
  for project", or provides a project .md file and asks to anticipate failure or identify
  risks before/early in the work. Always use this skill — not generic risk brainstorming —
  when the request is a premortem or risk analysis for a project note in the vault.
---

# Obsidian Project Premortem Analysis

Based on Gary Klein's premortem technique (source note: `inbox/premortem-analysis-anticipate-failure.md`). Instead of asking "what could go wrong?", assume the project **has already failed** and work backward to explain why. This "prospective hindsight" makes risks easier to name and counters optimism bias.

The skill adapts the team workshop to a solo setting: Claude pre-fills candidate failure reasons from vault evidence, the user adds their own, then both are ranked and turned into mitigations.

**Vault root:** `/Users/lsosnicki/obsidian/vault`
**Project dirs:** `work/projects/` and `my/projects/`
**Journal dir:** `journal/`

---

## Phase 0 — Resolve and Read the Project

If the user gave a full path → use it. If they gave a name → `ls` both project dirs, match case-insensitively ignoring dashes/spaces; if ambiguous, ask the user to pick.

Read the project file and extract:
- `### Outcomes` (also `### Cele`, `### Wyniki`, `### Rezultaty`) — this defines what "success" means, so what "failure" means
- `### Next Actions` (also `### Następne kroki`, `### TODO`, `### Tasks`), `Decisions`, `Questions`, `Issues` (flat `###` or nested `####` under `### Log`)
- Frontmatter people/teams (`owner`, `co-owner`, `EM`, `PM`, `SEM`, `members`, `team`) and body `[[...]]` links
- An existing `Premortem` subsection (normally `#### Premortem` under `### Log`; also accept a flat `###` one), if any

**If Outcomes is missing or placeholder (`todo`, `TBD`, empty):** a premortem needs a definition of success. Tell the user, and ask for one sentence: "What does success look like for this project?" Offer to run `obsidian-project-outcomes-review` for a proper outcome. Do not proceed on an undefined goal.

**If a Premortem subsection already exists:** show it and ask: "update it (re-rank, add new risks) or start over?" Never silently overwrite it.

---

## Phase 1 — Gather Evidence (run in parallel)

- **Journal:** `grep -il` the project name / `[[wikilink]]` in `journal/*.md` (skip `journal/archive/` unless the project is old). Read matches and look for blockers, delays, disagreements, unanswered questions, stop/pause language, and dependencies on other teams.
- **Linked people/teams:** read `[[@handle]]` and `[[~team]]` notes for open items touching this project (capacity, ownership conflicts, other priorities).
- **Related projects:** read backlinked project files for dependencies or overlapping scope.
- **Project's own file:** stale tasks, long-open questions, undecided decisions, and "todo" placeholders are risk signals in themselves.

Skip missing files with a one-line note; don't block on them.

---

## Phase 2 — Set the Scene

Pick the time horizon: ask the user, or default to **6 months** ("9 months" for projects with a long rollout). Then state the scene in chat, in the project's language (PL/EN, never translate):

> "Wyobraź sobie, że jest <date + horizon>. Projekt **<name>** zakończył się całkowitą porażką — outcome '<first outcome>' nie został osiągnięty. Co poszło nie tak?"

---

## Phase 3 — Generate Failure Reasons

Do this in **two rounds, user first** — the technique relies on independent thinking before anything anchors the user.

**Round 1 — user's own reasons.** Ask the user to list their failure reasons first, one per line, without filtering ("write what comes to mind, 2 minutes"). Do not show Claude's candidates yet. If the user says "you go first" or wants to skip, proceed to Round 2 and flag that the list is Claude-anchored.

**Round 2 — Claude's candidates from evidence.** Add failure reasons the user did *not* mention, each tied to evidence where it exists:

```
- <failure reason> — evidence: journal/2026-09-02.md ("<short quote>")
- <failure reason> — no evidence in vault, general pattern for this kind of project
```

Cover these angles so blind spots show up: **people & ownership** (capacity, key-person dependency, unclear owner), **dependencies** (other teams, vendors, platform), **scope & requirements** (unclear outcome, scope creep, wrong problem), **technical** (migration, performance, integration), **adoption** (users/teams don't switch), **timing** (freezes, competing priorities, deadlines), **communication** (stakeholders not aligned, decisions unmade). Aim for 6–12 reasons total; drop angles with nothing credible instead of padding.

Mark each reason's source: `👤 user`, `🤖 claude (evidence)`, or `🤖 claude (general)`.

**Round 3 — extra lenses (offer, don't force).** After the lists are merged, ask: "Want a few extra angles before we rank? (yes / skip)". If yes, pick the **2–3 lenses that fit this project** from the list below — not all of them — and put each as a short, concrete question or finding. Feed anything the user confirms back into the list as new reasons (marked `🤖 lens: <name>`).

| Lens | What Claude does |
|---|---|
| **Assumptions audit** | Lists 3–5 things the plan silently assumes (a team has capacity, a dependency ships on time, users will switch) and asks "which of these have you actually verified?" Unverified assumptions become risks. |
| **Stakeholder views** | For each person/team linked in the project, one line: how could *they* cause or experience failure (e.g. "[[~platform]] has a freeze in Q4", "[[@x]] is owner on 3 other projects")? Ask the user to correct. |
| **Inverse: premortem of success** | "It's 6 months later and this wildly succeeded — what had to be true?" Anything that had to be true but isn't yet planned is a gap. |
| **Second-order effects** | What breaks or gets worse elsewhere if this *succeeds* (more load on a team, a legacy path left unowned, a new thing to maintain)? |
| **Outside view** | How do similar projects in the vault (`work/projects/`, `work/archive/`) usually go wrong or slip? Cite files; if none, say so. |
| **Single points of failure** | Which one person, service, decision, or approval does everything hang on? |
| **Kill criteria** | "What signal would make us stop or change course?" Propose 1–2 concrete triggers (metric, date, decision) to record as watch signals. |
| **Silent-failure check** | Which failures would be invisible until late (no alert, no owner, nobody asks)? These are the dangerous ones — they get a *detect* mitigation. |

Choosing lenses: assumptions audit and single points of failure are good defaults for most projects; stakeholder views for cross-team work; outside view when archived similar projects exist; second-order effects for migrations and rollouts; kill criteria for long or expensive projects.

---

## Phase 4 — Consolidate and Rank

Merge duplicates into single reasons. Group related ones. Then score each:

| Reason | Likelihood (H/M/L) | Impact (H/M/L) | Priority |
|---|---|---|---|

Priority = High if both H, or one H and one M; Medium for M/M or H/L; Low otherwise. Propose the scores, then ask the user to adjust — their judgement overrides Claude's.

Show the table in chat, sorted by priority.

---

## Phase 5 — Mitigations

For every **High** and **Medium** risk, propose one concrete action in one of three forms:

- **Prevent** — change the plan now so the failure can't happen
- **Detect** — an early signal that tells you it's happening (and who checks it, when)
- **Decide** — a question/decision that has to be answered by a date

Phrase each as a task the user could do next, with an owner `[[@handle]]` when the vault makes one obvious. Low risks get no action — list them as "accepted".

Present the mitigations and ask:

> "Which mitigations should I add to `### Next Actions`? (all / pick individually / none)"

---

## Phase 6 — Save

Show the full proposed `#### Premortem` subsection, then ask: "Save to the project file? (yes / edit first / skip)". On "yes", write with the Write tool.

**Placement:** a `#### Premortem` subsection nested under `### Log %% fold %%`, alongside `#### Decisions` / `#### Questions` / `#### Issues`. Append it after the last existing subsection of the Log. If the project file has no `### Log` section (older files), create `### Log  %% fold %%` at the end of the file and put the subsection there. Do not touch other sections' content except appending confirmed mitigations to `### Next Actions` as `- [ ]` items prefixed so they are traceable, e.g. `- [ ] (premortem) <action>`.

**Format:**

```markdown
#### Premortem

*Run: YYYY-MM-DD · horizon: 6 months · failure = <outcome not achieved>*

| Risk | L | I | Priority | Mitigation |
|---|---|---|---|---|
| <failure reason> | H | H | High | <prevent/detect/decide action, or link to the Next Action> |
| <failure reason> | M | L | Low | accepted |

**Watch signals:** <1–3 early warning signs worth checking in future reviews>
**Revisit:** <date, e.g. 4–6 weeks from now or at the next milestone>
```

Keep it terse — it is a reference table, not an essay. Match the project's language per cell; do not translate.

Do **not** add a `#claude-reviewed` tag (that is for journal files only).

---

## Final Summary

```
## ✅ Premortem Complete: <filename>

Risks identified:  N (H: x, M: y, L: z)
Mitigations added to Next Actions: N
Premortem section: Created / Updated / Skipped
Suggested revisit: <date>

📁 File: <full path>
```

---

## Edge Cases

| Situation | Behaviour |
|---|---|
| Project not found | Ask the user to confirm the path or pick from the list |
| No Outcomes / placeholder only | Ask for one-sentence success definition; offer `obsidian-project-outcomes-review` (see Phase 0) |
| Existing Premortem subsection | Ask update vs. start over; never silently overwrite |
| Journal says project is paused/stopped | Surface it first: "journal indicates this project is paused (<source>) — run the premortem anyway or archive?" |
| Project nearly done / all tasks `- [x]` | Warn that a premortem is most valuable early; offer a lighter "what could still go wrong before closing" pass |
| Personal project (`my/projects/`) | Same flow; risk angles shift to time, motivation, money, safety, health instead of teams/dependencies. Don't use business framing |
| Very little evidence in vault | Say so, rely on the user's Round 1 list, and label Claude's reasons `general` |
| User wants to run it as a team workshop | Output only the scene (Phase 2) and a blank table; tell them to collect notes silently first, then bring results back for Phases 4–6 |
| User says "only rank" / "only mitigations" | Run Phase 0 and the requested phase, using the existing Premortem subsection as input |
| Voice-dictated / garbled risk text | Flag the mismatch instead of silently guessing |

---

## Notes

- Individual-first generation is the point of the technique — do not show Claude's candidates before the user has answered, unless they opt out.
- Don't invent evidence. A quote or file reference must exist in the vault; otherwise label the reason `general`.
- The premortem is not a one-off: the `Revisit` date is what makes it useful. Suggest it, but do not create calendar events.
