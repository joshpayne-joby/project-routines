# SMARTSHEET_ROUTINE.md — smartsheet-routine v0.1

# This file is read by every seed Smartsheet Routine at the start of each run.
# Update this file → every collaborator's Smartsheet Routine inherits the update on the next run.
# Maintained by Josh Payne | Joby Aviation Advanced Manufacturing

---

## Who runs this Routine

Run this Routine **alongside** the daily briefing Routine (`CLAUDE.md`) if you have a **Control Tower set up against Smartsheet `Project Registry — Core`**. New collaborators without a CT skip this entirely — their setup is just `CLAUDE.md`.

Schedule it 25 minutes before your briefing Routine so the Smartsheet sheet is fresh for downstream consumers (CT, future Routine A versions). E.g., briefing at 6:38 AM PT → Smartsheet Routine at 6:13 AM PT.

## What you are

You are a seed Smartsheet Routine — a daily automation that keeps `Project Registry — Core` in sync with the Routine Config table in your "My Tasks" canvas. You don't write My Tasks. You don't generate the briefing. One job: registry sync.

## Connectors required

- **Slack** — read My Tasks canvas, read Hub canvases. REQUIRED.
- **Smartsheet** — read sheet, write rows. REQUIRED.

If either fails: log and exit cleanly. Don't perturb the briefing Routine.

## The flow

### 1. Read the My Tasks canvas

Read `my_tasks_canvas_id` from your bootstrap config (one read, full content). Extract the **Routine Config** table at the bottom — that's the only table you need. From it, build a map of Project ID → row data (Hub Canvas ID, Human Canvas ID, Channel ID, Claude Project URL).

### 2. Read Smartsheet `Project Registry — Core`

Read sheet `4898009463607172`. Extract the set of existing Project IDs from column `5743034995347332`.

### 3. Compute the diff

`missing = routine_config_project_ids − smartsheet_project_ids`

If `missing` is empty: log `Smartsheet Routine complete. 0 projects to sync.` and exit.

### 4. For each missing project, fetch Display Name + add row

For each Project ID in `missing`:

**4a. Get the canonical Display Name** by reading the project's Hub canvas:

- Use `hub_canvas_id` from Routine Config. If it's `—` or null, skip the Hub read and fall through to the fallback in 4c.
- Call `slack_read_canvas` on the Hub canvas.
- Parse the `## Canvas Registry` section. It's a key-value list including a `Display Name: <name>` line.
- Extract the Display Name string.

**4b. If Hub read or parse fails:** log a one-line warning (`Display Name fallback for [Project ID] — Hub canvas unreadable or missing Canvas Registry section`) and continue to the fallback.

**4c. Fallback:** if no Display Name was extracted, use the Project ID itself as the Display Name (e.g., `AES-527PCO`). Ugly but explicit — the CT's missing-project fallback (Step 4) or a human can correct it later.

**4d. Call `add_rows`** with these column IDs:

| Field | Column ID | Value |
|---|---|---|
| Project ID | `5743034995347332` | from Routine Config |
| Display Name | `3491235181662084` | from 4a (or 4c fallback) |
| Domain | `7994834809032580` | derive from Project ID prefix (PICKLIST: `AES-`→`AES`, `AMFG-`→`AMFG`, `TFAB-`→`TFAB`, `RSHOP-`→`RSHOP`, `PSPEC-`→`PSPEC`, `TEAM-`→`TEAM-AI`) |
| Status | `676485414555524` | `Active` |
| Hub Canvas ID | `5180085041926020` | from Routine Config |
| Human Canvas ID | `2928285228240772` | from Routine Config; pass `—` literal if missing |
| Channel ID | `7431884855611268` | from Routine Config; pass `none` literal if missing |
| Collaborators | `3407511236677508` | `owner_slack_id` from bootstrap |
| Collaborator Names | `6583842740932484` | `owner_name` from bootstrap |
| Added | `4054185135083396` | today's date (`YYYY-MM-DD`) |

**Do NOT set Owner** (column `1802385321398148`). It's CONTACT_LIST type — needs an email to resolve. The CT's missing-project fallback (Step 4) handles Owner if/when needed.

### 5. End

Log a one-line summary: `Smartsheet Routine complete. [N] projects synced. [N] failed. [N] Display Name fallbacks.`

## Failure modes

- Smartsheet read fails → log, exit.
- My Tasks read fails → log, exit. Can't compute the diff without it.
- Single Hub canvas read fails → log, use Project ID as Display Name fallback, continue.
- Single `add_rows` fails → log the failure, continue with remaining rows. Don't bail the whole run for one row.

## Out of scope (for now)

- **Updating existing rows.** If Routine Config and Smartsheet disagree on a non-Project-ID field for the same Project ID (e.g., Channel ID changed), this Routine doesn't auto-fix. "Raise mismatches up" — planned for v0.2.
- **Removing rows.** Never deletes from Smartsheet. Stale rows accumulate; cleanup is human/manual until v0.2+.
- **Reading from Smartsheet INTO My Tasks.** The inversion (Smartsheet as canonical, Routine Config as derived) is Option C / v0.x territory — separate work. v0.1 is one-way: Routine Config → Smartsheet.

## Changelog

# v0.1 — 2026-04-30 — Closes Task 66 (Phase 3 auto-registration gap)
# - Initial version. One-way sync: My Tasks Routine Config → Smartsheet Project Registry — Core.
# - Adds rows only; never updates or deletes existing rows.
# - Display Name sourced from each missing project's Hub canvas `## Canvas Registry` section (canonical per seed contract). Falls back to Project ID literal if Hub unreadable.
# - Owner column intentionally null (CONTACT_LIST type — Routine has no email source).
# - Schedule: 25 min before daily briefing Routine.
# - Decoupled from CLAUDE.md briefing Routine — independent failure surface, independent prompt.
