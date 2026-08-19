# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is an Obsidian vault + Claude Code skills configuration. The actual note content lives in `./vault/` (on disk, outside this git repo). Claude Code skills automate workflows that read and write to that vault.

## Vault structure

Both `work/` and `my/` are organised using the **PARA method** (Projects → Areas → Resources → Archive):

- **Projects** — active initiatives with a defined outcome and end date (one `.md` file per project)
- **Areas** — ongoing responsibilities without a fixed end (people, teams, domains)
- **Resources** — reference material, templates, guides
- **Archive** — completed or paused items

```
~/obsidian/vault/
├── journal/                    # Daily notes: YYYY-MM-DD.md
├── inbox/                      # Incoming files, data files, clipped articles, PDFs, saved files and others
├── work/                       # PARA system — professional context (InPost / GoKart)
│   ├── inpost-kanban.md        # Active project board (Doing / Backlog / Done)
│   ├── projects/               # P — one .md file per active project
│   ├── areas/
│   │   ├── people/             # A — @name.md person notes
│   │   ├── gokart/teams/       # A — team notes
│   │   ├── platform/           # A — platform/devops domain
│   │   └── tech/               # A — technical domain notes
│   ├── resources/              # R — reference material, templates, PM docs
│   └── archive/                # A(rchive) — completed / paused work items
└── my/                         # PARA system — personal context (never used in work skills)
    ├── personal-kanban.md      # Personal project board
    ├── me.md                   # Personal index (father, engineer, firearms, mind&body, finance)
    ├── projects/               # P — personal active projects
    ├── areas/                  # A — life areas: engineer, father, finance, firearms, mind&body
    ├── resources/              # R — books, cook, interview notes
    └── archive/                # A(rchive) — completed personal items
```

## Skills (`.claude/skills/`)

Four skills drive the main workflows. Each skill file is a detailed step-by-step spec that Claude must follow exactly:

| Skill | Trigger phrases | What it does |
|---|---|---|
| `obsidian-add-to-journal` | "add to journal", "meeting notes", "save to Obsidian" | Appends a meeting note to `journal/YYYY-MM-DD.md`; saves action items to project files |
| `obsidian-plan-my-day` | "plan my day", "what's on my agenda" | Builds today's journal file from calendar + emails + kanban |
| `obsidian-review-journal` | "review journal", "przejrzyj dziennik" | Corrects grammar/spelling (PL+EN), writes summary, adds `#claude-reviewed` tag |
| `obsidian-review-project` | "review project", "sprawdź projekt", "przejrzyj projekt" | Reviews a project file for language quality, outcomes quality, and next-actions relevance |
| `obsidian-save-to-inbox` | "save to vault", "zapisz do vaulta" | Summarises a PDF/URL and writes a structured note to `inbox/` |

## Journal file format

Daily notes follow this structure (populated by the `plan-my-day` skill using the template at `vault/work/resources/tools/obsidian/templates/daily.md`):

```markdown
---
tags:
  - claude-reviewed   # added by review skill; skip file if present
---
## Summary        # written by review skill; placed first

## My Day         # 3–7 task checkboxes; leave untouched during review

## Meetings       # ### Title [[project-link]] subsections
                     # format: ### Meeting Title (HH:MM–HH:MM)

## Notes

## Reflections
```

## Key conventions

- **Vault path**: always `~/obsidian/vault` (not the git repo root)
- **Project links**: `[[project-name]]` in headings — the name is the project filename without `.md`
- **People links**: `[[@handle]]` — handle = file name under `work/areas/people/`
- **Person notes**: freeform, no fixed template — running task checkboxes (`- [ ]`/`- [x]`) about things to raise/follow up with that person, occasionally a loose `### Goals` section, plus inline `[[...]]` links to teams/projects. Many are empty. Don't impose headings or frontmatter that aren't already there.
- **Action items**: saved to the matching project file, not left in the journal
- **Language**: notes are mixed Polish/English; detect per-section, never translate
- **Meeting titles**: strip time patterns (`HH:MM`, `HH:MM–HH:MM`) before using as headings
- **Kanban**: only `## Doing` column matters; `- [ ]` = active, `- [x]` = done today
- **`#claude-reviewed` tag**: presence means skip the file — never re-process it
- **Archive**: manual only, no skill does this automatically. Observed pattern: project/area files → `archive/<year>/`; inactive people → `archive/people/`; team-specific docs → `archive/<team-name>/`; kanban boards snapshotted as `<board-name>-archive-<year>.md`. Journal archive is flat (`journal/archive/YYYY-MM-DD.md`, no year subfolder) — inconsistent with the others but that's the real state.
- **Tags live in two places**: personal project files often tag via frontmatter (`tags:\n  - firearms`), while kanban boards mix in inline `#hashtag` text on the same line as the item. A tag-based search/analysis must check both — grepping only for `#hashtag` will silently miss frontmatter-tagged files.
- **Voice-dictated meeting notes can be garbled**: raw dictated text occasionally mis-transcribes meeting titles, acronyms, or proper nouns (e.g. a meeting title heard as unrelated words, or a topic word garbled into something that looks like a real acronym). When the dictated content doesn't cleanly match the calendar event or vault content, flag the mismatch to the user instead of silently guessing.

## Automation

A launchd job runs the daily flow unattended on weekdays:

- `~/Library/LaunchAgents/com.lsosnicki.obsidian-plan-my-day.plist` — runs `claude -p "First run skill obsidian-review-journal, then run skill obsidian-plan-my-day."` Mon–Fri (currently 08:00).
- Logs: `~/Library/Logs/obsidian-plan-my-day.log`
- After editing the plist, reload it: `launchctl unload` then `launchctl load` the file.

## MCP integrations

- **Microsoft 365** — calendar and email search (pre-approved in `settings.local.json`)
- **PDF Viewer** — read PDFs for inbox summarisation

Vault files themselves are read/written with the built-in Read/Write/Edit tools and Bash (`ls`, `cp`) — there is no "Filesystem" MCP server. Skill files should reference these, not `Filesystem:*` tool calls.
