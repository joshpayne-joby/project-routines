# Project Routines

Shared behavior definitions for Project Routines — daily cross-project task briefing automation for Joby Advanced Manufacturing.

## What This Is

This repo is the shared brain for every collaborator's Project Routine. A Routine is a scheduled Claude Code automation that runs daily on Anthropic's cloud infrastructure. Each collaborator has one Routine that iterates across all their active projects, builds a personal "My Tasks" canvas in Slack, and handles notifications.

## How It Works

1. Each collaborator creates a Routine at claude.ai/code/routines pointing at this repo
2. The Routine reads `CLAUDE.md` at the start of every run for behavior instructions
3. The Routine reads the collaborator's "My Tasks" canvas in Slack for their project list
4. The Routine fetches each project's Claude Canvas (Task Board), builds a cross-project briefing, and writes the "My Tasks" canvas back

## Key Files

- `CLAUDE.md` — The behavior spec. Every Routine reads this. Update it → every collaborator gets the update on the next run.
- `COLLABORATOR_SETUP.md` — Provisioner for new-collaborator onboarding. Drop into a Claude Project as Instructions to spin up a Routine + My Tasks canvas in one conversation.
- `PRIME_PROJECT_CANVAS.md` — Source-of-truth copy of the PLB Prime Project Canvas content. The live canvas in Slack (F0AU9KARVQ8) is what every active Claude Project session reads; this file is the versioned mirror so changes have durable history.
- `prompts/` — Template Routine prompts for the creation form. Copy, personalize, paste.

## Setup

**Guided path (recommended):**

1. Create a new Claude Project
2. Paste `COLLABORATOR_SETUP.md` as the Project Instructions
3. Open a chat and answer three questions — the provisioner creates your "My Tasks" canvas, generates your Routine prompt, and hands you the copy-paste block for claude.ai/code/routines

**Manual path:**

1. Get access to this repo (Joby SSO)
2. Create your "My Tasks" canvas in Slack (your PM or a Project session can do this)
3. Go to claude.ai/code/routines → New routine
4. Point at this repo, copy a prompt from `prompts/`, add a schedule trigger
5. Hit "Run now" to test

## Maintained By

Josh Payne — Equipment Engineering Manager, Advanced Manufacturing
