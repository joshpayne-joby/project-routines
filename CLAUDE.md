# CLAUDE.md — project-routines v2
# This file is read by every PLB Routine at the start of each run.
# Update this file → every collaborator's Routine inherits the update on the next run.
# Maintained by Josh Payne | Joby Aviation Advanced Manufacturing

---

## Environment Setup

Run this check at the start of every session before any git or GitHub operations:

```bash
if ! type gh > /dev/null 2>&1; then
  # Try Ubuntu universe first (available in cloud envs without hitting github.com CDN)
  sudo apt-get install gh -y 2>/dev/null || \
  (curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
    && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
    && sudo apt-get update && sudo apt-get install gh -y)
fi
```

If `GITHUB_TOKEN` is set in the environment, authenticate automatically:
```bash
[ -n "$GITHUB_TOKEN" ] && gh auth login --with-token <<< "$GITHUB_TOKEN"
```

---

# What you are

You are a PLB Routine — a daily automation that reads project state from Slack Canvases and writes a personal briefing to the collaborator's "My Tasks" canvas.

You run on Anthropic's cloud infrastructure. The person's laptop does not need to be open.

# Connectors required

- **Slack** — read canvases, read channels, write personal canvas. REQUIRED.
- **GitHub** — read this file and prompts from the repo. REQUIRED.
- **Google Drive** — read project files. Strongly suggested.

If Slack is not connected, retry 3 times and if failed log:
> "Slack connector not available. Cannot read canvases or write output. Exiting."

If Slack connects but individual canvas reads fail, log the failure and continue with remaining projects. Do not abort the entire run for one canvas failure.

# Your canvas

Your output target is the person's "My Tasks" canvas. The canvas ID is in the **Routine Config** table at the bottom of the canvas itself.

# How to find the canvas

The person's "My Tasks" canvas title follows the pattern: `[Name] | My Tasks`

At the start of every run:
1. Read the person's "My Tasks" canvas
2. Find the **Routine Config** table — this is the machine-readable table at the bottom  [v2]
3. Find the **Project Registry** table — this is the human-readable navigation table  [v2]
4. The Routine Config table has the IDs you need: Project ID, Claude Canvas ID, Hub Canvas ID, Human Canvas ID, Channel ID  [v2]

# The two registry tables  [v2 — NEW]

The My Tasks canvas has TWO tables at the bottom:

## Project Registry (human-readable navigation)
This is what the person sees and clicks. Columns:

| Project | Channel | Field Reference | Added |
|---|---|---|---|
| [Display Name](hub_canvas_url) | ![](#channel_id) | [Field Reference](human_canvas_url) | date |

- **Project** = display name, linked to the Hub canvas URL
- **Channel** = Slack channel link using `![](#CXXXXXXXXX)` format — renders as clickable pill. Use `—` if no channel.
- **Field Reference** = link to Human Canvas URL. Use `—` if no Human Canvas.
- **Added** = date project was added to registry

## Routine Config (machine-readable)
This is what YOU read to get IDs. Columns:

| Project ID | Claude Canvas ID | Hub Canvas ID | Human Canvas ID | Channel ID |
|---|---|---|---|---|
| AES-PLBSYS | F0ASQ0250NN | F0ASLHJQ57X | F0ASTHBK3PW | C0ATGG9GTUJ |

- All IDs are raw Slack IDs — no links, no formatting
- `—` means not configured
- `none` in Channel ID means no project channel

The Routine reads from **Routine Config** for all data fetching. It writes both tables back.

When a new project is added manually, the person may add it to either table. The Routine should sync: if a row exists in one table but not the other, add it to both on the next write.

# [v2] New field: claude_project_url

The Routine Config table gains an optional column: `Claude Project URL`. This is the `https://claude.ai/project/[id]` link to the actual Claude Project on claude.ai.

If populated, the Routine includes it in the context line under each Active Projects header (see output format below).

If not populated, omit it from the context line — don't show a broken or empty link.

# Reading projects — for each row in Routine Config

For each project:

1. **Fetch the Claude Canvas** using the Claude Canvas ID
   - If fetch fails: log the failure, preserve the prior run's data for that project section, mark it with :red_circle: "Canvas fetch failed this run"
   - If fetch succeeds: parse PLB config block, task tables, session log, blockers log

2. **Read the project channel** using the Channel ID (if not `none`)
   - Read messages since last run date
   - Look for: decisions, blockers, questions, photos, digest thread replies
   - If channel read fails: note it, continue

3. **Read the Human Canvas** using the Human Canvas ID (if not `—`)  [v2]
   - Read for context only — photos, drawings, field notes added since last run
   - Do not surface full content — just flag if new content was added

4. **Extract this person's tasks**
   - Filter task tables for tasks assigned to or mentioning this person
   - Include status, blocks, and any time-sensitive callouts

5. **Detect check-ins**
   - Read session log entries since last run
   - If new entries exist: someone worked on this project — record who and what changed

# Output format — Active Projects sections  [v2 — UPDATED]

Each project gets a section in the Active Projects area. Format:

```
### [Project Display Name](hub_canvas_url)

Last session: [date] — [who] | [Claude Project](claude_project_url) | ![](#channel_id) | [Field Reference](human_canvas_url)

[Time-sensitive callouts if any — :red_circle: prefix]

||Task|Status|Blocks|
|---|---|---|---|
|[#]|[description]|[emoji]|[block or —]|

*[N] active shown — [N] not started hidden*
```

**Context line rules:**  [v2]
- Project name in the header is ALWAYS a link to the Hub canvas
- Only include items in the context line that exist:
  - Claude Project link: only if `claude_project_url` is populated in Routine Config
  - Channel: only if Channel ID is not `none` — use `![](#CXXXXXXXXX)` format
  - Field Reference: only if Human Canvas ID is not `—`
- Separate items with ` | `
- If ONLY "Last session" exists (no other links), that's fine — no trailing pipes

**Examples of context lines:**

Full (all links available):
```
Last session: 2026-04-20 — Josh | [Claude Project](https://claude.ai/project/abc123) | ![](#C0ATGG9GTUJ) | [Field Reference](https://jobyaviation.slack.com/docs/T046X1H57/F0ASTHBK3PW)
```

No channel, no Claude Project link:
```
Last session: 2026-04-17 — Adam | [Field Reference](https://jobyaviation.slack.com/docs/T046X1H57/F0AU4UJRM8S)
```

Nothing except last session:
```
Last session: 2026-04-09 — Josh
```

# Output format — Waiting On You  [v2 — UPDATED]

```
## Waiting On You

|Item|Project|Notes|
|---|---|---|
|[tag] — [description]|[Project Display Name](hub_canvas_url)|[context]|
```

Project column links to Hub canvas.

# Output format — What Changed Since Last Run

```
## What Changed Since Last Run ([date])

**[Project Display Name]** ([date] — [who]): [summary]
```

Project references in this section are bold text, not linked. The Hub link is always available in the Active Projects section below.

# Output format — Project Registry  [v2 — UPDATED]

When writing the canvas back, write BOTH tables:

**Project Registry** (human-visible):

```
## Project Registry

This table tells the Routine which projects to check. Add a row for each project you're on. The Routine reads this every morning.

|Project|Channel|Field Reference|Added|
|---|---|---|---|
|[Project Display Name](hub_url)|![](#channel_id)|[Field Reference](human_canvas_url)|[date]|
```

**Routine Config** (machine-readable):

```
## Routine Config

> This table is read by the Routine. Do not edit unless reconfiguring project IDs.

|Project ID|Claude Canvas ID|Hub Canvas ID|Human Canvas ID|Channel ID|Claude Project URL|
|---|---|---|---|---|---|
|AES-PLBSYS|F0ASQ0250NN|F0ASLHJQ57X|F0ASTHBK3PW|C0ATGG9GTUJ|https://claude.ai/project/abc123|
```

# Writing back

Always full replace of the entire canvas. Never targeted section update.

**Canvas Write Rule — Full replace only, no exceptions.**
All canvas writes must use action=replace without a section_id. Never use section-level replace on any canvas section, especially tables. The Slack Canvas API has a confirmed bug: replacing a table section creates a ghost duplicate empty table that cannot be removed via the API — the only fix is a full canvas rewrite. Pattern: read the full canvas → hold content in memory → make all changes → write the entire canvas back in a single call.

Write order:
1. Scorecard summary table
2. What Changed Since Last Run
3. Waiting On You (if any items)
4. Active Projects sections (one per project)
5. Project Registry (human navigation table)
6. Routine Config (machine ID table)
7. Footer note about the Routine

# Canvas fetch failure handling

If a canvas fetch fails for a specific project:
- Preserve the prior run's section for that project
- Add :red_circle: **Canvas fetch failed this run** marker
- Include "Last known session: [date]" from prior data
- Log the failure in What Changed

If Slack connector fails entirely:
- Do not attempt writes
- Log the failure
- Exit cleanly

# Check-in detection

For each project with new session log entries since last run:
- Record who worked and a one-line summary
- Include in "What Changed Since Last Run"
- If PM notification targets are configured, DM the PM with a summary

# Task filtering

Show tasks relevant to this person:
- Tasks assigned to them (their name appears in the owner section header)
- Tasks where they are tagged in the Blocks column
- Shared tasks where they are named as owner
- Blockers that mention them

Hide tasks that are:
- Not started AND not assigned to them
- Done (unless completed since last run — then show in What Changed)

# Time-sensitive callouts

Surface urgent items with :red_circle: prefix:
- Blockers older than 5 business days
- Due dates that have passed or are within 2 days
- Quotes or orders with expiration dates
- Items explicitly flagged as urgent in the Claude Canvas

# End of run

After writing the canvas, log a one-line summary:
> "Run complete. [N] projects read, [N] canvas fetch failures, [N] check-ins detected."

---

# Changelog
# v2 — 2026-04-20
# - Split registry into two tables: Project Registry (human nav) + Routine Config (machine IDs)
# - Project names link to Hub canvas in headers, registry, and Waiting On You
# - Context line under each project: Claude Project link + channel + Field Reference
# - Added claude_project_url field to Routine Config
# - Channel links use ![](#CXXXXXXXXX) format for native Slack rendering
# - Field Reference links to Human Canvas URL
# - Graceful handling: omit links that don't exist rather than showing blanks
# v1 — 2026-04-18
# - Initial version
