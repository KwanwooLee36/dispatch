# Changelog

Reconstructed from `git log -p -- .claude-plugin/plugin.json` against full commit history. A version band groups every commit that shipped between one `plugin.json` version bump and the next; the bump commit itself is the last entry in its own band. This is documentation only — no historical version number is changed.

## Audit note

Two feat-level commits shipped user-facing skill behavior with no accompanying version bump, contradicting the Release Checklist in `CLAUDE.md` ("Bump version in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`"):

- **89f58c8** — "feat: add Testing specialist agent to skeptic skill" (2026-05-17). Added an 8th specialist agent and a `plan test` subcommand to `skills/skeptic/SKILL.md`, updated agent counts in `README.md`. No version change. The bump to 2.1.0 didn't land until the next commit, `a249a40`, three days of unbumped shipped history later (license manifest c7fdb0b + this commit).
- **dba5611** — "feat: add roadmap-first routing to skeptic Future Work and recon gap persistence" (2026-05-17). Added new routing behavior to `skills/skeptic/SKILL.md` and `skills/recon/SKILL.md`. No version change. The bump to 2.2.0 didn't land until the following commit, `ddeb2d0`.

Both are new agent/routing surfaces on existing skills — squarely "user-facing behavior changed" under the checklist's intent, even though the checklist's wording didn't make that unambiguous at the time (see CLAUDE.md fix in this same change).

## [2.3.1] - 2026-08-03

- `f5d3fc4` (2026-07-21) — docs: relocate audit reports, adopt writing-skills convention, tend structure pass.
- `fa0dd11` (2026-07-21) — docs: generalize internal-tooling references in audit reports.
- `b973a8d` (2026-07-21) — merge: 2026-07-21 audit batch — 17 behavior findings fixed, F2/F4-F12 closed, docs consolidated (2.3.0).
- `00a1741` (2026-08-03) — docs(skills): scale the injected Design Docs block to INDEX.md-only. **Bumped version 2.3.0 → 2.3.1.**

## [2.3.0] - 2026-07-21

- `1cda7e0` (2026-07-21) — feat: close all 17 SKILL.md behavior findings (B1-B17). **Bumped version 2.2.4 → 2.3.0.**

## [2.2.4] - 2026-07-21

- `82f801b` (2026-07-20) — docs: scope Design Doc Persistence to 5 skills, document recon's distinct mechanism.
- `1e1a06c` (2026-07-20) — docs: add roadmap + lorg report from maintenance sweep.
- `16eb09d` (2026-07-21) — docs: add skill-behavior verification audit + dogfood branch recommendation.
- `c7156b7` (2026-07-21) — docs: add competitive gap research vs public CC plugin marketplaces.
- `af9fa35` (2026-07-21) — fix: close consistency findings F2, F4-F12 across all six skills. **Bumped version 2.2.3 → 2.2.4.**

## [2.2.3] - 2026-07-20

- `dd9838c` (2026-07-20) — fix: correct /skeptic:recon to /dispatch:recon namespace. **Bumped version 2.2.2 → 2.2.3.**

## [2.2.2] - 2026-07-19

- `32407bc` (2026-07-18) — docs: add CHANGELOG.md reconstructing version history, tighten release checklist.
- `0476e52` (2026-07-18) — docs: add exhaustive consistency audit report (2026-07-18).
- `0f15cfb` (2026-07-19) — docs: add exhaustive consistency audit report (2026-07-19). **Bumped version 2.2.1 → 2.2.2.**

## [2.2.1] - 2026-07-18

- `dc0b13b` (2026-07-19) — chore(gitignore): ignore .scorched/ (COA link, Work s139).
- `9158c10` (2026-07-18) — chore: gitignore .scorched/ scratch state, matching .skeptic/ pattern.
- `609f192` (2026-07-18) — docs: deduplicate quick-mode warning in migrate skill. **Bumped version 2.2.0 → 2.2.1.**

## [2.2.0] - 2026-05-18

- `dba5611` (2026-05-17) — feat: add roadmap-first routing to skeptic Future Work and recon gap persistence. **Shipped without a version bump.**
- `ddeb2d0` (2026-05-18) — feat: add Concept & Strategy agent to skeptic plan mode. Added `plan concept` subcommand (Opus agent) reading concept/strategy findings plus recon/landscape reports; added Concept to the overarching plan fan-out (7→8 category agents); removed misleading console footer lines. **Bumped version 2.1.0 → 2.2.0.**

## [2.1.0] - 2026-05-17

- `c7fdb0b` (2026-05-14) — feat: add license tracking manifest (`licenses.json`).
- `89f58c8` (2026-05-17) — feat: add Testing specialist agent to skeptic skill. **Shipped without a version bump.**
- `a249a40` (2026-05-17) — feat: integrate recon into skeptic flow + competitive intelligence in plan. Added Step 3.5 (post-concept-agent recon prompt), inline competitor research fan-out, recon-derived synthesis findings, and opportunistic reads of `.skeptic/recon-*` / `.landscape/` reports by the plan command. **Bumped version 2.0.0 → 2.1.0.**

## [2.0.0] - 2026-05-13

- `57dd7e2` — Initial commit: dispatch v2.0.0, multi-agent analysis toolkit for Claude Code.
