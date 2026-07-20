# Roadmap — dispatch

**Last updated:** 2026-07-20

dispatch is a documentation-only Claude Code plugin (Maintenance status): six multi-agent
analysis skills authored as `skills/*/SKILL.md`. There is no runtime code. Roadmap items are
therefore about skill correctness, consistency, and authoring workflow — not shipped features
in a compiled sense. Relevance filter (Maintenance-status): landscape, TDD, and
frontend-design skills are intentionally out of scope.

## Phase 1: Skill build-out
<!-- status: complete -->

**Built:** Six skills shipped and stabilized (v2.0 → v2.2.3): skeptic (8 domain critics +
plan/fix modes), recon, tribunal, cartograph, landscape, migrate. Recon integrated into the
skeptic flow; roadmap-first routing added to skeptic Future Work and recon gap persistence;
Concept & Strategy agent added to skeptic plan mode.

## Phase 2: Maintenance & consistency
<!-- status: active -->

### Milestone 2.1: Consistency hardening
<!-- status: active -->
<!-- target: 2.2.x -->

- [x] Reconstruct CHANGELOG from plugin.json history; tighten Release Checklist (32407bc)
- [x] Exhaustive consistency audits (2026-07-18, 2026-07-19)
- [x] Fix `/skeptic:recon` → `/dispatch:recon` namespace (dd9838c)
- [x] Scope Design Doc Persistence to 5 skills; document recon's distinct mechanism (82f801b)
- [ ] Review branch `scorched/dispatch-dogfood-verification-harness` (56 files, dogfood
      reports + fixture repos) — inspect, then merge or discard. Scan for kerd/kivna/PullMD
      refs before any push (public-repo divergence rule). *(owner-gated; tracked in TODO Backlog)*

### Milestone 2.2: Authoring workflow adoption
<!-- status: upcoming -->
<!-- target: 2.3.0 -->

- [ ] Adopt `/superpowers:writing-skills` as the front-door for future `skills/*/SKILL.md`
      edits (accepted from lorg 2026-07-20) — replaces ad-hoc edit + after-the-fact audit
- [ ] Run `/kerd:tend` to adjudicate switch-in-flagged missing structure (`docs/archive/`,
      `docs/playbook.md`, `kivna/vault.json`, `kivna/sessions/`, `.gitignore` entries) —
      decide per item whether a public docs-only plugin adopts or intentionally omits it
      (accepted from lorg 2026-07-20)

## Phase 3: Skill enhancements
<!-- status: upcoming -->

Deferred until a concrete new specialist, mode, or skill is proposed. Skill scheduling at
this phase's start: `/kerd:lorg` (reassess tooling, ~21-day cadence — last run 2026-07-20),
`/dispatch:skeptic` self-review post any minor bump, `/toolkit:switch-optimize` after any
CLAUDE.md/workflow change, `/kerd:trim` after Milestone 2.1 closes.
