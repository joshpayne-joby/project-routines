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

- [ ] Create "seed Overview" canvas in Slack — one screen, ~150 words + prominent repo link. Serves as the Slack-side entry point.
- [x] Append repo-link footer to Prime Project canvas (`F0AU9KARVQ8`) — 2026-04-23
- [x] Append repo-link footer to seed Registry canvas (`F0AUJ8FV6JH`) — 2026-04-23
- [x] Append repo-link footer to each production Hub (AMFG-TEMPER `F0ASSM6C23F`, AES-WINGFLIP `F0AUJHC031P`, AES-527PCO `F0AT7RWPUDQ`, AES-ACEBRDG `F0ATCJBTUQK`, AES-PLBSYS `F0ASLHJQ57X`) — 2026-04-23. Private-repo URLs (joby/project-routines); SSO-gated by design.

## Phase 2 — Validation (before external handoff)

- [ ] Live-test Control Tower v1.3 missing-project fallback (low stakes, Josh's account only)
- [ ] Smoke-test COLLABORATOR_SETUP.md v0.4 on a real new collaborator (coordinate with the candidate first)
- [ ] Smoke-test PROJECT_SETUP.md v1.6 on a real new project (first non-dry-run use)

## Phase 3 — External handoff

- [ ] Pick one external user (PM on an active project, not an insider)
- [ ] Hand them the repo URL, watch them self-serve, don't hover
- [ ] Iterate README based on their bounce points

---

## Known blockers (not checklist items — status trackers)

- **B3** — Org-level GitHub App install still pending. Collaborators can't merge PRs against this repo; Josh commits on their behalf. Unblocks when IT approves.

## Minor follow-ups surfaced during Phase 1 close (2026-04-23)

- [x] `mirrors/PRIME_PROJECT_CANVAS.md:258` bare `MIRROR_PATHS.md` ref — **resolved 2026-04-23.** Live canvas `F0AU9KARVQ8` updated to `docs/MIRROR_PATHS.md` as part of the Prime v1.5 write-back; mirror re-snapshotted to match.
- [x] **Path A token sweep — seed rebrand across 4 live Claude Canvases + Prime (2026-04-23).** 5 token renames applied per Claude Canvas (`## PLB Configuration` → `## seed Configuration`; `plb_mirror_*` → `seed_mirror_*`; `PLB state file` → `seed state file`) across WINGFLIP (F0ARXHUEA74), 527PCO (F0AT4D7CLN9), ACEBRDG (F0ATK00P44S), TEMPER (F0ASZKZ31PE). Prime canvas (F0AU9KARVQ8) bumped to v1.5 with token renames + `MIRROR_PATHS.md` → `docs/MIRROR_PATHS.md`. Second pass same day cleaned version-line signature glitch (unclosed italic, `<@UNDBYGY1J>` → prose `Josh Payne`, restored "seed" prefix) and synced repo-link footer to prose. All 6 full-replace writes succeeded on first try. TEMPER's `![](slack_date:YYYY-MM-DD)` tokens preserved verbatim; pre-existing TEMPER corruption (duplicate tables) left in place per token-only scope. **Slack title-slot note:** Prime canvas title still reads `Prime Project | Active Session Behavior` with a double-space where a seedling emoji used to be — Slack strips emoji shortcodes from title slots at creation and the API cannot update titles (known platform limit, per `feedback_slack_canvas_title.md`). Body h1 is clean (`# :seedling: Prime Project | Active Session Behavior`). Title-slot fix requires Slack UI rename if ever desired — not blocking.
- `README.md:89` role line ("Joby Aviation, Advanced Manufacturing") — Josh to confirm or refine (was flagged earlier but still pending).
- Public mirror (`joshpayne-joby/project-routines`) currently carries only the orphan CLAUDE.md + pointer README. When CLAUDE.md changes, re-push the orphan with the new CLAUDE.md so Routines read the latest. None of today's commits touched CLAUDE.md, so mirror is still current.

## Out-of-scope for launch (upstream decisions, document only)

- Routine creation API (browser-only — upstream Anthropic)
- Slack Canvas title slot API updates (UI-only — upstream Slack)
- Slack Canvas section-level replace bug (full-replace workaround — upstream Slack)

---

*Created 2026-04-22. Living doc.*
