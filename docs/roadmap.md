# Roadmap — dispatch

**Last updated:** 2026-07-21

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
<!-- target: 2.3.x -->

- [x] Reconstruct CHANGELOG from plugin.json history; tighten Release Checklist (32407bc)
- [x] Exhaustive consistency audits (2026-07-18, 2026-07-19)
- [x] Skill-behavior verification audit (2026-07-21) — traced every invocation flow, file-I/O
      path, model/roster claim, tool grant, and example command across all 6 SKILL.md;
      17 new findings (B1–B17) in `docs/audits/skill-behavior-verification-2026-07-21.md`
- [x] Fix `/skeptic:recon` → `/dispatch:recon` namespace (dd9838c)
- [x] Scope Design Doc Persistence to 5 skills; document recon's distinct mechanism (82f801b)
- [x] Review branch `scorched/dispatch-dogfood-verification-harness` (56 files, dogfood
      reports + fixture repos) — inspected 2026-07-21: self-test verified (6/6 conformant
      PASS, 6/6 drift detected), no kerd/kivna/PullMD refs. **Recommendation: MERGE** after
      3 fixes. See `docs/audits/skill-behavior-verification-2026-07-21.md` §5.
- [ ] Execute the dogfood-harness merge (owner-gated): drop the `plugin.json` 2.2.1 bump
      (main is 2.2.3), relocate the two tracked `.scorched/dogfood-report-*.md` out of the
      git-ignored dir, amend CLAUDE.md's "documentation-only (no runtime code)" line for
      `tests/dogfood/`'s `python3` verifier
- [x] Fix the still-open consistency findings F2, F4–F12 (2026-07-21) — landscape `survey-*`
      glob, Testing agent model, skeptic write-scope claim, `concept` plan type, `/dispatch:`
      namespace made canonical repo-wide (verified against `plugin.json`/`marketplace.json`),
      stale pinned model IDs, unified recon/tribunal error wording, cartograph Domain
      supplement, tribunal "four jobs", duplicate step 7, empty recon Teardown; plus a
      cosmetic table/invocation-block normalization across all 6 SKILL.md. Bumped 2.2.3 → 2.2.4
- [x] Trigger-overlap audit (2026-07-21) — 6 description overlaps (O1–O6) with disambiguation
      recommendations in `docs/audits/trigger-overlap-audit-2026-07-21.md`; O6 recorded as a
      deliberate overlap, not to be "fixed"
- [ ] Apply the O1–O4 disambiguators to the six `description:` lines — user-facing change,
      needs its own version bump
- [x] Fix the new behavior findings (B1–B17) (2026-07-21) — all 17 closed, none deferred:
      landscape breadth-before-length + warn-not-reject (B1, B2), migrate ask-before-write (B3),
      tribunal free-text prompt replaces the impossible `AskUserQuestion` call (B4), skeptic
      inline recon now persists `.skeptic/recon-*.md` (B5), roadmap-first routing on all five
      persistence surfaces (B6), migrate read-only tool grant (B9), plus B7/B8/B10/B11/B13–B17.
      Release Checklist reconciled with the real `marketplace.json` shape (B12). Bumped 2.2.4 → 2.3.0

### Milestone 2.2: Authoring workflow adoption
<!-- status: upcoming -->
<!-- target: 2.4.0 -->

- [x] Adopt `/superpowers:writing-skills` as the front-door for future `skills/*/SKILL.md`
      edits (accepted from lorg 2026-07-20; recorded 2026-07-21) — replaces ad-hoc edit +
      after-the-fact audit. `CLAUDE.md` Conventions now names it, cross-referencing the three
      audits in `docs/audits/` as the motivating precedent
- [x] Run `/kerd:tend` to adjudicate switch-in-flagged missing structure (2026-07-21;
      accepted from lorg 2026-07-20). Found first: `TODO.md`, `docs/playbook.md`,
      `docs/archive/`, `kivna/`, `.slainte` are **already all gitignored** (pre-existing,
      not new this session) — dispatch already decided this Kerd scaffolding stays
      local-only, never tracked in the public repo. Per-item calls:
      - **ADOPT** `docs/archive/` — created empty locally; already gitignored so zero
        public-repo effect, cheap, matches Archive Policy convention
      - **OMIT** `docs/playbook.md` — redundant with what CLAUDE.md/README/roadmap already
        cover for a docs-only plugin; local-only file no session currently reads
      - **OMIT** `kivna/vault.json`, `kivna/sessions/` — kivna scaffolding, default-omit per
        guardrail; no vault folder exists for dispatch and creating one adds private-fleet
        linkage with no public benefit (kivna/ is fully gitignored regardless)
      - **OMIT** adding literal `kivna/input/`/`kivna/output/` `.gitignore` lines — already
        covered by the blanket `kivna/` ignore (line 2); would be redundant
      - Also found (out of the originally-flagged list, not acted on — see TODO backlog):
        `.loop/` and `.session-start-sha` are untracked and NOT gitignored (loop-orchestrator
        scratch, separate from Kerd's own scaffolding)
- [x] Competitive gap research vs. public Claude Code plugin marketplaces (2026-07-21) —
      surveyed wshobson/agents, the official marketplace, VoltAgent subagents, and the
      official plugin spec; 9 ranked recommendations in
      `docs/audits/competitive-gap-research-2026-07-21.md`
- [ ] Adopt progressive disclosure — split each skill into `SKILL.md` + `references/`
      (all 6 exceed the 8 KB ecosystem cap; skeptic is 60.8 KB). Rec. 1 of the gap research
- [x] Fix the release-checklist / manifest mismatch (2026-07-21, closed as B12) — `CLAUDE.md`
      step 1 now names `plugin.json` as the only version field and states that
      `marketplace.json` has none; new step 2 covers what marketplace.json *is* kept in sync on
  - [ ] Still open from Rec. 3 of the gap research: add `$schema` + `displayName` to `plugin.json`
- [ ] Backfill `CHANGELOG.md` — its newest band is `[2.2.0]`; versions 2.2.1, 2.2.2, 2.2.3,
      2.2.4 and 2.3.0 shipped unrecorded. Pairs with the release-checklist fix above
- [ ] Add minimal CI (`claude plugin validate` + link check) — complements, does not replace,
      the dogfood harness. Rec. 4 of the gap research
- [ ] Record the standing scope decline of multi-harness distribution (Codex/Cursor/Gemini)
      in `CLAUDE.md` so future lorg scans stop resurfacing it. §4F of the gap research

## Phase 3: Skill enhancements
<!-- status: upcoming -->

Candidates now on the table from the 2026-07-21 competitive gap research
(`docs/audits/competitive-gap-research-2026-07-21.md`), in its ranked order — each is an
owner decision, not a committed item:

- [ ] Adversarial-verification pass: a verifier agent that has not seen a specialist's
      reasoning must knock findings down against a rubric before synthesis scores them
      (rec. 2 — highest-value pattern import; consumes skeptic's existing canonical
      finding format)
- [ ] Haiku tier for mechanical passes (roadmap/TODO routing, console formatting, dedup
      arithmetic); consider `inherit` for user model choice. dispatch is 16 Opus / 7 Sonnet
      with no cheap tier (rec. 5)
- [ ] Diff/PR-scoped review mode (`/dispatch:skeptic diff` or `pr <n>`) reusing the existing
      8 critics with the diff as scope object — the one capability gap a user might switch
      plugins over (rec. 6)
- [ ] `CONTRIBUTING.md` + an authoring doc capturing the fan-out contract and the
      `Use when …` trigger convention (rec. 7) — should absorb the disambiguation rules from
      `docs/audits/trigger-overlap-audit-2026-07-21.md` so new skills do not re-collide
- [ ] Dynamic fan-out width — pick the skeptic roster from repo signals instead of always
      spending 8 critics. Resolve before adding any 9th/10th critic (rec. 8)
- [ ] New-skill candidates, none urgent: incident-response/postmortem fan-out, standalone
      test-strategy analysis, dependency/supply-chain audit, self-evaluation of dispatch's
      own agent prompts (most on-brand), C4 diagrams in cartograph (cheapest) (rec. 9)

Explicitly declined by that research: multi-harness distribution, breadth-matching large
marketplaces, accessibility/observability critics (until dynamic width lands), and any
runtime coordination program (breaks the documentation-only property).

Otherwise deferred until a concrete new specialist, mode, or skill is proposed. Skill scheduling at
this phase's start: `/kerd:lorg` (reassess tooling, ~21-day cadence — last run 2026-07-20),
`/dispatch:skeptic` self-review post any minor bump, `/toolkit:switch-optimize` after any
CLAUDE.md/workflow change, `/kerd:trim` after Milestone 2.1 closes.
