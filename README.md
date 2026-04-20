# Project Routines

PLB-specific implementation of canvas-routines — daily cross-project task briefing automation for Joby Advanced Manufacturing.

Forks: [joshpayne-joby/canvas-routines](https://github.com/joshpayne-joby/canvas-routines)

## What This Is

This repo is the shared brain for every collaborator's Project Routine. A Routine is a scheduled Claude Code automation that runs daily on Anthropic's cloud infrastructure. Each collaborator has one Routine that iterates across all their active projects, builds a personal "My Tasks" canvas in Slack, and handles notifications.

**Base behavior** comes from canvas-routines: read a Registry, fetch sources, synthesize, write back.

**PLB extensions** in this repo add:
- Reading Claude Canvases (Task Boards) and Hub Canvases — not just flat Slack canvases
- Session log and Blockers log parsing
- Task filtering by assignee (matched by Slack user ID in section headers)
- Project-aware scorecard (active projects, sessions yesterday, blockers by age)
- Auto-registration: discovers projects the collaborator was added to and adds them to their Registry
- Check-in DMs to PM when any project has new session activity
- Google Drive mirroring for version history

## How It Works

1. Each collaborator creates a Routine at claude.ai/code/routines pointing at this repo
2. The Routine reads `CLAUDE.md` at the start of every run for behavior instructions
3. The Routine reads the collaborator's "My Tasks" canvas in Slack for their project list
4. The Routine fetches each project's Claude Canvas (Task Board), builds a cross-project briefing, and writes the "My Tasks" canvas back

## Key Files

- `CLAUDE.md` — The behavior spec. Every Routine reads this. Update it → every collaborator gets the update on the next run.
- `prompts/` — Routine prompts per collaborator. Copy `TEMPLATE.md`, fill in identity fields, use as-is.

## Setup

1. Get access to this repo (Joby SSO)
2. Create your "My Tasks" canvas in Slack (your PM or a Project session can do this)
3. Go to claude.ai/code/routines → New routine
4. Point at this repo, copy a prompt from `prompts/`, add a schedule trigger
5. Hit "Run now" to test

For the generic setup guide (not PLB-specific), see [canvas-routines/setup/CANVAS_SETUP.md](https://github.com/joshpayne-joby/canvas-routines/blob/main/setup/CANVAS_SETUP.md).

## Maintained By

Josh Payne — Equipment Engineering Manager, Advanced Manufacturing
