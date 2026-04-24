# My Tasks — Routine Config Table — Contract

Every collaborator's "My Tasks" canvas contains a `## Routine Config` table. It is the Routine's source of truth for the project list and canvas IDs — the Routine reads this on every run to know which projects to brief on.

This contract exists because the table is both produced and consumed by Routine runs. Column drift breaks the next run silently: a renamed column means the Routine can't find the Claude Canvas ID, and the project "drops out" of tomorrow's briefing.

My Tasks has a second table — `## Project Registry` (human-readable) — that must stay in sync with Routine Config. Registry is not a parser contract; this file only specs Routine Config.

---

## Canonical format

```
## Routine Config

> This table is read by the Routine. Do not edit unless reconfiguring project IDs.

|Project ID|Claude Canvas ID|Hub Canvas ID|Human Canvas ID|Channel ID|Claude Project URL|
|---|---|---|---|---|---|
|[project_id]|[tasks_canvas_id]|[hub_canvas_id]|[human_canvas_id]|[channel_id or "none"]|[url or "—"]|
```

Markdown table, six columns, one row per project. Column headers are fixed — do not rename. Values are plain text (no backticks, no link-wrapping).

An empty table uses a single placeholder row of six `—` cells:

```
|—|—|—|—|—|—|
```

The Routine treats a single all-`—` row as "no projects registered yet" and skips the briefing. Never delete the row structure — an empty table body breaks Slack's canvas table renderer.

---

## Column spec

|Column|Type|Required|Notes|
|---|---|---|---|
|Project ID|string|Yes|Matches `[DOMAIN]-[SHORTNAME]` (e.g. `AES-WINGFLIP`). Must match the value in the project's Hub Canvas Registry.|
|Claude Canvas ID|Slack canvas ID|Yes|`F0...` format. No backticks. The Routine fetches this canvas each run.|
|Hub Canvas ID|Slack canvas ID|Yes|`F0...` format. Used by the missing-project fallback and for context links.|
|Human Canvas ID|Slack canvas ID|Yes|`F0...` format. Currently read-only from the Routine's perspective, but required for schema completeness.|
|Channel ID|Slack channel ID OR `none`|Yes|`C0...` format or literal `none`. Used for channel catch-up in session flows — the Routine itself does not post.|
|Claude Project URL|URL OR `—`|Optional|`https://claude.ai/project/...` link. `—` when no Claude Project exists. Surfaces as a click-through in the briefing.|

Missing optional values use the em-dash `—`. Missing required values mean the row is malformed — the Routine logs and skips.

---

## Producers

|Source|When|
|---|---|
|[`COLLABORATOR_SETUP.md`](../COLLABORATOR_SETUP.md)|At My Tasks canvas creation — template includes an empty Routine Config table (single `—` row) or pre-populated rows harvested from Hubs during setup.|
|[`mirrors/CONTROL_TOWER.md`](../mirrors/CONTROL_TOWER.md) missing-project fallback|When a project is mentioned that isn't yet in My Tasks — writes a new row to both Project Registry and Routine Config.|
|The collaborator, manually|When adding a project they've been told to track. Use full-replace only — read the canvas, append the row, write the entire canvas back.|
|The Routine itself|At end of run when writing the daily briefing, the Routine preserves the existing Routine Config rows verbatim. It does **not** re-derive the table; it only reads it.|

Full-replace writes only. Slack Canvas section-level replace on tables creates a duplicate ghost table that cannot be removed via API.

---

## Consumers

|Consumer|What it reads|Failure mode if format drifts|
|---|---|---|
|The Routine (daily briefing)|All six columns|Cannot parse row → project silently drops from the briefing. The collaborator notices when a project they expected to see isn't there.|
|The Control Tower|All six columns (via My Tasks)|Cannot parse row → project is invisible to the cross-project standing report.|

The Routine is the only writer of the rest of the My Tasks canvas (briefing body, "Waiting On You", etc.) but treats Routine Config as read-only input.

---

## Invariant — Routine Config stays in sync with Project Registry

My Tasks has two tables side by side:

- `## Project Registry` — human navigation. Columns: Project | Channel | Field Reference | Added. Humans read this.
- `## Routine Config` — machine parsing. Columns above. Routines and Control Tower read this.

Every project row in one table must have a corresponding row in the other. Producers writing a new project row must write to both. The Control Tower's missing-project fallback calls this out explicitly: "Write rows to both."

A project present in Project Registry but missing from Routine Config is invisible to every automation. A project present in Routine Config but missing from Project Registry is invisible to the human. Both states are bugs.

---

## Change safety

**Non-breaking (additive):**
- Populating the `Claude Project URL` column for a row where it was `—`.
- Changing a `Channel ID` value from `none` to a real channel (or vice versa).
- Adding new project rows.

**Breaking:**
- Adding a column — every consumer must be updated to accept the new schema before a producer writes it.
- Renaming a column header.
- Reordering columns — consumers index by column position in some paths.
- Removing the placeholder `—` row when the table is otherwise empty (Slack render breaks).

---

## Live example — Josh's My Tasks (subset)

```
## Routine Config

> This table is read by the Routine. Do not edit unless reconfiguring project IDs.

|Project ID|Claude Canvas ID|Hub Canvas ID|Human Canvas ID|Channel ID|Claude Project URL|
|---|---|---|---|---|---|
|AES-PLBSYS|F0ASQ0250NN|F0ASLHJQ57X|F0ASTHBK3PW|none|https://claude.ai/project/...|
|TFAB-GRIEVEHOOD|F0ASUCJ72HM|F0AT6E1N9L5|F0ASUCRTSRM|none|—|
```

---

*contracts/my-tasks-routine-config.md v1.0 | April 2026 | Josh Payne*
*Producer of record: COLLABORATOR_SETUP.md (My Tasks template, `## Routine Config` section, lines 182–188)*
