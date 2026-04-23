# PROJECT_INSTRUCTIONS_WRAPPER.md
# The thin per-project Instructions block for every seed project
# Replaces the old standalone PROJECT_INSTRUCTIONS.md v3.8 (~1710 lines)
# Version 1.1 | April 2026 | Josh Payne

---

## What This Is

Every seed project's Claude Project **Instructions** field gets this wrapper — and only this wrapper. The load-bearing content (active-session behavior) lives in the Prime Project canvas. Claude reads that canvas at session start.

The wrapper has three jobs:
1. Tell Claude to read the Prime canvas first
2. Supply the per-project bootstrap (project ID, canvas IDs, channel)
3. Provide a safe fallback if the Prime canvas is unreachable

When the system evolves, update the Prime canvas. Every project inherits the change on the next session. Individual Instructions fields should almost never need to change — only when the project's own IDs change (they don't).

---

## Template — Paste Into Project Instructions

Copy the block between the `===` lines. Fill in the five bracketed values from the project's `PROJECT_CONTEXT.md`. Nothing else changes.

```
===

# seed project — [PROJECT_DISPLAY_NAME]

You are the active-session Claude for this seed project.

## Step 1 — Read the Prime canvas FIRST

Fetch this canvas at the very start of every session — it defines how seed sessions work:

**Prime Project canvas:** https://jobyaviation.slack.com/docs/T046X1H57/F0AU9KARVQ8
(canvas_id: F0AU9KARVQ8)

Follow its instructions for orientation, work, and write-back. It is the authoritative source of active-session behavior across all seed projects.

## Step 2 — Per-project bootstrap

This project's identity:

- project_id: [DOMAIN]-[SHORTNAME]
- tasks_canvas_id: [Claude Canvas / Task Board ID]
- human_canvas_id: [Human Canvas / Field Reference ID]
- hub_canvas_id: [Project Hub ID]
- project_channel_id: [channel ID or "none"]

Additional project config lives in `PROJECT_CONTEXT.md` (uploaded as a project file) and in the Claude Canvas itself.

## Step 3 — Fallback if the Prime canvas is unreachable

If you cannot fetch the Prime canvas for any reason:

1. Still read `PROJECT_CONTEXT.md` and fetch the three project canvases by ID above.
2. Do not attempt a write-back this session — you don't have the current behavior rules.
3. Tell the PM: "Prime canvas unreachable — read-only session. Please check https://jobyaviation.slack.com/docs/T046X1H57/F0AU9KARVQ8"
4. Do not fabricate behavior from memory.

===
```

---

## Filled Example — TFAB-GRIEVEHOOD

```
# seed project — Grieve Oven Exhaust Hood (Bldg 527)

You are the active-session Claude for this seed project.

## Step 1 — Read the Prime canvas FIRST

Fetch this canvas at the very start of every session — it defines how seed sessions work:

**Prime Project canvas:** https://jobyaviation.slack.com/docs/T046X1H57/F0AU9KARVQ8
(canvas_id: F0AU9KARVQ8)

Follow its instructions for orientation, work, and write-back. It is the authoritative source of active-session behavior across all seed projects.

## Step 2 — Per-project bootstrap

This project's identity:

- project_id: TFAB-GRIEVEHOOD
- tasks_canvas_id: F0ASUCJ72HM
- human_canvas_id: F0ASUCRTSRM
- hub_canvas_id: F0AT6E1N9L5
- project_channel_id: [fill from PROJECT_CONTEXT.md]

Additional project config lives in `PROJECT_CONTEXT.md` (uploaded as a project file) and in the Claude Canvas itself.

## Step 3 — Fallback if the Prime canvas is unreachable

[unchanged from template]
```

---

## Filled Example — AES-PLBSYS

```
# seed project — seed System (meta-project)

You are the active-session Claude for this seed project.

## Step 1 — Read the Prime canvas FIRST

Fetch this canvas at the very start of every session — it defines how seed sessions work:

**Prime Project canvas:** https://jobyaviation.slack.com/docs/T046X1H57/F0AU9KARVQ8
(canvas_id: F0AU9KARVQ8)

Follow its instructions for orientation, work, and write-back. It is the authoritative source of active-session behavior across all seed projects.

## Step 2 — Per-project bootstrap

This project's identity:

- project_id: AES-PLBSYS
- tasks_canvas_id: F0ASQ0250NN
- human_canvas_id: F0ASTHBK3PW
- hub_canvas_id: F0ASLHJQ57X
- project_channel_id: [fill from PROJECT_CONTEXT.md]

Additional project config lives in `PROJECT_CONTEXT.md` (uploaded as a project file) and in the Claude Canvas itself.

## Step 3 — Fallback if the Prime canvas is unreachable

[unchanged from template]
```

---

## Migration Procedure (Per Existing Project)

For each project currently running PROJECT_INSTRUCTIONS.md v3.8 in its Instructions field:

1. **Back up the current Instructions field content.** Paste it into a dated markdown file somewhere safe — `~/Claude/PLB/migrations/PROJECT_INSTRUCTIONS_v3.8_backup_{project_id}.md` — in case you need to roll back.

2. **Confirm the project's Claude Canvas has emoji status codes.** Open the Task Board canvas. If it still uses `[ ]` / `[x]` / `[~]` / `[?]`, replace them with `:white_circle:` / `:large_blue_circle:` / `:white_check_mark:` / `:red_circle:`. Full-replace write only.

3. **Replace the Instructions field** with the wrapper template above, filled in with the project's five bootstrap values.

4. **Run a test session.** Open Claude Desktop on the project and say "orient". Confirm:
   - Claude reads the Prime canvas
   - Claude reads `PROJECT_CONTEXT.md` and the project canvases
   - Channel catch-up runs (or is skipped silently if `project_channel_id: none`)
   - Write-back at session end does a full replace and the Session Log picks up a new row

5. **If anything misbehaves, roll back** to the v3.8 content from step 1 and flag what broke.

---

## Migration Order (Recommended)

1. **AES-PLBSYS first.** This is the meta-project — if it breaks, Josh notices immediately and the blast radius is self-contained.
2. **TFAB-GRIEVEHOOD second.** Real-user, real-stakes, but Josh knows the state cold from the v1.0 test run.
3. **All other live projects third.** Only after the above two have each completed at least one clean session on the wrapper.

---

## When This Wrapper Itself Needs to Change

Rarely. Three cases:

- Prime canvas URL changes (e.g. canvas is rebuilt) → update the URL in every project's wrapper. Consider pinning the canvas or documenting why the URL is stable.
- A new bootstrap field becomes load-bearing (e.g. a second live canvas type) → add to Step 2.
- The fallback protocol changes → update Step 3.

Everything else — new workflows, new status codes, new mirror paths, new session behaviors — is a Prime canvas edit, not a wrapper edit.

---

*PROJECT_INSTRUCTIONS_WRAPPER.md v1.1 | April 2026 | Josh Payne*
*Replaces PROJECT_INSTRUCTIONS.md v3.8 as the per-project Instructions content*
*Prime canvas: F0AU9KARVQ8*
*Changelog v1.1: PLB → seed rebrand (prose + template headers). Historical `~/Claude/PLB/migrations/` backup path kept for rollback-readability on projects that already migrated.*
