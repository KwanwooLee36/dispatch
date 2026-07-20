# /kerd:lorg — dispatch

Project profile:
  Tech: Documentation-only Claude Code plugin — no runtime code, no package manager.
        Skill logic lives entirely in `skills/*/SKILL.md`. Plugin manifests in `.claude-plugin/`.
  Themes: multi-agent fan-out, adversarial review, competitive/domain research,
          decision analysis, codebase mapping, migration planning, skill authoring.

---

## Tier 1: Installed but not activated here
Last scanned: 2026-07-20

### superpowers:writing-skills — relevance 65 (theme 30, tech 15, recency 20, friction 0)
Creating/editing skills and verifying they work before deployment.

Why here: dispatch **is** a bundle of six Claude Code skills — every future change to
`skills/*/SKILL.md` (new specialist, new mode, trigger wording) is exactly writing-skills'
domain. No evidence it has been invoked here; skill edits have been ad-hoc + audited after
the fact (two consistency audits in `.scorched/`). Adopt it as the standard front-door for
skill edits.

Already installed. Try: /superpowers:writing-skills

### kerd:tend — relevance 30 (theme 0, tech 15, recency 0, friction 0)
Structural health: audit/converge repo to Kerd conventions (dirs, vault, config).

Why here: switch-in flagged missing required structure (`docs/`, `docs/playbook.md`,
`kivna/vault.json`, `docs/archive/`, `kivna/sessions/`, two `.gitignore` entries). tend
decides which of these a docs-only public plugin actually needs vs. intentionally omits.
Owner call — dispatch is a public superset, may deliberately stay minimal.

Already installed. Try: /kerd:tend

## Tier 2: Available but not installed
Last scanned: 2026-07-20

No vault configured (`kivna/vault.json` absent) → curated-source scan (Source B) skipped.
Marketplace: toolkit + superpowers + kerd already installed; no new docs-only-plugin
maintenance gap surfaced. No matches.

## Tier 3: Worth exploring
Last scanned: 2026-07-20

### wshobson/agents — relevance ~18 (reference only)
Multi-harness agentic plugin marketplace (Claude Code, Codex, Cursor, …).

Why here: adjacent to dispatch's fan-out subagent model — a reference point for
parallel-analysis patterns, not a consumer dependency. dispatch is itself an analysis
plugin, so value is comparative, not adoptive.

Explore: https://github.com/wshobson/agents

---
 2 installed matches · 0 available · 1 to explore
---
