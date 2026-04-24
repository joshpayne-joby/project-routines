# Hub Canvas Registry — Contract

Every Project Hub emits a `## Canvas Registry` section. It is a machine-readable key-value list that exposes the five IDs needed to reach the rest of the project.

This contract exists because the registry is parsed by consumers with narrow tolerance — drift in label text, bullet format, or ID wrapping causes silent parse failures.

---

## Canonical format

```
## Canvas Registry

> Machine-readable canvas index. The seed Routine and Control Tower read this to find the project's canvases.

- **Project ID:** `[project_id]`
- **Display Name:** [Display Name]
- **Claude Canvas:** `[tasks_canvas_id]` — [Open]([tasks_canvas_url])
- **Human Canvas:** `[human_canvas_id]` — [Open]([human_canvas_url])
- **Channel:** `[project_channel_id or "none"]` — ![](#[project_channel_id])
- **Claude Project:** —
```

One bullet per field. Field label is bolded, followed by ` ` (colon-space). IDs are backtick-wrapped. Open-link and channel-mention suffixes are optional to humans but required by the format — keep them.

---

## Field spec

|Field|Type|Required|Notes|
|---|---|---|---|
|Project ID|string, backticked|Yes|Matches `[DOMAIN]-[SHORTNAME]` (e.g. `AES-WINGFLIP`). Unique across all seed projects.|
|Display Name|plain text|Yes|Human-readable. Matches the Claude Project name.|
|Claude Canvas|Slack canvas ID, backticked|Yes|`F0...` format. Followed by ` — [Open](url)`.|
|Human Canvas|Slack canvas ID, backticked|Yes|`F0...` format. Followed by ` — [Open](url)`.|
|Channel|Slack channel ID, backticked, OR `"none"`|Yes|`C0...` format or literal `none` (no backticks around `none`). Followed by ` — ![](#channel_id)` when a real ID.|
|Claude Project|URL or `—`|Optional|Placeholder `—` when no Claude Project exists yet. URL when set.|

The Hub's own canvas ID (Hub Canvas ID) is **not** listed in the registry — consumers use the canvas they just fetched.

---

## Producers

|File|When|
|---|---|
|[`PROJECT_SETUP.md`](../PROJECT_SETUP.md)|At new-project creation — Hub template includes the Canvas Registry section populated with real IDs on first write.|
|Retrofit (manual)|When bringing a pre-existing Hub into the system. Paste the canonical format at the top of the Hub below the title, fill the five fields.|

---

## Consumers

|File|What it parses|Failure mode if format drifts|
|---|---|---|
|[`mirrors/CONTROL_TOWER.md`](../mirrors/CONTROL_TOWER.md) missing-project fallback|All five IDs|Cannot harvest IDs → tells the owner "Hub hasn't been retrofitted" and briefs from the Claude Canvas directly, skipping the My Tasks write.|
|[`COLLABORATOR_SETUP.md`](../COLLABORATOR_SETUP.md) project pre-registration|All five IDs|Cannot pre-register the project → tells the PM "Hub needs to be retrofitted" and keeps going with a single Project ID + Hub URL pair.|

Both consumers degrade gracefully on parse failure. The contract still matters: silent degradation hides misconfigured Hubs, and the owner only finds out when the project "drops out" of tomorrow's briefing.

---

## Change safety

**Non-breaking (additive):**
- Adding a new optional bullet *below* the existing six. Consumers ignore unknown bullets.
- Changing the Claude Project value from `—` to a real URL.
- Changing the Channel value from `none` to a real channel ID (and vice versa).

**Breaking:**
- Renaming a label (`Claude Canvas:` → `Task Board:`) — consumers match on label text.
- Reordering bullets — some consumers scan in order.
- Removing the backticks around an ID — consumer regex expects backtick boundaries.
- Converting the KV list to a table — parsers are bullet-based.

Any breaking change must land as a coordinated patch in the producer AND every consumer listed above.

---

## Live example — AES-WINGFLIP

```
## Canvas Registry

> Machine-readable canvas index. The seed Routine and Control Tower read this to find the project's canvases.

- **Project ID:** `AES-WINGFLIP`
- **Display Name:** Wing-flip gantry
- **Claude Canvas:** `F0ARXHUEA74` — [Open](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0ARXHUEA74)
- **Human Canvas:** `F0AUJH9UT8R` — [Open](https://jobyaviation.enterprise.slack.com/docs/T046X1H57/F0AUJH9UT8R)
- **Channel:** `C0ATZAHBLHZ` — ![](#C0ATZAHBLHZ)
- **Claude Project:** —
```

---

*contracts/hub-canvas-registry.md v1.0 | April 2026 | Josh Payne*
*Producer of record: PROJECT_SETUP.md v1.7 (`## Canvas Registry` section, lines 411–420)*
