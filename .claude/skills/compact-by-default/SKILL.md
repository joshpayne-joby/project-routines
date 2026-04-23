---
name: compact-by-default
description: Default-compact rendering for project sections in My Tasks. Each project renders as a one-line bullet under "Project Summary (N)" unless it promotes to a full Active Projects section due to red flag, Waiting On You item, or recent session within 2 days.
when_to_use: Apply per-project during the seed Routine's Active Projects render loop. Decide FULL vs COMPACT on each project independently using the decision tree below. Supersedes the earlier compact-quiet-project skill, which only compacted stale-and-quiet projects.
---

# Compact By Default

Default rendering for projects in the daily seed Routine briefing is COMPACT — a single bullet line. Projects promote to a FULL Active Projects section only when there's active signal worth detail.

This shifts the cost model: full sections are the exception, not the default. Reduces briefing payload (~60-66% on Josh's 14-project My Tasks at release-time audit) which is necessary to clear the Routine's Stream idle timeout threshold during canvas writeback.

## Decision tree

For each project, evaluate in order. First hit wins:

1. **Has red flag** — any `:red_circle:` symbol appears in the project's content (red task row, callout in section header, or attention flag) → render **FULL**
2. **In Waiting On You** — project appears in the My Tasks "Waiting On You" section → render **FULL**
3. **Recent session** — last session date is within the last 2 days → render **FULL**
4. **Otherwise** → render **COMPACT**

All three checks read state already loaded by the Routine — no extra fetches.

## Compact line format

```
- **[Display Name](hub_canvas_url)** — last session YYYY-MM-DD, <status> <!-- PROJECT_ID -->
```

Where `PROJECT_ID` is the immutable Project ID from the Routine Config row for this project (e.g. `AES-CIRSAW`, `AMFG-TEMPER`, `TFAB-IRIS`). The HTML comment is a parser anchor — it's invisible in Slack canvas rendering but lets the next run's Activity-based compose look up this project's prior section deterministically, even if the Display Name has drifted. See CLAUDE.md § Activity-based compose for the full rationale.

Status word, picked deterministically from session age:

- `active` — last session within 7 days, no red, not waiting (would have been FULL via rule 3 inside 2 days; 3-7 days = active-but-quiet)
- `quiet` — last session 8-30 days ago
- `dormant` — last session >30 days ago, or unknown
- `new` — project registered but no session log entries yet

Edge rendering:

- No hub canvas URL on record → render display name as plain bold (no link), append `(no hub)` after the status word
- Display name missing → fall back to project code from Routine Config
- Session date unknown → omit the `last session YYYY-MM-DD,` clause; just `<status>`

## Section structure

Compact entries collect under a single section:

```
## Project Summary (N)

- **[Project A](url)** — last session 2026-04-20, active <!-- AES-CIRSAW -->
- **[Project B](url)** — last session 2026-04-09, dormant <!-- AES-PICKLE -->
- **[Project C](url)** — last session 2026-04-16, quiet <!-- AMFG-TEMPER -->
```

Where `(N)` is the count of compacted projects.

Placement: AFTER the last full Active Projects section, BEFORE Project Registry.

If `N = 0`, omit the entire section — no header, no placeholder.

Sort: most recent `last session` first; `new` projects last.

## Special case: empty Active Projects

If zero projects render FULL on a given run, still emit the "Active Projects" header with a single-line italic placeholder:

```
## Active Projects

_No projects with active signal._
```

Keeps the structural anchor stable so downstream consumers (and human readers) don't think the section was forgotten.

## What stays the same

- Project still counted in the scorecard table at the top of the canvas
- Project still appears in "What Changed Since Last Run" if there was new activity since last run (a new entry resets staleness; if it's also within 2 days, the project promotes to FULL anyway via rule 3)
- Project still has rows in Project Registry and Routine Config tables — unchanged
- Project's Hub canvas link is preserved in the compact line for navigation

## Promote-to-full rules

A compacted project promotes to a FULL Active Projects section when ANY decision-tree criterion flips:

- A new task gets `:red_circle:` status, OR a `:red_circle:` callout/attention flag is added to the project's section
- A new Waiting On You item lands pointing at the project
- A new session log entry within the last 2 days lands

The owner can also manually request "show me [project] in full" — the Routine should respect that for the next run as a one-time override.

## Edge cases

| Case | Rule |
|---|---|
| Project with zero tasks | COMPACT (no tasks ⇒ no red ⇒ falls through unless other signals trigger) |
| Yellow callouts only, no red | COMPACT (yellow is not promotion-worthy; if owner wants visibility, add a Waiting On You item or log a session) |
| Newly registered, no history | COMPACT with `new` status word |
| Red task in archived/closed project | FULL (red is load-bearing; archive should remove from Registry first) |
| Waiting On You item points at a project not in Registry | Surface as a mismatch — do not silently render. (Per the framework's raise-mismatches-up rule.) |
| Multiple session logs same day | Use max date; tiebreaker irrelevant |
| Red callout in section header but no red task row | FULL (callout-level red counts; do not strip-mine for task-row-only red) |

## Relationship to the earlier `compact-quiet-project` skill

This skill **supersedes** `compact-quiet-project`. The old skill only compacted projects meeting all of: stale 7+ days AND no red flags AND not in Waiting On You. That's a strict subset of the new rule's COMPACT branch, so any project the old skill compacted, this skill also compacts.

The old skill is marked deprecated in place. It will be removed once this skill stabilizes through one full release cycle.
