---
name: compact-quiet-project
description: Render a project with no recent session activity, no red flags, and no items waiting on the owner as a one-line summary instead of a full Active Projects section. Reduces seed Routine briefing payload size when many projects are stale.
when_to_use: Use during the daily seed Routine briefing emit. Apply per-project after reading Routine Config — if all three quietness criteria pass, replace the full Active Projects section with a single bullet line and group all such bullets under a "Quiet Projects (N)" section after Active Projects.
---

# Compact Quiet Project

When briefing a person's Active Projects, most projects in a healthy week have no movement — last session was 7+ days ago, no blockers turned red, nothing is waiting on the owner. Rendering each with a full task table costs ~6-10 lines per project for zero new signal. This skill replaces those full sections with one line each and groups them under a single "Quiet Projects (N)" header.

## When to apply

Apply per-project. A project qualifies as quiet if ALL THREE are true:

1. **Stale** — `last_session_date` is 7 or more days older than today.
2. **No red flags** — zero tasks currently with `:red_circle:` status in the project's task table; no time-sensitive callouts (expired quotes, missed deadlines, "needs attention" markers) in the project's section header.
3. **No waiting-on-owner** — the project does NOT appear in the My Tasks "Waiting On You" section.

If ANY of the three is false, fall through to the full Active Projects section format. This is a per-project decision, not all-or-nothing.

## Determining last_session_date

Read the project's Claude Canvas. The most recent session log entry's date is the `last_session_date`. If the canvas has no session log entries at all, treat the project as not-stale (don't compact unknown projects — render the full section so the gap is visible).

## Compact rendering format

Replace the project's full Active Projects section with a single bullet line:

```
- **[Project Display Name](hub_canvas_url)** — quiet [N]d, last session [YYYY-MM-DD]
```

Where:

- `[Project Display Name]` and `hub_canvas_url` come from the Routine Config row.
- `[N]d` is days since last session (e.g., `9d`, `14d`).
- `[YYYY-MM-DD]` is the last session date in ISO format.

## Grouping under "Quiet Projects (N)"

Collect ALL compacted entries under a single section, placed AFTER the last full Active Projects section and BEFORE the Project Registry table:

```
## Quiet Projects ([N])

- **[Project A](url)** — quiet 8d, last session 2026-04-17
- **[Project B](url)** — quiet 14d, last session 2026-04-11
- **[Project C](url)** — quiet 9d, last session 2026-04-16
```

Where `[N]` is the count of compacted projects.

If `[N]` is 0, omit the entire "Quiet Projects" section — no header, no placeholder.

## What stays the same

- Project still counted in the scorecard table at the top of the canvas.
- Project still appears in "What Changed Since Last Run" if there was new activity since last run (a new entry would auto-disqualify the project from compact treatment, since it would reset staleness).
- Project still has rows in Project Registry and Routine Config tables — unchanged.
- Project's Hub canvas link is preserved in the compact line for navigation.

## Promote-back rules

A compacted project gets promoted back to a full Active Projects section when ANY criterion flips:

- A new session log entry resets staleness (last session within 7 days).
- A blocker turns `:red_circle:` (red flag triggered).
- A new task or item is added to "Waiting On You" pointing at this project.

The owner can also manually request "show me [project] in full" — the Routine should respect that for the next run as a one-time override.
