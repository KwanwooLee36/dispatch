# Changelog

## [2.3.2] - 2026-08-04

- feat: apply trigger-overlap disambiguators O1-O4 to skill descriptions — recon drops the bare `market research` trigger and gates on competitors already named in a skeptic report; landscape gets an explicit "use recon instead" redirect; tribunal gets decided-vs-undecided (vs migrate), named-options (vs landscape), and adopt-vs-compete-with (vs recon) clauses; migrate gets a decision-already-made clause pointing back to tribunal. See `docs/audits/trigger-overlap-audit-2026-07-21.md` O1-O4. **Bumped version 2.3.1 → 2.3.2.**

Reconstructed from `git log -p -- .claude-plugin/plugin.json` against full commit history. A version band groups every commit that shipped between one `plugin.json` version bump and the next; the bump commit itself is the last entry in its own band. This is documentation only — no historical version number is changed.

## Audit note

Two feat-level commits shipped user-facing skill behavior with no accompanying version bump, contradicting the Release Checklist in `CLAUDE.md` ("Bump version in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`"):

- **89f58c8** — "feat: add Testing specialist agent to skeptic skill" (2026-05-17). Added an 8th specialist agent and a `plan test` subcommand to `skills/skeptic/SKILL.md`, updated agent counts in `README.md`. No version change. The bump to 2.1.0 didn't land until the next commit, `a249a40`, three days of unbumped shipped history later (license manifest c7fdb0b + this commit).
- **dba5611** — "feat: add roadmap-first routing to skeptic Future Work and recon gap persistence" (2026-05-17). Added new routing behavior to `skills/skeptic/SKILL.md` and `skills/recon/SKILL.md`. No version change. The bump to 2.2.0 didn't land until the following commit, `ddeb2d0`.

Both are new agent/routing surfaces on existing skills — squarely "user-facing behavior changed" under the checklist's intent, even though the checklist's wording didn't make that unambiguous at the time (see CLAUDE.md fix in this same change).

## [2.2.0] - 2026-05-18

- `dba5611` (2026-05-17) — feat: add roadmap-first routing to skeptic Future Work and recon gap persistence. **Shipped without a version bump.**
- `ddeb2d0` (2026-05-18) — feat: add Concept & Strategy agent to skeptic plan mode. Added `plan concept` subcommand (Opus agent) reading concept/strategy findings plus recon/landscape reports; added Concept to the overarching plan fan-out (7→8 category agents); removed misleading console footer lines. **Bumped version 2.1.0 → 2.2.0.**

## [2.1.0] - 2026-05-17

- `c7fdb0b` (2026-05-14) — feat: add license tracking manifest (`licenses.json`).
- `89f58c8` (2026-05-17) — feat: add Testing specialist agent to skeptic skill. **Shipped without a version bump.**
- `a249a40` (2026-05-17) — feat: integrate recon into skeptic flow + competitive intelligence in plan. Added Step 3.5 (post-concept-agent recon prompt), inline competitor research fan-out, recon-derived synthesis findings, and opportunistic reads of `.skeptic/recon-*` / `.landscape/` reports by the plan command. **Bumped version 2.0.0 → 2.1.0.**

## [2.0.0] - 2026-05-13

- `57dd7e2` — Initial commit: dispatch v2.0.0, multi-agent analysis toolkit for Claude Code.
