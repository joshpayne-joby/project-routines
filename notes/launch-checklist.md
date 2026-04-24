# seed — Pre-launch Checklist

Tracks what needs to land before sharing this repo beyond the maintainer. Living doc — edit freely as items clear, shift, or decompose.

Origin: carved out of the 2026-04-22 gap audit + README draft session.

---

## Forward references from README to resolve

- [x] `contracts/` folder — cross-cutting schemas. **Lean scope locked 2026-04-23: ship the two load-bearing contracts only, skip ceremony.**
  - [x] `contracts/hub-canvas-registry.md` — KV bullet section every Hub emits. **Landed 2026-04-23 v1.0.** Highest-leverage contract (Control Tower + COLLABORATOR_SETUP both parse this).
  - [x] `contracts/my-tasks-routine-config.md` — machine-readable table schema. **Landed 2026-04-23 v1.0.** Routine's own memory; breaks tomorrow's briefing if schema drifts.
  - [~] `contracts/seed-registry.md` — skipped for launch. Three-column table is near-self-documenting. Revisit only if a second writer appears.
  - [x] `contracts/claude-canvas-config.md` — **Walked back skip 2026-04-23, launched v0.5.** Field directory (not format lock) covering Identity, Canvas Registry, Communication, Photo Breadcrumbs, Mirror, Session State, Digest, Code Integration groups. Surfaced one real drift: digest_* fields referenced in Prime canvas but missing from PROJECT_SETUP v1.7 creation template — flagged in the contract as a next-edit resolution.
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
- [ ] Append repo-link footer to Prime Project canvas (`F0AU9KARVQ8`)
- [ ] Append repo-link footer to seed Registry canvas (`F0AUJ8FV6JH`)
- [ ] Append repo-link footer to each production Hub (AMFG-TEMPER, AES-WINGFLIP, AES-527PCO, AES-ACEBRDG, AES-PLBSYS)

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

- `mirrors/PRIME_PROJECT_CANVAS.md:258` references `MIRROR_PATHS.md` (bare filename). The live canvas `F0AU9KARVQ8` almost certainly still has the same bare ref — fixing just the repo mirror would desync from the canvas. Fix both: edit the live canvas to `docs/MIRROR_PATHS.md`, re-snapshot.
- `README.md:89` role line ("Joby Aviation, Advanced Manufacturing") — Josh to confirm or refine (was flagged earlier but still pending).
- Public mirror (`joshpayne-joby/project-routines`) currently carries only the orphan CLAUDE.md + pointer README. When CLAUDE.md changes, re-push the orphan with the new CLAUDE.md so Routines read the latest. None of today's commits touched CLAUDE.md, so mirror is still current.

## Out-of-scope for launch (upstream decisions, document only)

- Routine creation API (browser-only — upstream Anthropic)
- Slack Canvas title slot API updates (UI-only — upstream Slack)
- Slack Canvas section-level replace bug (full-replace workaround — upstream Slack)

---

*Created 2026-04-22. Living doc.*
