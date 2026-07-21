# Dispatch

Multi-agent analysis and planning toolkit for Claude Code. Fan-out parallel specialists for adversarial review, competitive research, decision analysis, codebase mapping, domain surveys, and migration planning.

## Skills

| Skill | Location |
|-------|----------|
| Skeptic | skills/skeptic/SKILL.md |
| Recon | skills/recon/SKILL.md |
| Tribunal | skills/tribunal/SKILL.md |
| Cartograph | skills/cartograph/SKILL.md |
| Landscape | skills/landscape/SKILL.md |
| Migrate | skills/migrate/SKILL.md |
| Plugin manifest | .claude-plugin/ |

## Conventions

- Documentation-only plugin (no runtime code)
- Skill logic lives entirely in SKILL.md files
- Skill references use `/dispatch:<skill>` namespace (e.g. `/dispatch:skeptic`)
- Future edits to any `skills/*/SKILL.md` should go through `/superpowers:writing-skills`
  (verify-before-deploy workflow) rather than an ad-hoc edit followed by a separate
  consistency audit — see `docs/audits/consistency-audit-2026-07-18.md`,
  `docs/audits/consistency-audit-2026-07-19.md`, and
  `docs/audits/skill-behavior-verification-2026-07-21.md` for the cost of the ad-hoc pattern
  this replaces

## Release Checklist

1. Bump `version` in `.claude-plugin/plugin.json` if user-facing behavior changed. That is the **only** version field in the repo — `.claude-plugin/marketplace.json` carries no `version` key (its `plugins[]` entry holds `name`, `description`, `source`, `category`), so nothing there gets bumped. "User-facing behavior changed" covers: a new skill; a new specialist/agent added to an existing skill; a new subcommand, mode, or routing path added to an existing skill (e.g. a new fan-out step, a new decision branch); or any change to what a skill produces or reads. Docs-only wording/typo fixes do not require a bump.
2. Keep the `description` (and `name`/`category`) in `marketplace.json` in sync with `plugin.json` when the plugin's scope changes — that, not a version, is what marketplace.json tracks.
3. Update README.md if user-facing behavior changed
4. Update skill descriptions in SKILL.md frontmatter if triggers changed
5. Commit and push
