# CLAUDE.md — project-routines v2.6
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

6. **Set the activity flag** — per the Activity-based compose section below, mark this project as `active` or `inactive` for this run.

# Activity-based compose

To stay within Anthropic's stream idle timeout window, the Routine regenerates only the project sections that have changed since the last run. Inactive sections are copied verbatim from the existing My Tasks canvas. This drops generation work from "all N projects" to "the 2-3 with activity today," which keeps the compose step inside the timeout budget. Pattern stays durable as project counts scale.

## Capture prior canvas state at run start

When you read the My Tasks canvas at the start of the run (per "How to find the canvas"), preserve TWO things in memory before iterating projects:

1. **`last_run_date`** — extract the `YYYY-MM-DD` prefix from the canvas's `Last updated: ...` line at the top. If the line is missing or unparseable, treat as `0000-00-00` (first-run behavior — every project is active).

2. **`existing_sections`** — a map from **Project ID** to the verbatim section content from the canvas. Walk the canvas markdown and key by the **project ID anchor** embedded in each section header as an HTML comment (e.g. `<!-- AES-CIRSAW -->`):
   - Active Projects sections begin with `### [Display Name](hub_url) <!-- PROJECT_ID -->`. Extract `PROJECT_ID` from the anchor; capture the full block (header + body) until the next `### ` header or the next `## ` divider.
   - Project Summary entries (compact one-liners under `## Project Summary (N)`) carry the same anchor at end of line: `- **[Display Name](url)** — last session YYYY-MM-DD, <status> <!-- PROJECT_ID -->`. Key by anchor.
   - **Fallback for legacy canvases** — if a section header lacks the anchor (canvas predates v2.6), match by Display Name against the Routine Config row's Display Name as a one-time bridge. After the next write completes, every section will have its anchor and the fallback won't fire again.
   - Result: every project that has rendered content in the existing canvas has an entry in `existing_sections`, keyed deterministically by stable Project ID rather than drift-prone Display Name.

## Activity check per project

For each project in Routine Config, evaluate AFTER its per-project reads complete (Claude Canvas, channel, Human Canvas). A project has **activity since last run** if ANY of:

- New session log entry in the Claude Canvas with a date later than `last_run_date`
- One or more new project channel messages with timestamp later than `last_run_date` (focus on human posts; ignore bot/automation noise)
- A new Waiting On You item pointing at this project (compared to the existing canvas's Waiting On You section)
- The project is **not** in `existing_sections` — treat as active (new project, first render = a generation)

Otherwise the project is **inactive**.

## Compose strategy

When composing the Active Projects + Project Summary regions of the new canvas:

- **Active project** → fully regenerate the section per the Output format rules below. Apply the `compact-by-default` skill (FULL or COMPACT decision tree). Place in the destination it lands in (Active Projects or Project Summary).
- **Inactive project** → copy its entry from `existing_sections` verbatim into the same destination. No regeneration. Whatever rendering state the project was in last run carries forward.
- **New project (not in `existing_sections`)** → render fresh as if active.

The fixed-format sections (scorecard, What Changed Since Last Run, Waiting On You, Project Registry, Routine Config, footer, What's New in seed) are always regenerated — they're aggregations across projects and need fresh state.

## Format-change escape hatch

If a CLAUDE.md release changes the rendering format (new column, restructured table, new prose pattern), inactive sections will display the old format until those projects become active. This is an acceptable transient — most projects rotate through activity within a few weeks. To force a full regenerate run, the owner can manually note `_full refresh next run_` anywhere in My Tasks before the next Routine fires; the Routine should detect that string and treat all projects as active.

If parsing the existing canvas section for a project fails (header malformed, content broken), treat the project as active and regenerate. Don't propagate broken state.

# Output format — Active Projects sections

This section's rules apply ONLY to projects flagged `active` for this run by the Activity-based compose check above. Inactive projects skip this entirely — their existing section is copied verbatim.

Apply the `compact-by-default` skill at `.claude/skills/compact-by-default/SKILL.md` to determine FULL vs COMPACT rendering on each active project independently.

**Default is COMPACT.** A project renders FULL only when at least one is true:
- Any `:red_circle:` symbol appears anywhere in the project's content (red task row, callout in the section header, or attention flag)
- The project appears in the My Tasks "Waiting On You" section
- The project's last session date is within the last 2 days

If none of those is true, the project renders as a single line under a "Project Summary (N)" section placed after the last full Active Projects section and before Project Registry. See the skill for compact line format, status word taxonomy, edge cases, and promote-back rules.

The format below applies only to projects that promote to FULL.

```
### [Project Display Name](hub_canvas_url) <!-- PROJECT_ID -->

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
5. Active Projects sections (one per project — only when promoted to FULL per `compact-by-default` skill rules; if zero projects promote, emit the "Active Projects" header with a one-line `_No projects with active signal._` placeholder)
6. Project Summary section (compact one-liners for projects without active signal; omit entirely if zero qualify)
7. Project Registry (human navigation table)
8. Routine Config (machine ID table)
9. Footer note about the Routine

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
# v2.6 — 2026-04-27 — Task 61 (third attempt)
# - Activity-based compose: Routine regenerates only project sections with activity since last_run_date; inactive sections are copied verbatim from the existing My Tasks canvas.
# - Diagnosis (sister Project Chat, after v2.5 still failed): root cause is generation latency, not payload size or read volume. Even ~42-line briefings (post v2.5 trim) take long enough to compose that the stream idles out. Cannot shrink the timeout window — Anthropic infra. Cannot do incremental section writes — Slack canvas section-write API duplicates instead of replacing.
# - Strategy: capture last_run_date + existing_sections map at run start; activity check per project (new session log / new channel messages / new Waiting On You item / new project = active); active = regenerate, inactive = verbatim copy from prior canvas.
# - Typical day: 2-3 active out of N projects. Generation work drops ~80%. Scales — at 40+ projects, most are still quiet on any given day.
# - Project ID anchors embedded in every section header as HTML comments (`### [Display Name](hub_url) <!-- AES-CIRSAW -->`) make existing_sections lookup deterministic. Display names can drift; Project IDs are immutable. Compact lines under Project Summary carry the same anchor at end of line. Legacy canvases (predating v2.6) fall back to Display Name matching for one bridge run; subsequent writes embed anchors everywhere.
# - Format-change escape hatch: owner notes `_full refresh next run_` in My Tasks to force regenerate-all on next fire.
# - compact-by-default skill still applies to active projects; only changes which projects skip compaction entirely (inactive = no compaction logic, just copy).
# v2.5 — 2026-04-27 — Task 61 (rolled forward)
# - SUPERSEDES v2.4 — two-phase write protocol rolled back. Diagnosis was wrong: stream idle timeout fires during canvas content generation (model pauses mid-stream while emitting long content), not during idle between reads and the write call. Splitting turns didn't isolate the failure mode.
# - Live test 2026-04-27: model improvised around v2.4 spec ("output token limit prevents emitting the full block"), went directly to write, hit same Stream idle timeout. Real cause is generation length.
# - Restored v2.3 "Writing back" section: single-phase, no marker block, original Canvas Write Rule + Write order.
# - Replaced compact-quiet-project skill with compact-by-default skill at .claude/skills/compact-by-default/SKILL.md. New rule defaults all projects to one-line summary; promotes to FULL Active Projects section only on red flag (anywhere in project content), Waiting On You item, OR session within 2 days.
# - Old compact-quiet-project skill marked deprecated in place; will be removed once next release stabilizes.
# - "Quiet Projects (N)" section renamed to "Project Summary (N)" — compaction is now the default, not the exception.
# - Audit at release (Josh's 14-project My Tasks): 4 FULL (Panel BOM, JobyWorks/UNS, 527 Post Cure, Pickle Autoclave — all red-flag triggered), 10 COMPACT. Estimated payload reduction ~60-66% in Active Projects region.
# - Caveat: "Waiting On You" criterion is currently a no-op in Josh's canvas (no such section exists, just a header counter). Kept in the rule for forward-compat with other collaborators.
# v2.4 — 2026-04-27 — Task 61 (ROLLED BACK in v2.5)
# - [Two-phase write protocol — rolled back. Diagnosis was wrong; see v2.5.]
# v2.3 — 2026-04-26
# - Added compact-quiet-project skill at .claude/skills/compact-quiet-project/SKILL.md — collapses projects with no recent activity, no red flags, no waiting-on-you items into one-line summaries
# - Quiet projects group under a single "Quiet Projects (N)" section placed after Active Projects, before Project Registry; omitted entirely if zero qualify
# - Reduces briefing payload: ~6-10 lines saved per qualifying project. Audit at release: 4 of Josh's 14 projects qualify (~24 lines saved).
# - Output format updated: skill referenced from "Output format — Active Projects sections"; write order item 6 added for Quiet Projects section
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
