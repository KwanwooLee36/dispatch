# Dispatch Trigger-Overlap Audit — 2026-07-21

**Scope:** the `description:` frontmatter of all six `skills/*/SKILL.md` files plus the skill
summaries in `README.md`. The question asked of every pair: *is there a natural user request that
both descriptions plausibly claim?* If yes, the pair is an overlap and needs a disambiguator.

**Report only — no descriptions were reworded.** Recommendations are written as drop-in replacement
lines so a later session can apply them verbatim.

**Relationship to the other 2026-07-21 audit:** `docs/audits/skill-behavior-verification-2026-07-21.md`
traced *executability* (can an agent follow each step and produce the promised artifact). It contains
no trigger/routing finding — its §4 prior-findings table covers F1–F12 and H1–L6, none of which is a
description-overlap item. So nothing below duplicates it. Where it establishes a fact this audit
relies on (e.g. recon's hard skeptic-report prerequisite), it is cited rather than re-derived.

---

## Current trigger inventory

| Skill | Trigger phrases in `description:` | Hard prerequisite |
|-------|-----------------------------------|-------------------|
| skeptic | `skeptic`, `skeptic fix`, `roast my code`, `critique this project`, `find flaws`, `what's wrong with this` | none (codebase in cwd) |
| recon | `recon`, `competitor research`, `market research`, `competitive analysis`, `compare competitors` | **yes** — a prior `/dispatch:skeptic` report with Concept & Strategy findings |
| landscape | `landscape`, `survey`, `research the space`, `what exists for X`, `market research` | none — explicitly standalone |
| tribunal | `tribunal`, `decide`, `compare options`, `should we use X or Y`, `which is better` | 2–4 options; project codebase mandatory |
| cartograph | `cartograph`, `map this codebase`, `explain this repo`, `developer guide`, `onboard me` | codebase in cwd |
| migrate | `migrate`, `migration plan`, `how do I move from X to Y`, `upgrade path` | codebase in cwd; a named source→target pair |

---

## O1 — [HIGH] recon ↔ landscape — the phrase `market research` is in both, verbatim

**Ambiguous request:** *"Do some market research on note-taking apps."*

Both descriptions claim this literally. The consequence is asymmetric, which is what makes it worth
fixing rather than tolerating: landscape runs fine from a cold start, but recon **errors out** if no
skeptic report exists — `skills/recon/SKILL.md:25`, *"Run `/dispatch:skeptic` first. Recon needs
concept agent findings."* So half the time this request routes to a skill that cannot run at all,
and the user is told to run a *third* skill first.

The descriptions do gesture at the distinction — landscape says *"without requiring a prior skeptic
run"*, recon says *"building on skeptic's concept agent findings"* — but both clauses sit at the end,
after the trigger list, and neither is phrased as a routing instruction. A router matching on
`market research` hits the shared phrase before it reads the qualifier.

Second axis, currently undocumented in either description: **what they research.** Landscape surveys
a *domain* the user names (`.landscape/survey-*.md`: solutions catalog, approaches, opportunity
ranking). Recon researches a *specific competitor list extracted from a report about the user's own
project*, and outputs a gap matrix relative to that project. A user who wants "what's out there"
wants landscape; a user who wants "how do we compare to them" wants recon.

**Recommendation — remove the collision and state the routing rule in both directions:**

- **landscape** — keep `market research`, add an explicit redirect:
  > `…'what exists for X', 'market research', or wants standalone domain/market research without requiring a prior skeptic run. Use recon instead when the goal is comparing this project against a named competitor set from an existing skeptic report. Spawns parallel research agents across solutions, approaches, community, and business dimensions.`
- **recon** — drop the bare `market research` trigger (it is landscape's by breadth) and make the
  prerequisite a gate, not a footnote:
  > `Use when the user says 'recon', 'competitor research', 'competitive analysis', 'compare competitors', or wants deep competitive analysis of this project against competitors already named in a skeptic report. Requires a prior /dispatch:skeptic run with the concept agent — use landscape instead for open-ended domain research with no prior report.`

---

## O2 — [HIGH] tribunal ↔ migrate — "should we move from X to Y" satisfies both

**Ambiguous request:** *"Should we move from Express to Fastify?"*

- tribunal claims `should we use X or Y`, `decide`, `which is better`.
- migrate claims `how do I move from X to Y`, `migration plan`, `upgrade path`.

The request contains both the decision verb ("should we") and the migration shape ("from X to Y").
The two skills answer genuinely different questions — tribunal produces a comparison matrix and a
recommendation with a confidence level; migrate assumes the decision is made and produces a
risk-ranked execution plan with a go/no-go — but nothing in either description says *which question
you are being asked*.

Note this is not purely academic: migrate's own output includes a go/no-go verdict, so it partly
answers the decision question too, which makes the mis-route feel plausible rather than obviously
wrong.

**Recommendation — split on decided-vs-undecided:**

- **tribunal** — add: `Use for a decision that has not been made yet; if the choice is already made and the question is how to execute the move, use migrate.`
- **migrate** — add: `Assumes the decision is already made; use tribunal first if the user is still choosing between the target and alternatives.`

---

## O3 — [MEDIUM] tribunal ↔ landscape — "what are the options for X"

**Ambiguous request:** *"What are the options for state management in React?"*

- landscape claims `what exists for X`, `research the space`, `survey`.
- tribunal claims `compare options`, `which is better`.

Distinguisher that exists in behaviour but not in the descriptions: **tribunal requires the user to
supply 2–4 named options** (`skills/tribunal/SKILL.md` Step 2 errors otherwise — *"Tribunal requires
2-4 options. You provided X."*), and it requires a project codebase for grounding. Landscape takes an
unbounded domain string and discovers the options itself. So the real rule is: *named options →
tribunal; "find me the options" → landscape.*

**Recommendation:**

- **tribunal** — add after the trigger list: `The user must name 2-4 specific options; if they are asking what the options even are, use landscape.`
- **landscape** — no change needed if O1's redirect clause is applied; the phrase *"standalone
  domain/market research"* already reads as discovery rather than adjudication.

---

## O4 — [MEDIUM] recon ↔ tribunal — "compare competitors" vs "compare options"

**Ambiguous request:** *"Compare us against Notion and Obsidian — which is better?"*

`compare competitors` (recon) and `compare options` / `which is better` (tribunal) are one word
apart. The names in the request are simultaneously "competitors" and "options".

Distinguisher: tribunal adjudicates a choice **the user will make** (pick one of these to use);
recon analyses products the user **competes with** (find gaps against these). Tribunal's output is a
recommendation; recon's is a gap matrix and a differentiator map. Additionally, recon can only run
when those names already appear in a skeptic report — a user typing them freshly into the prompt
does not satisfy recon's input contract at all.

**Recommendation:**

- **recon** — the O1 rewrite already scopes it to *"competitors already named in a skeptic report"*,
  which resolves most of this. Optionally strengthen with: `Not for choosing between tools to adopt — that is tribunal.`
- **tribunal** — add: `For options you might adopt; for products you compete with, use recon.`

---

## O5 — [LOW] cartograph ↔ skeptic — "analyse my codebase"

**Ambiguous request:** *"Take a look at this codebase and tell me about the architecture."*

skeptic reviews architecture, design, and code quality; cartograph maps structure, data flow, and
conventions. Both read the whole repo and both report on architecture.

They are already fairly well separated by *tone* words — skeptic's triggers are all hostile
(`roast`, `critique`, `find flaws`, `what's wrong`) and cartograph's are all explanatory
(`explain`, `developer guide`, `onboard me`) — so a request with either valence routes correctly.
Only a **neutral** request ("analyse", "look at", "tell me about") is genuinely ambiguous, and the
descriptions do carry the right words for it: skeptic says *"adversarial review"*, cartograph says
*"generating a developer guide"*.

**Recommendation:** no wording change required. If anything is added, the cheapest is a single
clause on cartograph: `Descriptive, not critical — use skeptic when the user wants problems found rather than the system explained.`

---

## O6 — [LOW] skeptic ↔ recon on "project viability"

Skeptic's Concept & Strategy agent covers market positioning, differentiation, and competitors, and
skeptic's own Step 3.5 can run recon's research inline. A request like *"is there a market for
this?"* matches skeptic's *"project viability"* clause and recon's *"competitive analysis"*.

This one is **intentional and self-resolving**: skeptic prompts *"Run recon?"* when its concept agent
finds competitors, so entering through skeptic reaches recon anyway. Worth recording only so a future
pass does not "fix" it by pulling the strategy language out of skeptic's description.

**Recommendation:** no change. Documented here as a deliberate overlap.

---

## Summary

| ID | Pair | Severity | Collision | Fix shape |
|----|------|----------|-----------|-----------|
| O1 | recon ↔ landscape | HIGH | `market research` appears verbatim in both | drop from recon; add two-way "use X instead when Y" clauses |
| O2 | tribunal ↔ migrate | HIGH | "should we move from X to Y" matches both | split on decided vs. undecided |
| O3 | tribunal ↔ landscape | MEDIUM | "what are the options for X" | split on named options (2-4) vs. discovery |
| O4 | recon ↔ tribunal | MEDIUM | `compare competitors` vs `compare options` | split on adopt vs. compete-with |
| O5 | cartograph ↔ skeptic | LOW | neutral "analyse this codebase" | optional one clause; tone words mostly suffice |
| O6 | skeptic ↔ recon | LOW | "project viability" / market questions | none — deliberate, skeptic offers recon inline |

**Applying these is a user-facing description change** and would therefore require a version bump per
`CLAUDE.md`'s Release Checklist. They were deliberately not applied in this pass.
