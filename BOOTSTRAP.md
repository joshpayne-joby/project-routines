# BOOTSTRAP.md
# First-time setup of seed in a new Slack workspace
# Version 0.1 | April 2026 | Josh Payne

---

## What this is

If you're in a Slack workspace with no seed infrastructure yet, this walks you from zero to "my first project is live." Expect about 30 minutes.

- **Already have a seed workspace and want your own daily briefing?** Go to [`COLLABORATOR_SETUP.md`](COLLABORATOR_SETUP.md) instead.
- **Already have a seed workspace and want to stand up a new project?** Go to [`PROJECT_SETUP.md`](PROJECT_SETUP.md) instead.

This doc is for the *first person* in a workspace — the one who has to create the shared canvases the provisioners depend on.

---

## Prerequisites

- Slack workspace where you can create canvases
- Claude Pro/Team with Claude Projects + Slack + GitHub connectors
- GitHub account (to fork this repo)
- ~30 minutes

---

## What you'll have at the end

- Your fork of this repo with your workspace's IDs plugged in
- Two workspace-wide Slack canvases: **Prime Project** (behavior spec) and **seed Registry** (project index)
- Your **My Tasks** canvas and a live **seed Routine** running on your Claude account
- One live project with its three canvases
- (Optional) A **Control Tower** Claude Project

---

## Phase 1 — Fork the repo

Fork [`joshpayne-joby/project-routines`](https://github.com/joshpayne-joby/project-routines) to your own GitHub org or user.

Why fork: the provisioners hard-code a Slack tenant URL and the seed Registry canvas ID in a handful of places. Forking lets you edit those once and point your Routines and Claude Projects at your fork. Upstream stays Joby-flavored; your fork stays yours.

```
git clone https://github.com/[your-org]/project-routines
cd project-routines
```

---

## Phase 2 — Create the two shared canvases

These are workspace-wide — every project and every collaborator touches them.

### 2a. seed Registry canvas

In Slack, create a new canvas titled `seed Registry`. Paste this body:

```
# seed Registry

Thin index of every active seed project in this workspace. One row per project.

|Project ID|Display Name|Hub|
|---|---|---|
|—|—|—|

Maintained automatically — PROJECT_SETUP.md appends a row when a new project is created. Consumed by COLLABORATOR_SETUP.md (project lookup) and the Control Tower (missing-project fallback).
```

Copy the canvas ID (starts with `F0…`). Keep it handy for Phase 3.

### 2b. Prime Project canvas

In Slack, create a new canvas titled `:seedling: Prime Project | Active Session Behavior`.

*Slack strips emoji shortcodes from canvas titles at creation — the title slot will render without the seedling. The body keeps it. This is a known platform limit; not blocking.*

For the body, open [`mirrors/PRIME_PROJECT_CANVAS.md`](mirrors/PRIME_PROJECT_CANVAS.md) and paste everything from the `# :seedling: Prime Project | Active Session Behavior` heading down to the footer. (Skip the four comment lines at the very top — those are mirror metadata.)

Copy the canvas ID. You'll need it in Phase 3.

---

## Phase 3 — Plug your workspace IDs into your fork

Open your local clone. You'll edit three files.

**Find-and-replace these across the repo:**

| Find | Replace with |
|---|---|
| `F0AUJ8FV6JH` | Your seed Registry canvas ID (from Phase 2a) |
| `F0AU9KARVQ8` | Your Prime Project canvas ID (from Phase 2b) |
| `jobyaviation.enterprise.slack.com` | Your Slack tenant URL (e.g. `yourco.slack.com`) |
| `jobyaviation.slack.com` | Same as above |
| `joshpayne-joby/project-routines` | `[your-org]/project-routines` |

Files that contain these refs:
- [`COLLABORATOR_SETUP.md`](COLLABORATOR_SETUP.md) (Configuration block + prompt template + close steps)
- [`PROJECT_SETUP.md`](PROJECT_SETUP.md) (Register step + Prime canvas link in close)
- [`mirrors/CONTROL_TOWER.md`](mirrors/CONTROL_TOWER.md) (Step 1 fallback + Bootstrap block)

Commit and push:

```
git add -A
git commit -m "chore: seed workspace config"
git push
```

---

## Phase 4 — Become your own first collaborator

You need a My Tasks canvas + a live Routine before Phase 5 makes sense. Run `COLLABORATOR_SETUP.md` on yourself:

1. In Claude, create a Claude Project named `seed | Collaborator Setup` (you'll re-use this Project every time you onboard someone new).
2. Paste your fork's `COLLABORATOR_SETUP.md` into **Project Instructions**.
3. Connect **Slack** and **GitHub** connectors.
4. Open a chat: "let's set me up."
5. Answer the three questions. You'll walk away with:
   - A `[Your Name] | My Tasks` canvas
   - A generated `[your-slug].md` prompt file
6. Follow the closing steps:
   - Commit the generated prompt file to your fork at `prompts/[your-slug].md` (your Routine will read it from there)
   - Go to [claude.ai/code/routines](https://claude.ai/code/routines) → **New routine**
   - Point it at your fork, branch `main`
   - Paste the prompt contents into the Routine
   - Set schedule: weekdays 6:30 AM (or whatever fits)
   - Hit **Run now**

Within a minute your My Tasks canvas populates. You are now a live seed collaborator.

---

## Phase 5 — Create your first project

1. In Claude, create a Claude Project named whatever fits (e.g. `Aircraft-Floor-Layout`). This becomes the project's permanent Claude home.
2. Paste your fork's `PROJECT_SETUP.md` into **Project Instructions**.
3. Connect **Slack** and **GitHub** connectors.
4. Open a chat: "let's get this project set up."
5. Answer the three questions.

`PROJECT_SETUP.md` creates:
- **Task Board** (Claude Canvas) — live project state
- **Field Reference** (Human Canvas) — photos, drawings, sign-offs
- **Project Hub** (with a `## Canvas Registry` KV section inside)
- A row appended to your seed Registry
- A `PROJECT_CONTEXT.md` file on the Claude Project

Once it's done, flip the Claude Project from "provisioner mode" to "active-session mode":

1. Open [`docs/PROJECT_INSTRUCTIONS_WRAPPER.md`](docs/PROJECT_INSTRUCTIONS_WRAPPER.md).
2. Copy the template block between the `===` lines.
3. Fill in the five bracketed values (`project_id`, `tasks_canvas_id`, `human_canvas_id`, `hub_canvas_id`, `project_channel_id`) from the `PROJECT_CONTEXT.md` that `PROJECT_SETUP.md` just generated.
4. Replace the Claude Project's Instructions field with the filled-in wrapper.

The wrapper is a thin fetcher that reads the Prime Project canvas at session start, so behavior updates ripple through every project automatically.

Finally, add a row to the **Project Registry** table in your My Tasks canvas pointing at this project (columns: Project / Channel / Field Reference / Added). The Routine picks it up on the next run.

---

## Phase 6 — (Optional) Stand up Control Tower

Skip this until you have 2+ active projects. For one project, the Control Tower is overkill — just open that project directly.

When ready:

1. In Claude, create a Claude Project named `[Your Name] | Project Control Tower`.
2. Paste your fork's [`mirrors/CONTROL_TOWER.md`](mirrors/CONTROL_TOWER.md) into **Project Instructions**. (The live canvas and the mirror file have the same content — either works as the source.)
3. Connect **Slack**.
4. In the Instructions, edit the **Bootstrap — Per-Person Config** block at the bottom with your values:

   ```
   my_tasks_canvas_id: [your My Tasks canvas ID from Phase 4]
   my_tasks_canvas_url: https://[your tenant]/docs/[workspace]/[my_tasks_canvas_id]
   owner_slack_id: [your Slack user ID]
   owner_name: [your name]
   ```

5. Open a chat. You'll get a cross-project standing report in under 30 seconds.

---

## Where things can go wrong

- **Claude GitHub App not installed at your org.** Your Routine can't read your fork. Install the Anthropic GitHub App at your org's GitHub settings first — one-time step, typically needs org admin.
- **Slack user ID lookup fails.** Paste your Slack user ID directly (Slack profile → "More" → "Copy member ID"). Provisioners handle this fallback gracefully.
- **Canvas title slot emoji stripped.** Slack drops emoji shortcodes from title slots at creation and the API can't update titles after the fact. Body h1 renders clean; the title slot is what it is. Not blocking — visible only at the canvas-list level.
- **Canvas write fails with a "section" error.** The Slack Canvas API has a confirmed bug on section-level replace that appends ghost duplicates. Every seed write pattern uses full-replace only. If a provisioner hits this, it falls back to pasting the full canvas content in chat so you can paste manually.
- **Routine runs but My Tasks is empty.** Open My Tasks → check the **Project Registry** table. If all rows are `—`, you haven't registered a project yet. Finish Phase 5, add the project to the Registry, trigger **Run now** again.

---

## Staying current with upstream

Upstream `joshpayne-joby/project-routines` gets updates to `CLAUDE.md`, the Prime Project canvas, and the contracts. To pull them into your fork:

```
git remote add upstream https://github.com/joshpayne-joby/project-routines
git fetch upstream
git merge upstream/main
```

Resolve conflicts (your workspace-specific edits should stay); push. If the Prime Project canvas has changed, re-paste the body from [`mirrors/PRIME_PROJECT_CANVAS.md`](mirrors/PRIME_PROJECT_CANVAS.md) into your live canvas (full-replace).

---

*BOOTSTRAP.md v0.1 — April 2026 — Josh Payne. First-time-in-a-workspace walkthrough. Assumes zero prior seed infrastructure. Iterate from here — this is the v0.1 linear cut.*

---

*Part of the [seed](https://github.com/joby/project-routines) framework. Maintained by Josh Payne. First-run walkthrough — update here when the provisioner or canvas contracts change.*
