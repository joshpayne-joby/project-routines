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

- [ ] Live-test Control Tower v1.3 missing-project fallback (low stakes, Josh's account only)
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

## Minor follow-ups surfaced 2026-04-24

- seed Registry canvas instance renamed to "Advanced Manufacturing seed Registry" (body h1 + title slot, `F0AUJ8FV6JH`). Concept name "seed Registry" unchanged in contracts/provisioners — distinguishing the workspace instance from the pattern.
- PROJECT_SETUP.md bumped v1.7 → v1.10 over the course of the day. v1.8 fixed header/footer version mismatch. v1.9 aligned the close step with the wrapper (replaced stale fat-paste path). v1.10 added a "Canvas Write Rules" section + explicit Hub-patch sub-step after a v1.9 smoke surfaced the ghost-table bug on Hub post-creation patch.
- COLLABORATOR_SETUP v0.4 sanity-read audit landed pre-handoff polish edits (commit `640acd2`): explicit slug rule (`firstname-lastname.md`), schedule reframing, "workspace seed Registry" phrasing, GitHub-down fallback rewrite.
- Public mirror state confirmed current 2026-04-24: CLAUDE.md SHA `b390f388...` matches local. No drift since 2026-04-23 force-push.
- Canvas write nuances learned 2026-04-24 (not yet codified into PROJECT_SETUP's Canvas Write Rules — current provisioner flows don't hit them, but worth capturing):
  - Full-replace ghosts the h1 when the h1 *changes* (observed renaming the Registry — Josh did Slack UI cleanup); cleans cleanly when the body h1 is *removed* (observed fixing Adam's onboarding canvas).
  - Creating a canvas with both the `title` parameter AND a body h1 always renders 2× (Slack stores them independently — known quirk per `feedback_slack_canvas_title.md`). COLLABORATOR_SETUP warns about this for My Tasks; the warning generalizes to any canvas creation.
- Adam's old onboarding canvas `F0AU8HUDH1S` deleted by Josh 2026-04-24 (PLB-branded, fork-required, corrupted with duplicate sections from prior section-replace bugs). Replaced by `F0AV0JM98HH`.

## Out-of-scope for launch (upstream decisions, document only)

- Routine creation API (browser-only — upstream Anthropic)
- Slack Canvas title slot API updates (UI-only — upstream Slack)
- Slack Canvas section-level replace bug (full-replace workaround — upstream Slack)

---

*Created 2026-04-22. Living doc.*
