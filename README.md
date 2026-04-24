# seed

A Claude-native project workspace framework for Joby Advanced Manufacturing.

Seed turns Slack Canvases into a shared project substrate — machine-readable state that both people and Claude work from. Each project gets three canvases; each collaborator gets a daily cross-project briefing built by a scheduled Claude automation; everything stays reachable whether you use Claude Desktop, Claude Code, or just Slack.

This repo is the commons — the behavior spec, the provisioners, and the contracts that hold the whole system together.

---

## Start here

Pick the path that matches where you're coming in:

### 🌱 I'm the first person setting this up in my workspace
You're the **first adopter**. There's no seed infrastructure yet — no Prime canvas, no Registry, nothing. Go to [`BOOTSTRAP.md`](BOOTSTRAP.md) — fork the repo, create two shared canvases, onboard yourself, spin up your first project. ~30 minutes. Do this once per workspace.

### 👤 I'm on a project and want my own daily briefing
You're a **collaborator**. Go to [`COLLABORATOR_SETUP.md`](COLLABORATOR_SETUP.md) — drop it into a Claude Project as Instructions, answer three questions, walk away with your "My Tasks" canvas and a ready-to-run Routine. 10 minutes.

### 🏗️ I need to spin up a new project
You're a **project owner**. Go to [`PROJECT_SETUP.md`](PROJECT_SETUP.md) — drop it into a Claude Project as Instructions, describe the project in one sentence, walk away with three Slack canvases and a registered project. 15 minutes.

### 🧠 I want to understand or change how this works
You're an **architect** (or curious). Start with [`CLAUDE.md`](CLAUDE.md) — the behavior spec every Routine reads. Then read [`mirrors/PRIME_PROJECT_CANVAS.md`](mirrors/PRIME_PROJECT_CANVAS.md) for active-session behavior and [`mirrors/CONTROL_TOWER.md`](mirrors/CONTROL_TOWER.md) for the cross-project view. The [`contracts/`](contracts/) folder holds the three schemas producers and consumers agree on — [`hub-canvas-registry.md`](contracts/hub-canvas-registry.md), [`my-tasks-routine-config.md`](contracts/my-tasks-routine-config.md), and the [`claude-canvas-config.md`](contracts/claude-canvas-config.md) field directory.

---

## How it fits together

Three layers, all talking through Slack Canvases:

**Per project — three canvases:**
- **Task Board** (Claude Canvas) — live project state. Claude writes; full-replace only.
- **Field Reference** (Human Canvas) — photos, drawings, sign-offs. Team writes; Claude reads.
- **Project Hub** — phase tracker, team roster, onboarding. Both edit occasionally.

**Per collaborator — one Routine:**
- Scheduled Claude Code automation, configured at [claude.ai/code/routines](https://claude.ai/code/routines), pointed at this repo.
- Reads [`CLAUDE.md`](CLAUDE.md) at every run for behavior.
- Reads the collaborator's "My Tasks" canvas for the project list.
- Fetches each project's Task Board, builds a cross-project briefing, writes back.

**Across the whole org — two shared canvases:**
- **Prime Project canvas** — active-session behavior spec. Every project's Claude session reads this first.
- **seed Registry** — thin index of all active seed projects. Consumed by the Control Tower and missing-project fallbacks.

---

## Live examples

Working canvases from Josh's Joby workspace — **not reachable from other Slack tenants.** If you're an adopter standing up seed in your own workspace, treat these as visual reference for what the canvases look like once populated. You'll have your own equivalents after running [`BOOTSTRAP.md`](BOOTSTRAP.md).

| What | Canvas |
|---|---|
| A project's Task Board | [AES-PLBSYS](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0ASQ0250NN) |
| A project's Field Reference | [AES-PLBSYS Field Reference](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0ASTHBK3PW) |
| A project's Hub | [AES-PLBSYS Hub](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0ASLHJQ57X) |
| A collaborator's daily briefing | [Josh's My Tasks](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0ATCQNF59V) |
| The active-session behavior canvas | [Prime Project](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0AU9KARVQ8) |
| The project index | [seed Registry](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0AUJ8FV6JH) |
| The cross-project aggregator | [Control Tower Instructions](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0AUGD2CC9J) |

---

## Repo layout

```
├── README.md                       You are here
├── BOOTSTRAP.md                    First-time setup in a new workspace (one-shot per workspace)
├── CLAUDE.md                       Routine behavior spec (read every run)
├── COLLABORATOR_SETUP.md           Provisioner — new-collaborator Routine + My Tasks canvas
├── PROJECT_SETUP.md                Provisioner — new project + three canvases + Registry row
├── mirrors/                        Versioned snapshots of live Slack canvases
│   ├── PRIME_PROJECT_CANVAS.md     Active-session behavior (canvas F0AU9KARVQ8)
│   └── CONTROL_TOWER.md            Cross-project aggregator (canvas F0AUGD2CC9J)
├── docs/                           Supporting specs and how-tos
│   ├── MIRROR_PATHS.md             Manual vs. automated session-mirror paths
│   └── PROJECT_INSTRUCTIONS_WRAPPER.md  Thin per-project Claude Project Instructions wrapper
├── contracts/                      Cross-cutting schemas (what consumers/producers agree on)
│   ├── hub-canvas-registry.md      Format of the Hub's `## Canvas Registry` KV section
│   ├── my-tasks-routine-config.md  Schema of My Tasks `## Routine Config` table
│   └── claude-canvas-config.md     Field directory for the `## seed Configuration` block
├── prompts/                        Per-collaborator Routine prompt files
│   └── TEMPLATE.md                 Blank template — copy + fill
└── notes/                          Maintainer scratchpad — tangents, worklog, in-flight thinking
```

---

## Known limits

- **GitHub App org install pending.** Collaborators can't merge PRs against this repo directly yet — the maintainer commits on their behalf. Drafts and patches welcome via any channel.
- **Routine creation is browser-only.** No API path. The last step of `COLLABORATOR_SETUP.md` is "paste into [claude.ai/code/routines](https://claude.ai/code/routines) and click New routine."
- **Slack Canvas writes are full-replace only.** The Slack Canvas API has a confirmed bug where section-level replace on a table creates a ghost duplicate that can't be removed via API. Every write pattern in the system accounts for this.
- **Slack canvas title slot is API-unsettable.** The `title` parameter on create works; updates to the title after creation require the Slack UI.

---

## Maintainer

Josh Payne — Joby Aviation, Advanced Manufacturing.

Questions, issues, or patches: find me in Slack (`@Josh Payne`) or open an issue on this repo.
