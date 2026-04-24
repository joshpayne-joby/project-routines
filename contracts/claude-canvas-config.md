# Claude Canvas Configuration — Field Directory

Every project's Claude Canvas (Task Board) has a `## seed Configuration` section near the top. It's a KV block of machine-readable fields — grouped under `###` subheaders (Identity, Canvas Registry, Communication, etc.) — that drive session behavior: channel catch-up, digest posts, photo breadcrumbs, mirror path, session memory.

Unlike `hub-canvas-registry.md` and `my-tasks-routine-config.md`, this contract is **not** a parse-format lock. It's a **field directory**: for each field, where is it written, where is it read, and what happens when it drifts or goes missing.

The `## seed Configuration` block grows. New fields land in the template regularly (digest, photo, code integration). This directory is the index so you don't have to grep four files to know who cares about `photo_staging_dm_id`.

---

## Canonical structure

The full block lives in [`PROJECT_SETUP.md`](../PROJECT_SETUP.md) lines 208–258 as the creation template. Producer-of-record.

```
## seed Configuration

project_id: ...
project_display_name: ...
...

### Canvas Registry (locked at creation)

tasks_canvas_id: ...
...

### Communication

project_channel_id: ...
...
```

Prose form — no code fence. Keys are `snake_case`, values on the same line after `: `. Groups are separated by `###` subheaders (Identity is implicit — first group under the `## seed Configuration` heading, no subheader). Claude is the parser, not a regex: format flexibility is fine as long as keys, values, and group intent are legible.

---

## Field groups

Fields are grouped by purpose. Groups are comment-delimited in the block (`# Canvas Registry`, `# Communication`, etc.). The comments are not parsed — they're for humans.

### Identity

|Field|Type|Required|Producer|Consumer|Notes|
|---|---|---|---|---|---|
|`project_id`|string `DOMAIN-SHORTNAME`|Yes|PROJECT_SETUP creation|Every session at orient|Must match Hub Canvas Registry + My Tasks Routine Config.|
|`project_display_name`|plain text|Yes|PROJECT_SETUP creation|Sessions for briefing output|Human name. Matches Claude Project name.|
|`project_original_name`|plain text|No|PROJECT_SETUP creation|Humans reading the canvas|Historical record at kickoff; rarely read.|
|`seed_mirror_filename_prefix`|string `seed-[project_id]`|Yes|PROJECT_SETUP creation|Session end (mirror filename)|Pure formatting convention. Hard-coded to `seed-[project_id]`.|
|`file_ecosystem`|`google` / `microsoft` / `mixed`|Yes|PROJECT_SETUP creation|Session file lookups|Which connector to prefer.|
|`primary_site`|site code|Yes|PROJECT_SETUP creation|Hierarchy context|Informational; no parser reads this today.|
|`team_sites`|comma-separated|No|PROJECT_SETUP creation|Informational|No parser reads this today.|

### Canvas Registry (locked at creation)

|Field|Type|Required|Producer|Consumer|Notes|
|---|---|---|---|---|---|
|`tasks_canvas_id`|Slack canvas ID `F0...`|Yes|PROJECT_SETUP creation|Self-reference; never change|This IS the canvas this block lives in.|
|`tasks_canvas_url`|URL|Yes|PROJECT_SETUP creation|Briefing output|Click-through.|
|`human_canvas_id`|Slack canvas ID `F0...`|Yes|PROJECT_SETUP creation|Sessions at orient|Read-only for Claude.|
|`human_canvas_url`|URL|Yes|PROJECT_SETUP creation|Briefing output|Click-through.|
|`hub_canvas_id`|Slack canvas ID `F0...`|Yes|PROJECT_SETUP creation|Sessions at orient; Control Tower fallback|Hub is read at session start.|
|`hub_canvas_url`|URL|Yes|PROJECT_SETUP creation|Briefing output|Click-through.|

"Locked at creation" means: **do not change these after the canvas exists**. The canvas IDs are tied to the Slack canvases themselves; renaming or recreating a canvas requires a coordinated update across the Claude Canvas, Hub Canvas Registry, and My Tasks Routine Config.

### Communication

|Field|Type|Required|Producer|Consumer|Notes|
|---|---|---|---|---|---|
|`project_channel_id`|Slack channel ID `C0...` OR `none`|Yes|PROJECT_SETUP creation|Prime canvas session orient; channel catch-up; digest post|`none` skips channel catch-up silently.|
|`project_channel_type`|`channel` / `group_dm` / `none`|Yes|PROJECT_SETUP creation|Session catch-up path|Distinguishes public channel from DM.|

### Photo Breadcrumbs

|Field|Type|Required|Producer|Consumer|Notes|
|---|---|---|---|---|---|
|`photo_staging_dm_id`|Slack DM channel ID or blank|No|PROJECT_SETUP creation|Prime canvas Photo Breadcrumbs step|Blank disables photo scan.|
|`photo_project_keywords`|comma-separated|No|PROJECT_SETUP creation|Photo match heuristic|Names/nicknames for matching DM photos to this project.|
|`photos_drive_folder_url`|URL or blank|No|PROJECT_SETUP creation|Informational for PM|Target folder for manual uploads.|

### Mirror

|Field|Type|Required|Producer|Consumer|Notes|
|---|---|---|---|---|---|
|`seed_mirror_drive_folder_url`|URL or blank|Optional|PM sets manually|Manual-path filename context|Target folder for saved exports.|
|`seed_mirror_script_url`|Apps Script URL or blank|Automated only|PM sets after Apps Script deploy|Session end automated-mirror path|Blank = manual path; populated = automated path.|

See [`docs/MIRROR_PATHS.md`](../docs/MIRROR_PATHS.md) for how these fields interact.

### Session state

|Field|Type|Required|Producer|Consumer|Notes|
|---|---|---|---|---|---|
|`last_session_date`|ISO date `YYYY-MM-DD`|Yes|Every session at write-back|Next session orient; channel catch-up window|The single most-written field. Updated to today's date on every close.|
|`pending_photo_transfer`|`true` / `false`|No|Session when photos queued|Next session|Default `false`.|
|`pending_photo_count`|integer|No|Session when photos queued|Next session|Default `0`.|

### Digest (known drift — not in current PROJECT_SETUP template)

The Prime canvas references three digest fields that are **not emitted by PROJECT_SETUP's creation template as of v1.8**:

|Field|Type|Default|Producer|Consumer|Notes|
|---|---|---|---|---|---|
|`digest_enabled`|`true` / `false`|`false` (implicit)|PM sets manually|Session start reply scan; session end digest post|If unset, digest system is inactive.|
|`digest_channel_id`|Slack channel ID|Falls back to `project_channel_id`|PM sets manually|Digest post target|Separate channel for digest-only posts.|
|`digest_escalation_sessions`|integer|`5`|PM sets manually|Escalation marker threshold|Sessions before a Pending item gets `:warning:` flag.|

**Resolve: either add these to the PROJECT_SETUP v1.8 template with sensible defaults, or move the digest system to "opt-in, PM writes fields manually" and document that clearly.** This directory surfaces the drift — next edit to PROJECT_SETUP should close it.

### Code Integration (optional, rarely used)

All fields blank unless the project has a paired code repo. No session parser reads these today — written once, consumed by humans for context.

|Field|Type|
|---|---|
|`repo_path`|local path|
|`repo_remote`|git remote URL|
|`repo_platform`|`github` / `gitlab` / ...|
|`default_branch`|branch name|
|`dev_command`|shell command|
|`dev_url`|localhost URL|
|`github_channel_id`|Slack channel ID|
|`branching_strategy`|`trunk` / `gitflow` / ...|
|`architecture_canvas_id`|Slack canvas ID|
|`architecture_canvas_url`|URL|

---

## Producers — who writes this block

|Producer|When|What they write|
|---|---|---|
|[`PROJECT_SETUP.md`](../PROJECT_SETUP.md)|At project creation (one-shot)|Entire block, all identity/registry/communication/mirror/photo fields populated with real values.|
|Active-session Claude|At every session end (write-back)|Preserves all fields verbatim except `last_session_date` (updated to today) and any session-queued state (`pending_photo_*`).|
|PM (manually)|Ad hoc — enabling digest, upgrading mirror, adding photo DM|Edits a single field. Full-replace canvas write only; section-replace corrupts.|

Full-replace writes only. Read the whole canvas, edit in memory, write the whole canvas. The Slack Canvas API section-replace bug applies here as everywhere.

---

## Consumers — who reads this block

|Consumer|Fields read|Failure mode if missing|
|---|---|---|
|[`mirrors/PRIME_PROJECT_CANVAS.md`](../mirrors/PRIME_PROJECT_CANVAS.md) session orient|Identity, Canvas Registry, `project_channel_id`, `last_session_date`|Cannot orient; session starts blind.|
|Prime canvas channel catch-up|`project_channel_id`, `last_session_date`|If `project_channel_id` missing or `none` → skip silently.|
|Prime canvas digest system|`digest_*` fields|All default-off; missing = feature inactive.|
|Prime canvas photo breadcrumbs|`photo_staging_dm_id`, `photo_project_keywords`|Blank `photo_staging_dm_id` disables feature.|
|Prime canvas mirror path|`seed_mirror_script_url`, `seed_mirror_drive_folder_url`, `seed_mirror_filename_prefix`|Missing `seed_mirror_script_url` → manual path.|

---

## When a new field is added

1. Add it to the `PROJECT_SETUP.md` creation template (usually at lines 208–258) with a sensible default.
2. Add it to this directory in the right group with producer/consumer/notes.
3. Update the Prime canvas prose if the new field is session-behavior-bearing.
4. Re-snapshot `mirrors/PRIME_PROJECT_CANVAS.md`.
5. No retroactive edit to existing Claude Canvases is required — fields default to missing. Missing fields must fail gracefully (the "digest" group demonstrates this — digest is off by default).

---

## Invariants

- **Canvas Registry fields are write-once.** `tasks_canvas_id`, `human_canvas_id`, `hub_canvas_id` do not change after creation.
- **`last_session_date` is the single most-written field.** Updated every session. Any session that doesn't update it has a broken write-back.
- **Missing optional fields fail off.** Digest, photo breadcrumbs, automated mirror — all three degrade to "feature inactive" when fields are missing.
- **The block is self-contained.** No field references outside the canvas. Consumers fetch this canvas and parse; they don't cross-reference Hub or My Tasks.

---

*contracts/claude-canvas-config.md v0.6 | April 2026 | Josh Payne*
*Directory scope, not format lock. Producer of record: PROJECT_SETUP.md v1.8 (lines 208–258). Known drift: digest_* fields (see Digest group above). v0.6: walked back the "fence is load-bearing" claim from v0.5 — recon across live canvases showed it wasn't earned. Dropped `seed_mirror_path` from required fields; path is inferred from `seed_mirror_script_url` presence.*
