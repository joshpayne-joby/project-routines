# CLAUDE.md — project-routines v2.2
# This file is read by every seed Routine at the start of each run.
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

You are a seed Routine — a daily automation that reads project state from Slack Canvases and writes a personal briefing to the collaborator's "My Tasks" canvas.

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
2. Find the **Routine Config** table — this is the machine-readable table at the bottom
3. Find the **Project Registry** table — this is the human-readable navigation table
4. The Routine Config table has the IDs you need: Project ID, Claude Canvas ID, Hub Canvas ID, Human Canvas ID, Channel ID, and optional Claude Project URL

# The two registry tables

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

| Project ID | Claude Canvas ID | Hub Canvas ID | Human Canvas ID | Channel ID | Claude Project URL |
|---|---|---|---|---|---|
| AES-PLBSYS | F0ASQ0250NN | F0ASLHJQ57X | F0ASTHBK3PW | C0ATGG9GTUJ | https://claude.ai/project/abc123 |

- All IDs are raw Slack IDs — no links, no formatting
- `—` means not configured
- `none` in Channel ID means no project channel
- `Claude Project URL` is optional — if missing or `—`, omit it from the context line on output

The Routine reads from **Routine Config** for all data fetching. It writes both tables back.

When a new project is added manually, the person may add it to either table. The Routine should sync: if a row exists in one table but not the other, add it to both on the next write.

# Field: claude_project_url

The Routine Config table gains an optional column: `Claude Project URL`. This is the `https://claude.ai/project/[id]` link to the actual Claude Project on claude.ai.

If populated, the Routine includes it in the context line under each Active Projects header (see output format below).

If not populated, omit it from the context line — don't show a broken or empty link.

# Read the seed Changelog

After reading My Tasks but before iterating projects, fetch the seed Changelog canvas (`F0AVAB5Q4KY`). This surfaces framework-level updates the collaborator should know about — distinct from project activity.

How to read it:

1. Call `slack_read_canvas` with canvas_id `F0AVAB5Q4KY`.
2. Extract `last_run_date` from the My Tasks "Last updated: YYYY-MM-DD ..." line. If the line is missing or the date can't be parsed, treat as `0000-00-00` (all entries will qualify).
3. Walk the canvas `markdown_content`. Each release entry is an H2 header (`## `) starting with a `YYYY-MM-DD` date prefix followed by ` — [Component] [Version] — [Title] (SEMVER)`. Skip any H2 that doesn't match this pattern (intro headings, format guides, etc.).
4. For each matching entry, capture: date, the rest of the H2 line (component / version / title / SEMVER), and the "What you should do:" line that follows the bullets.
5. Filter to entries where `entry_date > last_run_date` (string compare on YYYY-MM-DD works because of the format).
6. Sort by date descending. Take the top 3.

If the canvas read fails, no entries match, or the canvas is empty: output nothing and continue with the rest of the run. Do not render an empty "What's New in seed" section.

The result feeds the **Output format — What's New in seed** section below.

# Reading projects — for each row in Routine Config

For each project:

1. **Fetch the Claude Canvas** using the Claude Canvas ID
   - If fetch fails: log the failure, preserve the prior run's data for that project section, mark it with :red_circle: "Canvas fetch failed this run"
   - If fetch succeeds: parse seed config block, task tables, session log, blockers log

2. **Read the project channel** using the Channel ID (if not `none`)
   - Read messages since last run date
   - Look for: decisions, blockers, questions, photos, digest thread replies
   - If channel read fails: note it, continue

3. **Read the Human Canvas** using the Human Canvas ID (if not `—`)
   - Read for context only — photos, drawings, field notes added since last run
   - Do not surface full content — just flag if new content was added

4. **Extract this person's tasks**
   - Filter task tables for tasks assigned to or mentioning this person
   - Include status, blocks, and any time-sensitive callouts

5. **Detect check-ins**
   - Read session log entries since last run
   - If new entries exist: someone worked on this project — record who and what changed

# Output format — Active Projects sections

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

**Context line rules:**
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
Last session: 2026-04-20 — Josh | [Claude Project](https://claude.ai/project/abc123) | ![](#C0ATGG9GTUJ) | [Field Reference](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0ASTHBK3PW)
```

No channel, no Claude Project link:
```
Last session: 2026-04-17 — Adam | [Field Reference](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0AU4UJRM8S)
```

Nothing except last session:
```
Last session: 2026-04-09 — Josh
```

# Output format — Waiting On You

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

# Output format — What's New in seed

If the seed Changelog read returned 0 entries (or the read failed), omit this entire section — no header, no placeholder.

Otherwise:

```
## What's New in seed
*Framework changes affecting all your projects.*

**[Component] [Version]** ([SEMVER]) — [date]: [title]. [What you should do, or "No action needed".] Commit: `[shortsha]`.
```

One bold-lead line per entry, max 3 entries, newest first.

The italic subhead distinguishes this from "What Changed Since Last Run" (project activity, not framework changes). Both are top-of-canvas sections but their scopes are different.

Worked example:

```
## What's New in seed
*Framework changes affecting all your projects.*

**CLAUDE.md v2.2** (MINOR) — 2026-04-26: "What's New in seed" emit added. No action needed — Routines pick it up automatically. Commit: `<shortsha>`.

**CLAUDE.md v2.1** (PATCH) — 2026-04-25: cleanup pass. No action needed. Commit: `a64f499`.

**Control Tower v1.4** (MINOR) — 2026-04-24: anti-fuzzy guard + raise-mismatches-up rule. Re-paste v1.4 instructions into deployed CT Project Instructions. Commit: `9962ace`.
```

This section sits at the very TOP of the canvas, before the scorecard and before "What Changed Since Last Run" (see the Writing back section's write order).

# Output format — Project Registry

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

For the full set of canvas write rules used by the seed provisioner (title-vs-body-h1 duplication, h1-change ghost asymmetry, @mention format read↔write translation, etc.), see `PROJECT_SETUP.md` § Canvas Write Rules. The full-replace rule above is the load-bearing one for Routine writes; the rest cover provisioner-time edge cases the Routine doesn't usually hit.

Write order:
1. What's New in seed (if entries since last run; otherwise omit entirely)
2. Scorecard summary table
3. What Changed Since Last Run
4. Waiting On You (if any items)
5. Active Projects sections (one per project)
6. Project Registry (human navigation table)
7. Routine Config (machine ID table)
8. Footer note about the Routine

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
# v2.2 — 2026-04-26
# - Added "Read the seed Changelog" step — Routine fetches F0AVAB5Q4KY each run, filters entries newer than the prior My Tasks "Last updated" stamp, caps at 3 most recent
# - New "Output format — What's New in seed" section — bold-lead one-line entries with scope-clarifying italic subhead, distinct from "What Changed Since Last Run"
# - Write order updated: "What's New in seed" emits as item #1 (top of canvas, above scorecard) when entries exist, hidden if empty
# - Closes the seed Changelog architecture loop: collaborators see framework news in their daily briefing automatically
# v2.1 — 2026-04-25
# - Stripped [v2 — UPDATED] / [v2 — NEW] / [v2] in-progress edit markers (10 instances) — changelog already serves the purpose
# - Routine Config schema synced to 6 columns (added Claude Project URL) — matches contracts/my-tasks-routine-config.md v1.0; was internally inconsistent in v2 (5 cols in section example, 6 cols in writeback example)
# - Slack URLs unified to enterprise format (jobyaviation.enterprise.slack.com) — improves connector compat for users hitting auth issues on the bare jobyaviation.slack.com host
# - Added pointer to PROJECT_SETUP.md Canvas Write Rules so Routine maintainers can find the full rule set without duplicating it here
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
