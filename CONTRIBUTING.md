# Contributing to Dispatch

Dispatch is a documentation-only Claude Code plugin — every skill's behavior lives entirely
in its `SKILL.md`. There is no runtime code to build or test; contributing means editing
`SKILL.md` files, `README.md`, and the plugin manifest, and following the discipline below
so the skills stay coherent as a set.

## The fan-out contract

Every skill in this repo follows the same shape (see `README.md`'s architecture line):
a skill fans out **parallel specialist agents with no shared state**, each producing an
independent report within its own lens (e.g. skeptic's 8 domain critics, cartograph's 5
dimension specialists), then a **synthesis step aggregates after** all specialists return —
merging findings, resolving overlaps, and producing the single unified output the user
sees. Specialists never coordinate with each other mid-run and never read each other's
output; synthesis is the only place cross-specialist reasoning happens. Any new skill, or
any new specialist added to an existing skill, should fit this shape: parallel, stateless
relative to its siblings, synthesized after the fact.

## The trigger-description convention

Every `SKILL.md` frontmatter carries a `description:` line shaped as **"Use when..."**:
a list of literal trigger phrases a user might type, followed by an intent clause
describing what kind of request the skill answers. For example (`skills/skeptic/SKILL.md`):

```
description: Use when the user says 'skeptic', 'skeptic fix', 'roast my code', 'critique
this project', 'find flaws', 'what's wrong with this', or wants adversarial review of
architecture, design, code quality, security, performance, developer experience, testing,
or project viability. Spawns parallel specialist critics biased against the project.
```

This is the only routing signal Claude Code has when deciding which skill a natural-language
request should invoke, so the phrasing matters more than it looks: vague or overlapping
triggers cause genuine mis-routes, not just cosmetic ambiguity.

## Disambiguation discipline

Before shipping a new skill, or changing an existing skill's trigger keywords, **check the
new/changed keyword list against all 6 existing skills' `description:` lines for overlap** —
i.e., is there a natural user request that two descriptions would both plausibly claim? This
is exactly the check `docs/audits/trigger-overlap-audit-2026-07-21.md` ran across the current
six skills; read it for the full methodology and findings (O1–O6) rather than re-deriving the
process here.

Two resolution patterns from that audit, reusable for any newly found collision:

- **Redirect-clause pattern (O1–O4).** When a collision is real and asymmetric (one skill can
  run from a cold start, the other has a hard prerequisite; or the two skills answer genuinely
  different questions), resolve it by adding an explicit two-way redirect clause to both
  descriptions — state the distinguishing condition and name the other skill by name (e.g.
  "use recon instead when the goal is comparing this project against a named competitor set
  from an existing skeptic report"). Do not just add more keywords; state the routing rule in
  both directions so a router reading either description lands correctly.
- **Leave-it precedent (O6).** Not every overlap needs a fix. If the overlap is deliberate and
  self-resolving — the audit's example is skeptic's Concept & Strategy agent naturally leading
  into recon via skeptic's own inline "Run recon?" prompt — document the overlap and leave it
  rather than stripping language out to force artificial separation.

Applying a disambiguator is a **user-facing description change** and requires a version bump
per the Release Checklist below.

## Release Checklist

See `CLAUDE.md`'s Release Checklist for the full steps (plugin.json version bump rules,
marketplace.json sync, README/SKILL.md doc updates, commit and push). In short: bump
`.claude-plugin/plugin.json`'s `version` whenever user-facing behavior changes (new skill,
new specialist/agent, new subcommand/mode/routing path, or any change to what a skill
produces or reads); docs-only wording/typo fixes do not require a bump, but a trigger
description change (per the disambiguation discipline above) does.

## Editing workflow

Any edit to a `skills/*/SKILL.md` file goes through **`/superpowers:writing-skills`**
(verify-before-deploy workflow), not an ad-hoc edit followed by a separate consistency
audit — see `CLAUDE.md`'s Conventions section for why (the cost of the ad-hoc pattern is
documented in `docs/audits/consistency-audit-2026-07-18.md`,
`docs/audits/consistency-audit-2026-07-19.md`, and
`docs/audits/skill-behavior-verification-2026-07-21.md`).
