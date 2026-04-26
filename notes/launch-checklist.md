# seed — Pre-launch Checklist

Tracks what needs to land before sharing this repo beyond the maintainer. Living doc — edit freely as items clear, shift, or decompose.

Origin: carved out of the 2026-04-22 gap audit + README draft session.

---

## Forward references from README to resolve

- [x] `contracts/` folder — cross-cutting schemas. **Lean scope locked 2026-04-23: ship the two load-bearing contracts only, skip ceremony.**
  - [x] `contracts/hub-canvas-registry.md` — KV bullet section every Hub emits. **Landed 2026-04-23 v1.0.** Highest-leverage contract (Control Tower + COLLABORATOR_SETUP both parse this).
  - [x] `contracts/my-tasks-routine-config.md` — machine-readable table schema. **Landed 2026-04-23 v1.0.** Routine's own memory; breaks tomorrow's briefing if schema drifts.
  - [~] `contracts/seed-registry.md` — skipped for launch. Three-column table is near-self-documenting. Revisit only if a second writer appears.
  - [x] `contracts/claude-canvas-config.md` — **Walked back skip 2026-04-23, launched v0.5; tightened to v0.6 same day.** Field directory (not format lock) covering Identity, Canvas Registry, Communication, Photo Breadcrumbs, Mirror, Session State, Digest, Code Integration groups. Surfaced one real drift: digest_* fields referenced in Prime canvas but missing from PROJECT_SETUP v1.7 creation template — flagged in the contract as a next-edit resolution. v0.6 walked back an unearned "load-bearing fence" claim after live-canvas recon showed the fence wasn't preventing any failure mode; dropped `seed_mirror_path` from required fields.
  - [~] `contracts/prime-project-canvas.md` — skipped for launch. Prime canvas IS the behavior contract; mirrors/PRIME_PROJECT_CANVAS.md already serves as the versioned pointer.
- [x] `CONTROL_TOWER.md` — versioned mirror of Control Tower Instructions canvas `F0AUGD2CC9J` — **landed 2026-04-23 at `mirrors/CONTROL_TOWER.md` (v1.3).**

## Repo visibility and access

- [x] **Resolved 2026-04-23 via two-repo split:**
  - **Canonical:** `joby/project-routines` (private, org-gated). All substantive work lives here.
  - **Public mirror:** `joshpayne-joby/project-routines` (public, orphan history, machine-readable files only). Needed because Claude Code Routines can't SSO into org-private repos — mirror is the runtime read surface for `CLAUDE.md`.
  - Force-pushed fresh orphan to public mirror; force-pushed local fat history to joby canonical. Local tracks `joby/main`.
- [ ] Per-collaborator access to the public mirror is open-read — no add step. Private repo collaboration still pending GitHub App org install (B3).

## Phase 1 — Repo consolidation

- [x] Decide `seed/` subfolder layout vs. flat top-level — **locked 2026-04-23: function-grouped.** Root holds README, CLAUDE.md, COLLABORATOR_SETUP.md, PROJECT_SETUP.md (audience-direct). Folders: `contracts/`, `mirrors/`, `docs/`, `prompts/`, `notes/`.
- [x] Delete stale `~/Claude/seed/COLLABORATOR_SETUP.md` (v0.1 — repo has v0.4)
- [x] Delete stale `~/Claude/seed/PRIME_PROJECT_CANVAS.md` (older than repo copy)
- [x] Delete stale `~/Claude/seed/{PROJECT_SETUP,MIRROR_PATHS,PROJECT_INSTRUCTIONS_WRAPPER}.md` — all superseded by repo versions. Backup at `/tmp/seed-stale-backup-20260423.tar.gz`.
- [x] PLB → seed rebrand pass across all files
  - [x] `## PLB Configuration` → `## seed Configuration` (PROJECT_SETUP.md v1.7)
  - [x] Rename `plb_mirror_*` → `seed_mirror_*` (PRIME_PROJECT_CANVAS.md, PROJECT_SETUP.md, MIRROR_PATHS.md)
- [x] Move `~/Claude/seed/PROJECT_SETUP.md` (v1.6) → repo (landed as v1.7 with rebrand, commit `4e38766`)
- [x] `~/Claude/seed/MIRROR_PATHS.md` → repo `docs/` with rebrand (v1.1, part of commit `5b18fc4`)
- [x] `~/Claude/seed/PROJECT_INSTRUCTIONS_WRAPPER.md` → repo `docs/` with rebrand (v1.1, part of commit `5b18fc4`)
- [x] Promote `README.draft.md` → `README.md` (commit `a750fe7`, 2026-04-23)
- [ ] Verify Josh's role title in README (currently "Joby Aviation, Advanced Manufacturing" — fill in exact title if needed)
- [x] Confirm `.DS_Store` is gitignored (commit `a9dffa5`)

## Phase 1 — Canvas → repo discovery path

- [x] Create "seed Overview" canvas in Slack — **landed 2026-04-24 as `F0AV3E41QJ2`.** ~155 words, four sections (what it is / where everything lives / live in this workspace / maintainer). Title-slot emoji stripped per Slack limit (body h1 clean).
- [x] Append repo-link footer to Prime Project canvas (`F0AU9KARVQ8`) — 2026-04-23
- [x] Append repo-link footer to seed Registry canvas (`F0AUJ8FV6JH`) — 2026-04-23
- [x] Append repo-link footer to each production Hub (AMFG-TEMPER `F0ASSM6C23F`, AES-WINGFLIP `F0AUJHC031P`, AES-527PCO `F0AT7RWPUDQ`, AES-ACEBRDG `F0ATCJBTUQK`, AES-PLBSYS `F0ASLHJQ57X`) — 2026-04-23. Private-repo URLs (joby/project-routines); SSO-gated by design.

## Phase 2 — Validation (before external handoff)

- [~] Live-test Control Tower v1.3 missing-project fallback (low stakes, Josh's account only) — **partial pass 2026-04-24.** Asked CT (`F0AUGD2CC9J`-deployed) `What's the status on AES-527PCO?`. Round 1: fuzzy-match short-circuit — CT skipped Step 1, suggested AMFG-527OVEN from Routine Config without consulting seed Registry. Round 2 (after push-back): CT read seed Registry, resolved AES-527PCO → Hub `F0AT7RWPUDQ`, recognized Hub already represented in My Tasks under `AMFG-527OVEN`, correctly refused duplicate write. Dedup behavior tested cleanly via Hub-overlap. Write path untested by design (Registry has no projects genuinely missing from My Tasks). Side finding: AES-/AMFG- prefix drift between seed Registry and Routine Config is REAL for both `527PCO` ↔ `527OVEN` and `ACEBRDG` ↔ `PANELBOM` (same Hub canvases under different IDs) — deferred per Josh "let it ride until rewrite." v1.4 patch queued (see below).
- [~] Smoke-test COLLABORATOR_SETUP.md v0.4 on a real new collaborator — **handed to Adam Hickok 2026-04-24** via fresh onboarding canvas `F0AV0JM98HH` + seed Overview `F0AV3E41QJ2`. Pre-handoff polish landed in commit `640acd2` (slug rule, schedule, Registry phrasing, GitHub-down fallback wording). Old PLB-branded onboarding canvas `F0AU8HUDH1S` deleted by Josh as part of cleanup. Awaiting Adam's run-through.
- [x] Smoke-test PROJECT_SETUP.md on a real new project — **two iterations 2026-04-24.** v1.9 surfaced ghost-table on Hub post-creation patch (provisioner improvised section-replace, Slack's section-replace bug duplicated the table). v1.10 fixed by codifying Canvas Write Rules + explicit Hub-patch sub-step (full-canvas replace, no section_id). Re-smoke clean. All 6 throwaway canvases + 2 Claude Projects cleaned up by Josh.

## Phase 3 — Handoff to non-insider Joby PMs

**Scope clarified 2026-04-24:** Josh isn't pursuing non-Joby adoption for the foreseeable future. "External handoff" here means a Joby PM who isn't a seed maintainer — someone who consumes the system rather than building it. Adam's run (Phase 2) is partially fulfilling this role.

- [~] Adam Hickok onboarding 2026-04-24 — first non-insider Joby PM to run COLLABORATOR_SETUP v0.4 cold. Treat as the canonical "non-insider self-serve" smoke once it completes.
- [ ] Iterate README + COLLABORATOR_SETUP based on Adam's bounce points (where he had to ask, where he got lost, where the doc was clearer than expected)
- [ ] (Deferred indefinitely) Non-Joby external handoff. Public-mirror file expansion to support BOOTSTRAP/PROJECT_SETUP self-serve from outside the workspace is the gating dep — no plan to do this yet.

---

## Known blockers (not checklist items — status trackers)

- **B3** — Org-level GitHub App install still pending. Collaborators can't merge PRs against this repo; Josh commits on their behalf. Unblocks when IT approves.

## Minor follow-ups surfaced during Phase 1 close (2026-04-23)

- [x] `mirrors/PRIME_PROJECT_CANVAS.md:258` bare `MIRROR_PATHS.md` ref — **resolved 2026-04-23.** Live canvas `F0AU9KARVQ8` updated to `docs/MIRROR_PATHS.md` as part of the Prime v1.5 write-back; mirror re-snapshotted to match.
- [x] **Path A token sweep — seed rebrand across 4 live Claude Canvases + Prime (2026-04-23).** 5 token renames applied per Claude Canvas (`## PLB Configuration` → `## seed Configuration`; `plb_mirror_*` → `seed_mirror_*`; `PLB state file` → `seed state file`) across WINGFLIP (F0ARXHUEA74), 527PCO (F0AT4D7CLN9), ACEBRDG (F0ATK00P44S), TEMPER (F0ASZKZ31PE). Prime canvas (F0AU9KARVQ8) bumped to v1.5 with token renames + `MIRROR_PATHS.md` → `docs/MIRROR_PATHS.md`. Second pass same day cleaned version-line signature glitch (unclosed italic, `<@UNDBYGY1J>` → prose `Josh Payne`, restored "seed" prefix) and synced repo-link footer to prose. All 6 full-replace writes succeeded on first try. TEMPER's `![](slack_date:YYYY-MM-DD)` tokens preserved verbatim; pre-existing TEMPER corruption (duplicate tables) left in place per token-only scope. **Slack title-slot note:** Prime canvas title still reads `Prime Project | Active Session Behavior` with a double-space where a seedling emoji used to be — Slack strips emoji shortcodes from title slots at creation and the API cannot update titles (known platform limit, per `feedback_slack_canvas_title.md`). Body h1 is clean (`# :seedling: Prime Project | Active Session Behavior`). Title-slot fix requires Slack UI rename if ever desired — not blocking.
- `README.md:89` role line ("Joby Aviation, Advanced Manufacturing") — Josh to confirm or refine (was flagged earlier but still pending).
- Public mirror (`joshpayne-joby/project-routines`) currently carries only the orphan CLAUDE.md + pointer README. When CLAUDE.md changes, re-push the orphan with the new CLAUDE.md so Routines read the latest. None of today's commits touched CLAUDE.md, so mirror is still current.

## seed Changelog architecture (Stage 1 done 2026-04-26)

**Distribution layer for framework updates.** Reviewed via Control Tower + Project Instructions chats; both converged on a four-tier model. Stage 1 builds the source-of-truth canvas; Stages 2–3 wire it into daily flow.

- [x] **Stage 1 — seed Changelog canvas** (`F0AVAB5Q4KY`, 2026-04-26). Format `YYYY-MM-DD — [Component] [Version] — [Title] (SEMVER)`. Seeded with two real entries (CLAUDE.md v2.1 PATCH, Control Tower v1.4 MINOR). Linked from seed Overview canvas. Single source of truth — no parallel records.
- [x] **Stage 2 — "What's New in seed" My Tasks emit** (CLAUDE.md v2.2, commit `6105809`, 2026-04-26). Three additions: new "Read the seed Changelog" parsing section, new "Output format — What's New in seed" rendering section, write order updated (item #1, hidden if empty). Italic scope-clarifying subhead distinguishes from "What Changed Since Last Run." Payload cost ~5–10 lines per run when entries exist; offset by compact-quiet-project skill in same-day release.

## compact-quiet-project skill (CLAUDE.md v2.3, MINOR, 2026-04-26)

**First Claude Code Skill in the framework.** Lives at `.claude/skills/compact-quiet-project/SKILL.md`. Reduces briefing payload by collapsing quiet projects into one-line summaries grouped under a "Quiet Projects (N)" section.

- Commit: `c3aac16`
- Quietness rule: stale 7+ days AND zero `:red_circle:` tasks AND not in "Waiting On You" — all three must be true.
- Audit at release (Josh's 14-project My Tasks): 4 qualify — Grieve Oven Exhaust Hood (10d), Skinners Onboarding (10d), R&D Machine Shop Portal (11d), Automated Circular Saw (17d). Estimated ~24 lines saved.
- Disqualifiers seen in audit: 7 projects not stale (recent activity), 3 projects stale-but-red-flagged (Temper / 527 Oven / Pickle), 0 in Waiting On You (section is currently empty).
- Net payload effect after v2.2 + v2.3: roughly **-15 to -25 lines**, addresses the stream idle timeout pattern Josh has been hitting.
- Side finding: scorecard header in My Tasks claims "5 stale" but actual count under the 7-day rule is 7 — pre-existing data drift, surfaced per `feedback_raise_mismatches_up.md`. Not blocking the release; will resolve on next Routine run when the scorecard regenerates.

**First seed framework skill — pattern established.** Future skills follow the same path: `.claude/skills/<skill-name>/SKILL.md` in canonical, mirrored to public, referenced from CLAUDE.md by name + path.
- [ ] **Stage 3 — `#plb-mirror` → `#seed-updates` rename + pin Changelog canvas.** PATCH stays silent in channel (changelog only); MINOR gets one-liner; MAJOR gets full notice + DM. ~5 min when ready.
- [ ] **Stage 4 — migration notes on MAJOR only.** No work until first MAJOR release.

**Decisions resolved during review:**
- Component-scoped versioning, not single rolling seed-framework version (components evolve at different rates).
- "What's New" placement: My Tasks (cross-project ambient layer), not Hub canvases (per-project context).
- Window: "last 3 entries since last successful Routine run." No arbitrary calendar window. Hides if empty.
- MINOR is NOT silent — gets a Slack post. Both reviewers pushed back hard on the original "MINOR silent" proposal.
- Channel: rename `#plb-mirror` (already earmarked for retirement) rather than create `#seed-updates` from scratch.

## Minor follow-ups surfaced 2026-04-24

- seed Registry canvas instance renamed to "Advanced Manufacturing seed Registry" (body h1 + title slot, `F0AUJ8FV6JH`). Concept name "seed Registry" unchanged in contracts/provisioners — distinguishing the workspace instance from the pattern.
- PROJECT_SETUP.md bumped v1.7 → v1.11 over the course of the day. v1.8 fixed header/footer version mismatch. v1.9 aligned the close step with the wrapper (replaced stale fat-paste path). v1.10 added a "Canvas Write Rules" section + explicit Hub-patch sub-step after a v1.9 smoke surfaced the ghost-table bug on Hub post-creation patch. v1.11 added Rule 5 (title parameter only — never duplicate in body h1) and Rule 6 (full-replace ghosts an h1 *change* but cleans an h1 *removal*) — both surfaced live during the day's canvas work and codified into the spec the same day.
- COLLABORATOR_SETUP v0.4 sanity-read audit landed pre-handoff polish edits (commit `640acd2`): explicit slug rule (`firstname-lastname.md`), schedule reframing, "workspace seed Registry" phrasing, GitHub-down fallback rewrite.
- Public mirror state confirmed current 2026-04-24: CLAUDE.md SHA `b390f388...` matches local. No drift since 2026-04-23 force-push.
- Canvas write nuances learned 2026-04-24 — **codified into PROJECT_SETUP v1.11 Canvas Write Rules (commit `1f250d3`):**
  - Rule 5 — Creating a canvas with both the `title` parameter AND a body h1 always renders 2× (Slack stores them independently — known quirk per `feedback_slack_canvas_title.md`). Generalized from COLLABORATOR_SETUP's My Tasks warning to all canvas creation.
  - Rule 6 — Full-replace ghosts the h1 when the h1 *changes* (observed renaming the Registry — Josh did Slack UI cleanup); cleans cleanly when the body h1 is *removed* (observed fixing Adam's onboarding canvas). Asymmetry now in the spec.

## CT v1.4 patch — applied 2026-04-24

Landed both changes to the Fallback section. Canonical `mirrors/CONTROL_TOWER.md` updated; canvas `F0AUGD2CC9J` full-replaced clean (body h1 stripped, version lives in footer only — eliminates h1-change ghost risk on future bumps). Per Adam-canvas asymmetry: h1 *removal* cleans, h1 *change* ghosts. Stripping the body h1 was the right play.

- **Anti-fuzzy-match guard** (Fallback intro): *"Do not ask for a Hub link, **and do not offer a similar-looking ID from My Tasks as an alternative**, until you've consulted the seed Registry."*
- **Raise-mismatches-up rule** (new paragraph in Fallback intro): *"If the seed Registry and My Tasks disagree on the Project ID for the same Hub, if a referenced ID resolves to a Hub already represented under a different ID, or any other inconsistency surfaces during the lookup — surface it explicitly to the owner before continuing. Don't silently dedup, rename, or work around it. The mismatch is signal that wants a decision."*

**Outstanding for Josh:** re-paste the v1.4 instructions into his deployed CT Claude Project Instructions (https://claude.ai/project/019db149-8d23-7093-895c-05183cd4e36d). Canvas-only update doesn't auto-propagate to live Project Instructions.
- Adam's old onboarding canvas `F0AU8HUDH1S` deleted by Josh 2026-04-24 (PLB-branded, fork-required, corrupted with duplicate sections from prior section-replace bugs). Replaced by `F0AV0JM98HH`.

## Out-of-scope for launch (upstream decisions, document only)

- Routine creation API (browser-only — upstream Anthropic)
- Slack Canvas title slot API updates (UI-only — upstream Slack)
- Slack Canvas section-level replace bug (full-replace workaround — upstream Slack)

---

*Created 2026-04-22. Living doc.*
