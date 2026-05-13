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

## Release Checklist

1. Bump version in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
2. Update README.md if user-facing behavior changed
3. Update skill descriptions in SKILL.md frontmatter if triggers changed
4. Commit and push
