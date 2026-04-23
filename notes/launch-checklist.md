# seed — Pre-launch Checklist

Tracks what needs to land before sharing this repo beyond the maintainer. Living doc — edit freely as items clear, shift, or decompose.

Origin: carved out of the 2026-04-22 gap audit + README draft session.

---

## Forward references from README to resolve

- [ ] `contracts/` folder — cross-cutting schemas
  - [ ] `contracts/seed-registry.md` — canvas `F0AUJ8FV6JH`, three-column table schema, who writes/reads
  - [ ] `contracts/hub-canvas-registry.md` — KV bullet section every Hub emits
  - [ ] `contracts/my-tasks-routine-config.md` — machine-readable table schema
  - [ ] `contracts/claude-canvas-config.md` — the fenced PLB/seed Configuration block schema
  - [ ] `contracts/prime-project-canvas.md` — pointer to `F0AU9KARVQ8` as behavior contract
- [ ] `CONTROL_TOWER.md` — versioned mirror of Control Tower Instructions canvas `F0AUGD2CC9J`

## Repo visibility and access

- [ ] Flip repo visibility from public → private (pre-launch content shouldn't be public)
- [ ] Decide: keep on personal account (`joshpayne-joby`) with manual collaborator adds, or transfer to Joby org for auto org-member access (requires org admin + IT — related to B3)
- [ ] If staying on personal: add each collaborator's GitHub account so their Routine can read CLAUDE.md

## Phase 1 — Repo consolidation

- [x] Decide `seed/` subfolder layout vs. flat top-level — **locked 2026-04-23: function-grouped.** Root holds README, CLAUDE.md, COLLABORATOR_SETUP.md, PROJECT_SETUP.md (audience-direct). Folders: `contracts/`, `mirrors/`, `docs/`, `prompts/`, `notes/`.
- [ ] Delete stale `~/Claude/seed/COLLABORATOR_SETUP.md` (v0.1 — repo has v0.4)
- [ ] Delete stale `~/Claude/seed/PRIME_PROJECT_CANVAS.md` (older than repo copy)
- [ ] PLB → seed rebrand pass across all files
  - [ ] Resolve `## PLB Configuration` vs. `seed config block` naming mismatch (CLAUDE.md vs. PROJECT_SETUP template)
  - [ ] Rename `plb_mirror_*` fields in the config block schema
- [ ] Move `~/Claude/seed/PROJECT_SETUP.md` (v1.6) into repo
- [ ] Decision: move `~/Claude/seed/MIRROR_PATHS.md` into repo marked retired, or omit
- [ ] Decision: move `~/Claude/seed/PROJECT_INSTRUCTIONS_WRAPPER.md` into repo or omit
- [ ] Promote `README.draft.md` → `README.md` (after contracts/ and CONTROL_TOWER.md land)
- [ ] Verify Josh's role title in README (currently "Joby Aviation, Advanced Manufacturing" — fill in exact title if needed)
- [ ] Confirm `.DS_Store` is gitignored

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

## Out-of-scope for launch (upstream decisions, document only)

- Routine creation API (browser-only — upstream Anthropic)
- Slack Canvas title slot API updates (UI-only — upstream Slack)
- Slack Canvas section-level replace bug (full-replace workaround — upstream Slack)

---

*Created 2026-04-22. Living doc.*
