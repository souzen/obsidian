# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is an Obsidian vault + Claude Code skills configuration. The actual note content lives in `./vault/` (on disk, outside this git repo). Claude Code skills automate workflows that read and write to that vault.

## Way of Working

The vault follows a loose Capture → Organize → Distill → Express flow (CODE), each stage handled by different skills:

- **Capture** — raw input lands with minimal friction and no expectation of polish: `inbox/` (articles/PDFs via `obsidian-inbox-add`), `journal/YYYY-MM-DD.md` (meeting notes via `obsidian-journal-add`, daily tasks via `obsidian-journal-plan-my-day`). Mixed Polish/English and voice-dictation artifacts are expected here, not cleaned up yet.
- **Organize** — captured material is filed into the PARA structure (`work/`, `my/`): action items move out of the journal into the matching project file; completed/paused items eventually move to `archive/` (manual, no skill does this automatically).
- **Distill** — `obsidian-journal-review` moves the Decisions and Next Steps of journal entries tagged `#claude-review` into the linked project; `obsidian-project-outcomes-review` turns a project's raw next-actions into an outcomes-over-outputs framing and prunes stale/duplicate tasks.
- **Express** — distilled context becomes something usable in a real conversation: `obsidian-meeting-f2f-agenda` and `obsidian-meeting-any-agenda` turn accumulated notes into a concrete meeting agenda.

Consistency in capturing and reviewing matters more than any single note being polished or perfectly filed — see `inbox/the-code-system-my-way.md`.

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
│   │   │   └── teams/          # A — team notes; subfolders per org grouping (gokart/, platform/, consumer/), plus top-level team files
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

These skills drive the main workflows. Each skill file is a detailed step-by-step spec that Claude must follow exactly:

| Skill | Trigger phrases | What it does |
|---|---|---|
| `obsidian-journal-add` | "add to journal", "meeting notes", "save to Obsidian" | Appends a meeting note to `journal/YYYY-MM-DD.md` (journal only — never writes to project files) |
| `obsidian-journal-import-wispr` | "import wispr notes", "add new wispr notes to journal", "sync wispr" | Fetches finalized Wispr Flow notes, skips already imported ones, and runs `obsidian-journal-add` for each (asks before each note) |
| `obsidian-journal-plan-my-day` | "plan my day", "what's on my agenda" | Builds today's journal file's Meetings section from the calendar |
| `obsidian-journal-review` | "review journal", "przejrzyj dziennik" | For journal entries tagged `#claude-review`: copies Decisions into the linked project's Log, appends Next Steps to its Next Actions, then retags `#claude-reviewed` |
| `obsidian-project-outcomes-review` | "review project", "sprawdź projekt", "przejrzyj projekt", "wygeneruj outcome dla projektu" | Reviews a project file for language quality, outcomes quality, and next-actions relevance |
| `obsidian-inbox-add` | "save to vault", "zapisz do vaulta" | Summarises a PDF/URL and writes a structured note to `inbox/` |
| `obsidian-meeting-f2f-agenda` | "prepare f2f agenda", "przygotuj agendę na f2f", "agenda na spotkanie z @X" | Builds a proposed 1:1 agenda for a person from vault context (projects, teams, journal); offers to save as today's meeting note |
| `obsidian-meeting-any-agenda` | "prepare meeting agenda", "przygotuj agendę na spotkanie", "agenda na spotkanie o <project>" | Builds a proposed agenda for a project-related meeting from the project's own Outcomes/Next Actions/Decisions; offers to save as today's meeting note |

## Templates

`vault/work/resources/tools/obsidian/templates/` holds the latest templates for new notes: `daily.md` (journal), `project.md` and `@person.md`. The other files in that folder are older period notes (weekly, monthly, quarterly, yearly).

- **`project.md`** — frontmatter (`tags`, `date-added`), then `## Outcomes`, `## Next Actions`, `## Resources`, `## Log %% fold %%` (Decisions, Questions, Issues, Messages)
- **`@person.md`** — `## Talking points` (checkboxes to raise with the person), `## Log %% fold %%` (Decisions, Questions, Issues, Messages)

Existing notes may predate a template (e.g. person notes use `### Talking points`), so skills match headings by keyword, not by fixed level.

## Journal file format

Daily notes follow this structure (populated by the `obsidian-journal-plan-my-day` skill using the template at `vault/work/resources/tools/obsidian/templates/daily.md`):

```markdown
## Meetings           # ### Title [[project-link]] subsections
                       # format: ### Meeting Title (HH:MM–HH:MM)

## Journal %% fold %% # fold-wrapper heading; no content of its own
### Notes              # narrative notes about the day
### Reflections        # random reflections
```

## Journal meeting entries

Shared rules for any skill that attaches a note or agenda to a meeting in today's journal:

- **Find the meeting** (Outlook calendar, `Microsoft 365:outlook_calendar_search`): match by title keywords on the target day; else by approximate time (±30 min); else the meeting that ended latest before now. If today's journal already has a matching `### <title>` heading, use it instead. Keep title and `HH:MM–HH:MM`; never save participants.
- **Find the project**: look in `work/projects/` first (`ls`), then in `my/projects/` only if there is no match. Match the name case-insensitively, ignoring dashes/spaces. One match → use it; more than one → ask the user; none in either folder → the calling skill decides.
- **Insert the entry**: existing `### <title>` heading → insert under it; `## Meetings` exists but no such heading → append `### <title> (HH:MM–HH:MM) [[links]]` at the end of that section; no `## Meetings` section → append the section at the end of the file. Write the full file, not a patch.

## Key conventions

- **Vault path**: always `~/obsidian/vault` (not the git repo root)
- **Project links**: `[[project-name]]` in headings — the name is the project filename without `.md`
- **People links**: `[[@handle]]` — handle = file name under `work/areas/people/`
- **Team links**: `[[~team-slug]]` — team notes are named `~<slug>.md` under `work/areas/people/teams/` (recurse into subfolders). The `~` prefix, not the subfolder, is what identifies a team note.
- **Person notes**: follow `templates/@person.md` — `Talking points` (running task checkboxes about things to raise/follow up with that person; `obsidian-journal-review` appends f2f Next Steps here) and `Log`, plus inline `[[...]]` links to teams/projects. Older notes may be empty or have a loose `### Goals` section. Don't impose frontmatter that isn't already there.
- **Action items**: saved to the matching project file, not left in the journal
- **Language**: notes are mixed Polish/English; detect per-section, never translate
- **Meeting titles**: strip time patterns (`HH:MM`, `HH:MM–HH:MM`) before using as headings
- **Kanban**: only `## Doing` column matters; `- [ ]` = active, `- [x]` = done today
- **`#claude-review` / `#claude-reviewed` tags**: `#claude-review` on a meeting heading in the journal queues that entry for `obsidian-journal-review`; after processing it becomes `#claude-reviewed` and is never processed again
- **Archive**: manual only, no skill does this automatically. Observed pattern: project/area files → `archive/<year>/`; inactive people → `archive/people/`; team-specific docs → `archive/<team-name>/`; kanban boards snapshotted as `<board-name>-archive-<year>.md`. Journal archive is flat (`journal/archive/YYYY-MM-DD.md`, no year subfolder) — inconsistent with the others but that's the real state.
- **Tags live in two places**: personal project files often tag via frontmatter (`tags:\n  - firearms`), while kanban boards mix in inline `#hashtag` text on the same line as the item. A tag-based search/analysis must check both — grepping only for `#hashtag` will silently miss frontmatter-tagged files.
- **Voice-dictated meeting notes can be garbled**: raw dictated text occasionally mis-transcribes meeting titles, acronyms, or proper nouns (e.g. a meeting title heard as unrelated words, or a topic word garbled into something that looks like a real acronym). When the dictated content doesn't cleanly match the calendar event or vault content, flag the mismatch to the user instead of silently guessing.

## Automation

Scheduled tasks are launchd jobs. The plists in `launchd/` are the **source of truth** — edit them there, never in `~/Library/LaunchAgents/`.

- `launchd/com.lsosnicki.obsidian-plan-my-day.plist` — runs `claude -p "First run skill obsidian-journal-review, then run skill obsidian-journal-plan-my-day."` Mon–Fri (currently 08:00). Log: `~/Library/Logs/obsidian-plan-my-day.log`
- `launchd/com.lsosnicki.obsidian-import-wispr.plist` — runs `obsidian-journal-import-wispr` unattended Mon–Fri every hour, 10:00–17:00 (skips notes that would need a new journal entry). Log: `~/Library/Logs/obsidian-import-wispr.log`
- `launchd/com.lsosnicki.obsidian-review-journal.plist` — runs `obsidian-journal-review` unattended Mon–Fri every hour at :30, 10:30–17:30. Log: `~/Library/Logs/obsidian-review-journal.log`
- After editing a plist, deploy it: `launchctl unload ~/Library/LaunchAgents/<name>.plist; cp launchd/<name>.plist ~/Library/LaunchAgents/ && launchctl load ~/Library/LaunchAgents/<name>.plist`

## MCP integrations

- **Microsoft 365** — calendar and email search (pre-approved in `settings.local.json`)
- **PDF Viewer** — read PDFs for inbox summarisation

Vault files themselves are read/written with the built-in Read/Write/Edit tools and Bash (`ls`, `cp`) — there is no "Filesystem" MCP server. Skill files should reference these, not `Filesystem:*` tool calls.
