# MIRROR_PATHS.md
# Session mirror — how seed canvas snapshots get durable storage
# Version 1.1 | April 2026 | Josh Payne

> **The problem:** Slack Canvases are the live seed surface but they have no version history. Every seed project needs a durable snapshot of each session.
>
> **The solution:** Two paths. Manual is the default — works from day one, zero infrastructure. Automated is an opt-in upgrade when the PM is ready.

---

## Path 1 — Manual Export (Default)

**What it is:** At session end, Claude outputs the full Claude Canvas content as a downloadable markdown file. The PM saves it to Drive or SharePoint.

**Why it's the default:**
- Works immediately on any new project
- No Apps Script, no bot approvals, no IT ticket
- PM fully in control of where the file lives

**How it works:**

1. Session ends → Claude does the full-replace write to the Claude Canvas
2. Claude generates a markdown file with the same content
3. Filename: `{seed_mirror_filename_prefix}-{YYYY-MM-DD}.md`
   (e.g. `seed-AES-PLBSYS-2026-04-16.md`)
4. Claude offers the file as a downloadable artifact in chat
5. PM downloads → drops into the project's Drive/SharePoint folder

**PM setup:** Create one folder for seed mirrors. Save the URL in the Claude Canvas as `seed_mirror_drive_folder_url`.

**Trade-offs:**
- Manual step each session (30 seconds)
- No automatic thread-link back to Slack
- PM has to remember to save it — but even if they don't, the canvas still holds current state

---

## Path 2 — Automated Mirror (Opt-in Upgrade)

**What it is:** Claude posts the full canvas content to `#plb-mirror`. An Apps Script watches the channel, creates a timestamped Google Doc, and threads the URL back as a reply.

**Why it's an upgrade, not the default:**
- Requires one-time Apps Script setup per team/workspace
- Only works for Google Workspace teams (no SharePoint equivalent yet)
- Posting to `#plb-mirror` is a standard Slack message — no special bot scopes needed. The pending `files:read` approval is *not* required for this path.

**How it works:**

1. Session ends → Claude does the full-replace write to the Claude Canvas
2. Claude posts a message to `#plb-mirror` (`C0AS0SN3735`) with:
   - Header: `seed-{project_id}-{YYYY-MM-DD}`
   - Body: full Claude Canvas content as markdown
3. Apps Script trigger fires:
   - Reads the message
   - Creates a Google Doc in the configured mirror folder
   - Names it `seed-{project_id}-{YYYY-MM-DD}`
   - Pastes the canvas content
4. Apps Script posts a threaded reply to the original Slack message with the Doc URL
5. Claude (next session) can surface the mirror URL from the thread if needed

**PM setup (one-time):**

1. Get added to `#plb-mirror` (`C0AS0SN3735`)
2. Deploy the Apps Script (template lives at [TBD — add link when script is stable])
3. Configure the script with:
   - Target Drive folder ID
   - Slack webhook or bot token for thread replies
4. Paste the Apps Script URL into the Claude Canvas as `seed_mirror_script_url`
5. Set `seed_mirror_path: automated` in the Claude Canvas config

**What's tested:** Claude posting to `#plb-mirror` — works, standard message post.

**What's untested (as of 2026-04-16):** The Apps Script side — reading from `#plb-mirror`, creating the Doc, posting the URL back as a thread reply. This needs a live run on a real project before the automated path can be recommended without caveat.

---

## Deciding Between Paths

| Situation | Recommended path |
|---|---|
| Brand new project, PM just getting started | Manual |
| Google Workspace team, PM comfortable with Apps Script | Automated |
| Microsoft / SharePoint team | Manual (only option for now) |
| Mixed ecosystem team | Manual — or automated if Google is the primary mirror location |
| Very high session frequency (daily+) | Automated — manual becomes a chore |
| One-off or short-lived project | Manual |

The provisioner (`PROJECT_SETUP.md`) defaults to manual. At session end, Claude offers both options: "Want the manual export, or do you have the automated mirror set up?"

---

## What the Claude Canvas Stores

Both paths use the same config block in the Claude Canvas:

```
### Mirror Config
seed_mirror_path: [manual / automated]
seed_mirror_drive_folder_url: [URL of the mirror folder]
seed_mirror_script_url: [Apps Script URL — automated only, blank otherwise]
```

`seed_mirror_filename_prefix` (set during provisioning) is used by both paths for consistent naming.

---

## Upgrade Path

A project starts on manual. When the PM is ready:

1. Configure the Apps Script
2. Set `seed_mirror_path: automated` in the Claude Canvas
3. Fill in `seed_mirror_script_url`
4. Next session, Claude defaults to the automated path

Downgrade is just as easy — set `seed_mirror_path: manual` and Claude stops posting to `#plb-mirror`.

---

## Reference

- `#plb-mirror` channel: `C0AS0SN3735`
- Drive mirror folder (example): https://drive.google.com/drive/folders/1m0UDmpd_o44U4RG95VA6SfDomJ-B
- PLB Mirror Bot App ID: `A0AT74BAALQ` — pending `files:read` approval, but **not needed** for the automated path described here

---

*MIRROR_PATHS.md v1.1 | April 2026 | Josh Payne*
*Companion to PROJECT_SETUP.md v1.7 and Prime Project Canvas v1.4*
*Changelog v1.1: PLB → seed rebrand; `plb_mirror_*` fields renamed to `seed_mirror_*`; filename-prefix examples updated to `seed-...`; `#plb-mirror` Slack channel name kept as-is (live channel, separate rename).*
