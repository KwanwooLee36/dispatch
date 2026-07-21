# Competitive Gap Research — dispatch vs. the public Claude Code plugin landscape

**Date:** 2026-07-21
**Scope:** Report only. No skill file was modified.
**Trigger:** `docs/lorg-report.md` Tier 3 (`wshobson/agents`, relevance ~18, "reference only") plus a
broader survey of public Claude Code plugin marketplaces and multi-agent fan-out collections.

---

## 0. How to read this

Every finding carries a confidence tag:

- **[P]** *Primary* — verified by me against repo files (GitHub API tree/contents), official Anthropic
  docs, or dispatch's own `skills/*/SKILL.md`.
- **[S]** *Secondary* — from a search-result paraphrase or third-party write-up. Directionally useful,
  not citable as fact. Do not act on an **[S]** finding without confirming it first.

Nothing here is a mandate. Section 4 ranks recommendations and Section 5 lists what dispatch should
deliberately **not** copy.

---

## 1. What was surveyed

| Source | What it is | Scale (verified) | Conf. |
|---|---|---|---|
| [wshobson/agents](https://github.com/wshobson/agents) | Multi-harness agentic plugin marketplace (Claude Code, Codex CLI, Cursor, OpenCode, Gemini CLI, Copilot) | 90 plugin dirs under `plugins/`; 180 `SKILL.md` files; README claims 94 plugins / 203 agents / 175 skills / 109 commands | [P] |
| [anthropics/claude-code](https://github.com/anthropics/claude-code) official marketplace (`claude-plugins-official`) | First-party plugins, auto-added | `pr-review-toolkit`, `code-review`, `security-guidance`, others | [S] |
| [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents) | 100–130 subagent definitions, category-numbered | 10 numbered categories incl. `09-meta-orchestration`, `10-research-analysis` | [S] |
| [Claude Code plugins reference](https://code.claude.com/docs/en/plugins-reference) | Official manifest/component spec | Full `plugin.json` + `marketplace.json` schema | [P] |
| 2026 orchestration write-ups (claudefa.st, alexop.dev, Totalum, MindStudio, Shipyard) | Third-party pattern catalogues | fan-out, adversarial verification, tournament, grader/rubric | [S] |

**dispatch baseline (verified [P]):** 6 skills, all single-file. Sizes: skeptic 60.8 KB, landscape
20.5 KB, tribunal 18.1 KB, recon 17.9 KB, migrate 16.8 KB, cartograph 16.6 KB. No `agents/`,
`commands/`, `hooks/`, `references/`, or `assets/` directories anywhere. 23 model declarations across
all skills: **16 Opus, 7 Sonnet — zero Haiku, zero `inherit`.**

**Correction to the lorg entry:** wshobson's README advertises "16 orchestrators (multi-agent
coordination workflows)". The actual tree contains **2** orchestration commands
(`agent-orchestration/commands/{improve-agent,multi-agent-optimize}.md`) and ~6 files with
"orchestrator" in the name, most of which are ordinary domain agents (`tdd-orchestrator`,
`eval-orchestrator`, `ship-mate/orchestrate`). Its orchestration surface is **thinner than
dispatch's**, not thicker. [P] Treat "16 orchestrators" as marketing.

---

## 2. Skill categories dispatch does not cover

dispatch's charter is *multi-agent analysis and planning* — not code generation, not language
tooling. So the honest comparison set is only the analysis/planning slice of these marketplaces. Most
of wshobson's 90 plugins (`python-development`, `game-development`, `seo-content-creation`) are
irrelevant by charter, not gaps.

### 2A. Genuine gaps inside dispatch's charter

| # | Gap | Evidence elsewhere | Why it fits dispatch | Conf. |
|---|---|---|---|---|
| G1 | **Diff / PR-scoped review.** skeptic reviews a whole project from cwd; there is no mode that takes a diff, branch, or PR as the unit of analysis. | Official `pr-review-toolkit` fans out 6 distinct reviewers over a PR (Comment Analyzer, PR Test Analyzer, Silent Failure Hunter, Type Design Analyzer, Code Reviewer, Code Simplifier); `code-review` plugin likewise. wshobson `comprehensive-review`, `git-pr-workflows`. | Same fan-out shape skeptic already uses, different scope object. The most-requested review granularity in the ecosystem, and the one dispatch structurally cannot serve today. | [S] for toolkit roster, [P] for dispatch's absence |
| G2 | **Incident response / postmortem fan-out.** No skill takes a failure and fans specialists over it. | wshobson `incident-response`, `error-diagnostics`, `distributed-debugging`. | Naturally parallel (logs / recent diffs / infra / dependency-change / blast-radius lenses) and terminates in a synthesis verdict — dispatch's exact template. | [P] (plugin dirs verified) |
| G3 | **Test-strategy analysis as its own skill.** Testing is 1 of skeptic's 8 critics; there is no standalone "what should our test suite be" fan-out. | wshobson `tdd-workflows`, `unit-testing`, `performance-testing-review`; official PR Test Analyzer. | skeptic's testing critic only *attacks*; it never proposes a coverage strategy (skeptic is explicitly "no fixes — only exposure"). | [P] |
| G4 | **Dependency / supply-chain audit.** Nothing reads a lockfile. | wshobson `dependency-management`, `security-scanning`, `signed-audit-trails`. | Fits the read-only, report-producing mould. Note dispatch already tracks `licenses.json` for itself — the capability is adjacent. | [P] |
| G5 | **Self-evaluation of the plugin's own agents.** dispatch has no mechanism to score whether a specialist prompt is working. | wshobson `plugin-eval` plugin + `eval-orchestrator` agent; `review-agent-governance`. | dispatch's whole value is prompt quality across 23 agent definitions, and it currently verifies them only by hand-written audits (`docs/audits/skill-behavior-verification-2026-07-21.md`). | [P] |
| G6 | **Architecture-diagram output (C4 / structural rendering).** cartograph emits prose + tables only. | wshobson `c4-architecture`. | Low cost — a diagram section in the existing cartograph synthesis, not a new skill. | [P] |
| G7 | **Accessibility and observability lenses.** No a11y critic; no observability/monitoring critic. | wshobson `accessibility-compliance`, `observability-monitoring`. | Would be skeptic critics #9/#10, not new skills. Weakest of the gaps — see §5. | [P] |

### 2B. Categories dispatch covers that the surveyed collections do **not**

Worth stating plainly, because it defines the moat and should survive into positioning copy:

- **Structured adversarial review with scoring and history.** No surveyed collection scores a project
  0–100 per category, tracks repeat offenders across runs, or escalates severity after 3+ appearances.
  wshobson's review plugins are single-pass. [P] for dispatch's behaviour; [S] for the absence claim.
- **Multi-option decision arbitration (tribunal).** Advocate-per-option with a synthesis arbiter has no
  analogue in the surveyed set. [S]
- **Competitive/market research wired into engineering artifacts (recon + landscape).** Nothing else
  surveyed routes competitor gaps into `docs/roadmap.md` / `TODO.md`. [S]
- **Cross-skill data flow.** skeptic → inline recon → `/skeptic plan` reading `.skeptic/recon-*.md` and
  `.landscape/report-*.md` is a genuine pipeline. wshobson's `context-manager` agent is the only
  comparable idea and is far less concrete. [P]

**Conclusion for §2:** dispatch is not behind on *breadth*; it is deliberately narrow and deeper than
the alternatives inside its lane. G1 is the only gap that a user would plausibly switch plugins over.

---

## 3. Fan-out and prompt patterns dispatch does not use

This is the section with the most transferable value.

### 3A. Verification and quality-control patterns

| # | Pattern | What it is | dispatch today | Conf. |
|---|---|---|---|---|
| F1 | **Adversarial verification of the fan-out's own output.** For each producer agent, a *separate* agent that has never seen the producer's work tries to knock the finding down against a rubric; a finding survives only if the skeptic cannot. Producer and verifier never share a context window, which removes self-preferential bias. | dispatch is adversarial *toward the project* but never *toward its own findings*. Specialist output goes straight into synthesis, which dedups and scores but does not challenge. A hallucinated MAJOR finding is currently unfiltered. | [S] (pattern), [P] (dispatch's absence — verified across all 6 SKILL.md) |
| F2 | **Grader + rubric + forced revision.** A grader scores each subagent result against a rubric and sends it back to revise until it clears the bar. Reported as an Anthropic "Performance Outcomes" capability, June 2026. | dispatch accepts the first response from every agent. No re-dispatch, no quality floor. | [S] |
| F3 | **Tournament / judge panel.** N agents attempt the *same* task in different ways; judges compare until one wins. Used where there is no single right answer. | tribunal looks similar but is not: each advocate is *assigned a different option* and there is one synthesis pass, no elimination rounds, no repeated judging. A true tournament mode (same question, N independent attempts, judged) is absent. | [S] (pattern), [P] (tribunal's actual mechanism) |
| F4 | **Generate-and-filter.** Generate many candidates, filter by rubric or verification, dedup, return only tested survivors. | dispatch dedups (`>70% title-only word overlap`, verified in skeptic/recon/landscape) but never *filters by quality* — every non-duplicate finding ships. | [P] for dedup, [S] for the pattern |

**F1 is the single highest-value import.** It is cheap (one extra agent class), it directly targets
dispatch's most credible failure mode (an adversarial critic that invents a flaw to satisfy its hostile
framing), and it composes with what already exists — synthesis already has a canonical finding format
(`same file + same severity + similar title`) that a verifier could consume unchanged.

### 3B. Orchestration-shape patterns

| # | Pattern | dispatch today | Conf. |
|---|---|---|---|
| F5 | **Dynamic fan-out width** — the lead decides N at runtime rather than shipping a fixed roster. | Fixed rosters: skeptic 8, cartograph 5, migrate 5, landscape 4. Only recon and tribunal scale to input. A trivial repo pays for all 8 skeptic critics. | [P] |
| F6 | **Waves with decision gates between them.** | recon batches in waves of 10 (verified, `skills/recon/SKILL.md:61,246`) but the waves are pure throughput batching — no gate re-plans wave 2 based on wave 1. Every other skill is single-wave: fan out → synthesize. | [P] |
| F7 | **Orchestrator writes a coordination program** so coordination itself costs no model tokens. | Not applicable today — dispatch is documentation-only with no runtime code, a deliberate constraint (`CLAUDE.md`). Noted for completeness; adopting it would break the docs-only property. | [S] |
| F8 | **Shared-state / context-broker agent.** wshobson ships `agent-orchestration/agents/context-manager.md`. | dispatch passes no context between agents at all — "No additional context is passed — the agent discovers the project by exploring from cwd" (`skills/skeptic/SKILL.md:621`). This is a defensible independence choice, but it means 8 agents each re-explore the same tree. | [P] |
| F9 | **Progress observability via `SubagentStart` / `SubagentStop` hooks or a `Monitor`.** Both are first-class plugin components in the official spec. | dispatch prints wave progress text only in recon. A long skeptic run is otherwise silent. | [P] |

### 3C. Model-tier policy

wshobson documents a 4–5 tier policy [P, from `ARCHITECTURE.md`]:

| Tier | Model | Applied to |
|---|---|---|
| 1 | Opus | Architecture, security, production-critical |
| 2 | `inherit` | Complex tasks — the *user* picks the model |
| 3 | Sonnet | Documentation, testing, debugging |
| 4 | Haiku | Fast, simple operations |

(a README variant also lists a Tier 0 = Fable 5 for long-horizon autonomous work [S].)

dispatch uses **two tiers, 16 Opus / 7 Sonnet, and pins every one of them** [P]. Two consequences:

- **No cheap tier.** Mechanical passes — routing items into `TODO.md`, dedup arithmetic, formatting the
  console block — run on Sonnet or Opus. Haiku would serve them.
- **No user choice.** `inherit` lets a cost-sensitive user run the whole fan-out on their chosen model.
  dispatch's `quick` modes are the only cost lever, and they are coarse (whole-skill, not per-agent).

---

## 4. Packaging, versioning, and doc conventions worth adopting

### 4A. Progressive disclosure — the strongest structural finding

wshobson's authoring spec [P, `docs/authoring.md` + `ARCHITECTURE.md`]:

> "Progressive disclosure all the way down. Context files cap at ~150 lines. Skill bodies cap at ~8 KB."

Three tiers per skill:

1. `SKILL.md` (≤8 KB) — navigation, quick start, a one-paragraph decision tree, links out
2. `references/` — `details.md`, `api-reference.md`, `examples/`
3. `assets/` — templates, configs, scaffolding loaded by name

They lint it: `SKILL_OVER_CODEX_CAP` fires on any skill over 8 KB with no `references/` directory.

**Measured against dispatch [P]: all six skills would fail that lint.** Every one exceeds 8 KB and none
has a `references/` directory. skeptic at 60.8 KB is **7.6× the cap**. The official Claude Code spec
explicitly supports the split — "Skills can include supporting files alongside SKILL.md",
`skills/<name>/{SKILL.md, reference.md, scripts/}`.

Concretely, skeptic's 1,061 lines contain: 8 full specialist prompts, an inline recon protocol, a
synthesis protocol, a plan mode with 9 per-category sub-prompts, routing rules, and console-output
templates. The 9 `/skeptic plan <type>` sub-prompts alone (≈lines 846–971) are loaded on every single
skeptic invocation even though a plain `/skeptic full` never reads them. That is the textbook case for
`references/plan-modes.md`.

### 4B. Manifest fields dispatch is leaving on the table

Verified against the official plugins reference [P]. dispatch's `plugin.json` sets `name`,
`description`, `version`, `author`, `homepage`, `repository`, `license`, `keywords`. Unused and
relevant:

| Field | Value to dispatch |
|---|---|
| `$schema` | Editor autocomplete + validation. `marketplace.json` has one; `plugin.json` does not. Free consistency win. |
| `displayName` | `"Dispatch"` in the `/plugin` picker instead of the bare slug (needs CC ≥ 2.1.143). |
| `author.email` / `author.url` | Currently name-only. |
| `defaultEnabled` | Not obviously wanted — dispatch's skills are opt-in by invocation, not ambient. Listed for completeness. |
| `dependencies` | dispatch documents Kerd integration in prose only. If any behaviour becomes hard-required, this is the declarative home. Today the "works standalone" property is correct and should be kept. |

**Version semantics [P]:** setting `version` in `plugin.json` *pins* the plugin — users receive updates
only on a bump; pushing commits without bumping produces "already at the latest version". dispatch sets
`version: 2.2.3` in `plugin.json` and **omits `version` from the marketplace entry**. Per the
resolution order (`plugin.json` → marketplace entry → commit SHA → `unknown`) this works, and
`plugin.json` wins on conflict, so the current setup is correct and the omission is harmless. But it
makes the existing release checklist step ("bump version in **both** `plugin.json` and
`marketplace.json`") describe something that is not actually there — `marketplace.json` has no
`version` key at all. **That is a live doc/reality mismatch in `CLAUDE.md` worth fixing.**

### 4C. Mechanical validation — dispatch has none

wshobson's CI gates [P]:

- `make validate` — structural validation of every artifact; errors block CI, warnings advisory
- `make garden` — drift detection: dead links, orphaned marketplace entries
- `make test` — 386-test pytest suite
- `tools/check_agent_name_collisions.py --fail-on-duplicates`
- a `cli-smoke-test` job that installs the generated artifacts into real harnesses
- named lint codes: `MISSING_TRIGGER`, `SKILL_OVER_CODEX_CAP`, `AGENT_NAME_COLLISION`,
  `BARE_MODEL_ALIAS`, `harness_portability`

dispatch has **zero** automated validation [P] — no CI, no `claude plugin validate` invocation, no link
checker. Its quality mechanism is hand-written audit documents (five in `docs/audits/`,
plus the dogfood harness on an unmerged branch). Those audits are good, but they are
manual, dated, and re-derived each time. A minimal GitHub Action running `claude plugin validate` plus
a link check would catch the mechanical subset for near-zero maintenance and free the audits to do the
judgement work only a human/agent can.

Note the dogfood-verification-harness branch already referenced in `TODO.md` covers behavioural
conformance. **This finding is about the cheap mechanical layer beneath it, not a replacement for it.**

### 4D. Description-trigger discipline — dispatch already passes

wshobson lints `MISSING_TRIGGER`: a description must contain `"Use when …"`, `"Use this skill when …"`,
`"Use PROACTIVELY when …"`, `"Use after …"`, `"Trigger when …"`, or `"Auto-loads when …"`.

**Verified [P]: all six dispatch skills open with `Use when the user says '…'`.** dispatch would pass
this lint cleanly today. Worth writing the convention down in `CLAUDE.md` so it survives the next
author.

### 4E. Repo-hygiene files present elsewhere, absent here [P]

wshobson root: `AGENTS.md`, `ARCHITECTURE.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `GEMINI.md`,
`Makefile`, `.markdownlint.json`, `docs/` (9 files: `agents.md`, `agent-skills.md`, `architecture.md`,
`authoring.md`, `harnesses.md`, `plugins.md`, `usage.md`, …).

dispatch root: `CLAUDE.md`, `CHANGELOG.md`, `README.md`, `LICENSE`, `licenses.json`, `TODO.md`.
Missing and plausibly worth it: **`CONTRIBUTING.md`** (dispatch is public; there is no stated bar for a
new specialist or a new skill) and an **authoring/architecture doc** capturing the fan-out contract that
currently lives implicitly across six SKILL.md files. dispatch already has `docs/roadmap.md` and
`docs/audits/`; the gitignored local-workflow files (`docs/playbook.md`, `docs/archive/`,
`.slainte`, `kivna/vault.json`) are a separate, partly-overlapping concern that `/kerd:tend` owns.

### 4F. Multi-harness distribution — noted and not recommended

wshobson authors once in Markdown and generates harness-native artifacts for Codex, Cursor, OpenCode,
Gemini CLI, and Copilot, with checked-in per-harness registries and `make generate-all` to prevent
drift [P]. This is the single largest structural difference between the two projects.

It is also a large permanent cost: a generation toolchain, five registries, and round-trip tests. For a
solo-maintained six-skill plugin whose entire mechanism is Claude Code's `Agent` tool, porting is close
to meaningless — the fan-out does not exist on the other harnesses. **Recommend explicitly declining
this and recording the decision**, so it does not get re-litigated at every future lorg scan.

---

## 5. Ranked recommendations

Ordered by (value to dispatch) ÷ (cost + risk). Each is a proposal for owner decision, not a queued task.

| Rank | Recommendation | Cost | Why it ranks here |
|---|---|---|---|
| **1** | **Adopt progressive disclosure (§4A).** Split each skill into `SKILL.md` + `references/`. Start with skeptic: move the 9 `/skeptic plan <type>` sub-prompts and the 8 specialist prompt bodies into `references/`. | Medium, mechanical | Largest measurable defect found: 60.8 KB loaded on every invocation vs. an 8 KB industry cap. Pure win — no behaviour change, less context burned, and it makes the remaining SKILL.md reviewable. |
| **2** | **Add an adversarial-verification pass (F1).** A verifier agent per specialist (or one verifier over the merged finding set) that has not seen the producing agent's reasoning and must knock findings down against a rubric before synthesis scores them. | Medium | Directly hardens dispatch's core claim. The canonical finding format already in skeptic (`skills/skeptic/SKILL.md:631`) is the ready-made interface. |
| **3** | **Fix the release-checklist / manifest mismatch (§4B)** and add `$schema` + `displayName` to `plugin.json`. | Trivial | `CLAUDE.md` instructs bumping a `version` field that does not exist in `marketplace.json`. Doc lying about reality. |
| **4** | **Add minimal CI: `claude plugin validate` + a link check (§4C).** | Small | Near-zero maintenance; catches the mechanical class of error the manual audits currently spend effort on. Complements, does not replace, the dogfood harness. |
| **5** | **Introduce a Haiku tier and consider `inherit` (§3C).** Route mechanical passes (roadmap/TODO routing, console formatting, dedup arithmetic) to Haiku. | Small | 16-Opus/7-Sonnet with no cheap tier is the ecosystem outlier. Verify quality per-agent before switching any *analysis* agent down. |
| **6** | **Add a diff/PR-scoped review mode (G1)** — e.g. `/dispatch:skeptic diff` or `/dispatch:skeptic pr <n>` reusing the existing critics with the diff as the scope object. | Medium | The one capability gap a user might switch over. Reuses the whole existing roster, so cost is scoping logic, not new prompts. |
| **7** | **Write `CONTRIBUTING.md` + an authoring doc (§4E)**, including the `Use when …` trigger convention (§4D) dispatch already follows. | Small | Public repo with no stated contribution bar. |
| **8** | **Dynamic fan-out width (F5)** — let skeptic pick its roster from repo signals instead of always spending 8 critics. | Medium | Real saving, but risks the "maximum coverage" guarantee `/skeptic full` sells. Needs design thought first. |
| **9** | **Evaluate G2–G6 (incident response, test strategy, dependency audit, self-eval, C4 diagrams) as future skills.** | Varies | All plausible; none urgent. G5 (self-eval) is the most on-brand — dispatch's product *is* prompts. G6 is the cheapest (a cartograph section, not a skill). |

### Explicitly recommend NOT doing

- **Multi-harness distribution (§4F).** Fan-out via the `Agent` tool is Claude-Code-specific; the port
  has no destination. Record the decline.
- **Breadth-matching wshobson's 90 plugins.** Different product. dispatch's narrowness is the moat
  (§2B), and its charter is analysis and planning, not language/domain tooling.
- **Accessibility and observability critics (G7) *right now*.** They stretch skeptic's roster from 8 to
  10 and worsen the fixed-width cost problem in F5. Resolve F5 first, then reconsider.
- **A runtime/coordination program (F7).** Would break the documentation-only property that `CLAUDE.md`
  states as a core convention.

---

## 6. Open questions for the owner

1. Is the 8 KB `SKILL.md` cap a target dispatch wants to hit, or is "smaller than 60 KB" enough?
   (Codex compatibility, wshobson's stated reason, does not apply to dispatch.)
2. Does adversarial verification (rec. 2) run on every skeptic invocation, or only in `full` mode?
   It roughly doubles agent count in the naive form.
3. Is a diff/PR review mode (rec. 6) in charter, or does dispatch cede that to the official
   `pr-review-toolkit` and stay whole-project?
4. Should the multi-harness decline (§4F) be written into `CLAUDE.md` as a standing scope boundary so
   future lorg scans stop resurfacing it?

---

## 7. Sources

- [wshobson/agents](https://github.com/wshobson/agents) — README, `ARCHITECTURE.md`, `docs/authoring.md`, and the recursive git tree via the GitHub API
- [Claude Code plugins reference](https://code.claude.com/docs/en/plugins-reference) — manifest schema, component spec, version management
- [Discover and install prebuilt plugins](https://code.claude.com/docs/en/discover-plugins)
- [PR Review Toolkit plugin](https://claude.com/plugins/pr-review-toolkit) · [Code Review plugin](https://claude.com/plugins/code-review)
- [anthropics/claude-code `.claude-plugin/marketplace.json`](https://github.com/anthropics/claude-code/blob/main/.claude-plugin/marketplace.json)
- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)
- [Claude Code Agent Patterns: Orchestration Strategies](https://claudefa.st/blog/guide/agents/agent-patterns) · [Dynamic Workflows in Claude Code](https://claudefa.st/blog/guide/development/dynamic-workflows)
- [Claude Code Workflows: Deterministic Multi-Agent Orchestration](https://alexop.dev/posts/claude-code-workflows-deterministic-orchestration/)
- [Claude Code subagents: the 2026 production playbook](https://www.totalum.app/blog/claude-code-subagents-totalum)
- [Claude Code Subagents and Multi-Agent Orchestration Guide](https://hidekazu-konishi.com/entry/claude_code_subagents_and_orchestration_guide.html)
- [Multi-agent orchestration for Claude Code in 2026](https://shipyard.build/blog/claude-code-multi-agent/) · [Ultra Code Mode](https://www.mindstudio.ai/blog/claude-code-ultra-code-mode-deep-research-complex-tasks)
- Local: `README.md`, `CLAUDE.md`, `.claude-plugin/{plugin,marketplace}.json`, `skills/*/SKILL.md`, `docs/lorg-report.md`
