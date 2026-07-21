# Dispatch Exhaustive Consistency Audit — 2026-07-19

Cross-check of all 6 `skills/*/SKILL.md`, `README.md`, and `CLAUDE.md`. Dimensions
audited: slash-command / namespace references, model-roster claims, output file paths,
error-handling tables, permission-grant statements.

**Ground truth**: plugin name is `dispatch` (`.claude-plugin/plugin.json`), and
`CLAUDE.md:21` mandates the `/dispatch:<skill>` command namespace.

Report-only. Nothing fixed. Findings ranked most-severe first.

---

## F1 — [MAJOR] Recon documents a broken command namespace (`/skeptic:recon`)

`recon` skill's invocation is documented as `/skeptic:recon`, but the plugin is named
`dispatch` and the skill's own frontmatter name is `recon` — so the real command is
`/dispatch:recon`. Typing `/skeptic:recon` targets the wrong plugin namespace.

- `skills/recon/SKILL.md:13` — `/skeptic:recon         # Full mode ...`
- `skills/recon/SKILL.md:14` — `/skeptic:recon quick   # Quick mode ...`
- `skills/recon/SKILL.md:19` — "The project where `/skeptic:recon` is invoked (cwd)."
- `README.md:76-77` — Recon section repeats `/skeptic:recon` / `/skeptic:recon quick`.

**Contradicted by the rest of the repo** (which uses the correct `/dispatch:recon`):
- `README.md:55` — "same protocol as `/dispatch:recon`." → README self-contradicts (line 55 vs line 76).
- `skills/skeptic/SKILL.md:369` — "Run `/dispatch:recon` separately later if needed."
- `skills/skeptic/SKILL.md:1059` — "For deeper competitive research: /dispatch:recon"
- `CLAUDE.md:21` — namespace is `/dispatch:<skill>`.

**Impact**: the one documented way to invoke recon standalone is wrong; users copy-paste a
command that does not resolve.

---

## F2 — [MAJOR] Skeptic reads `.landscape/report-*.md`, but landscape writes `.landscape/survey-*.md`

Skeptic's plan mode loads competitive/market intelligence by globbing for
`.landscape/report-*.md`. The landscape skill never produces that filename — its output is
always `survey-YYYY-MM-DD-{slug}.md`. The glob can never match, so landscape data is
silently never fed into skeptic plans.

- Skeptic globs `.landscape/report-*.md`:
  - `skills/skeptic/SKILL.md:780` — "Glob for `.skeptic/recon-*.md` and `.landscape/report-*.md`."
  - `skills/skeptic/SKILL.md:957` — plan concept reads "landscape reports (`.landscape/report-*.md`)".
  - `README.md:59` — "landscape (`.landscape/report-*.md`) reports when they exist."
- Landscape actually writes `survey-*.md`:
  - `skills/landscape/SKILL.md:393` — "Write the full synthesis output to `.landscape/survey-{DATE}-{slug}.md`".
  - `skills/landscape/SKILL.md:399` / `:449` — same `survey-` prefix in flow + console.

**Impact**: functional break in the "Plan with Competitive Intelligence" feature — the
`{COMPETITIVE_INTELLIGENCE_BLOCK}` will always be empty for landscape data.

---

## F3 — [MAJOR] README claims *all* skills persist design docs to `docs/designs/`; recon does not

`README.md:157-159` ("Design Doc Persistence") states: "All skills can persist their output
as design docs in `docs/designs/`." Every skill implements a `docs/designs/*` persistence
step **except recon**. Recon's only file-out mechanism is per-gap "design notes" written to
`.skeptic/recon-designs/` — a different directory and a different artifact.

- README universal claim: `README.md:159`.
- Skills that DO have a `docs/designs/` step: skeptic `:786-815`, tribunal `:318-352`,
  cartograph `:301-335`, landscape `:471-505`, migrate `:335-369`.
- Recon has NO `docs/designs/` step — its persistence is:
  - `skills/recon/SKILL.md:324-349` — Step 7 writes to `.skeptic/recon-designs/[gap].md`.
  - `skills/recon/SKILL.md:419-420` — Step 10 "Teardown" is an empty heading (no design-doc step follows).

**Impact**: documentation over-promises; a user expecting recon output in `docs/designs/`
(and auto-injected into CLAUDE.md like the others) never gets it.

---

## F4 — [MAJOR] Skeptic dispatch step omits the Testing agent's model assignment

The Step 3 "Model assignment" bullet list — the operative instruction for spawning agents —
assigns models to only 7 of the 8 agents. The Testing agent is missing entirely.

- `skills/skeptic/SKILL.md:121-123`:
  - `model: "opus"` → Architecture, Design, Security, Concept & Strategy
  - `model: "sonnet"` → Code Quality, Performance, DX  ← **Testing absent**
- Contradicted by:
  - `skills/skeptic/SKILL.md:39` — "Sonnet: Code Quality, Performance, DX, **Testing**".
  - `skills/skeptic/SKILL.md:34` — roster lists Testing as `sonnet`.
  - `README.md:51` — Testing = Sonnet.

**Impact**: an executor following Step 3 literally has no model for Testing; behavior is
undefined for that agent.

---

## F5 — [MAJOR] Skeptic permission statement contradicts its own design-doc writes

`skills/skeptic/SKILL.md:563` asserts: "Write (TODO.md or docs/roadmap.md only — these are
the only places skeptic writes outside `.skeptic/`)." But skeptic's plan design-doc
persistence writes to `docs/designs/` and appends to `CLAUDE.md`.

- Permission claim: `skills/skeptic/SKILL.md:563`.
- Writes that violate it:
  - `skills/skeptic/SKILL.md:790,794` — writes `docs/designs/skeptic-plan-{type}-{slug}-YYYY-MM-DD.md`.
  - `skills/skeptic/SKILL.md:806-811` — appends a `## Design Docs` block to project `CLAUDE.md`.

**Impact**: the stated write-scope is inaccurate — `docs/designs/` and `CLAUDE.md` are
additional write targets the permission line denies exist.

---

## F6 — [MINOR] Skeptic Invocation quick-reference drops `concept` from plan types

The one-line plan-type list in the Invocation block omits `concept`, while the authoritative
argument-routing table and README both include it (and a full `plan concept` subcommand
exists).

- Missing `concept`: `skills/skeptic/SKILL.md:18` — `arch|design|code|security|perf|dx|test|debt`.
- Includes `concept`:
  - `skills/skeptic/SKILL.md:762` — routing lists `/skeptic plan concept`.
  - `skills/skeptic/SKILL.md:955-971` — full `plan concept` subcommand spec.
  - `README.md:37` — `arch|design|code|security|perf|dx|test|concept|debt`.
- Same omission recurs in the fix-console category list: `skills/skeptic/SKILL.md:727` and
  `:731` (`arch|design|code|security|perf|dx|test|debt`, no `concept`).

---

## F7 — [MINOR] Systemic bare-command form vs the `/dispatch:` namespace convention

`CLAUDE.md:21` documents commands as `/dispatch:<skill>`, but every SKILL invocation block
and the README use the bare form (`/skeptic`, `/tribunal`, `/cartograph`, `/landscape`,
`/migrate`). Consistent among themselves, but inconsistent with the stated convention.

- Convention: `CLAUDE.md:21`.
- Bare form: `README.md:30-37,89-91,103-107,126-129,140-142`;
  `skills/skeptic/SKILL.md:13-20`; `skills/tribunal/SKILL.md:12-16`;
  `skills/cartograph/SKILL.md:12-17`; `skills/landscape/SKILL.md:12-16`;
  `skills/migrate/SKILL.md:12-16`.

**Note**: distinct from F1 — F1 is a *wrong* namespace (`/skeptic:`), this is the *bare* form.

---

## F8 — [MINOR] Stale / inconsistently-pinned model IDs

Two skills pin exact model IDs in prose that (a) disagree with the actual dispatch (which
uses the generic `model: "opus"` / `"sonnet"` aliases everywhere), and (b) name
`claude-opus-4-6` / `claude-sonnet-4-5`, which are superseded (current is Opus 4.8). The
other four skills pin no IDs, so the roster prose is inconsistent across skills too.

- `skills/skeptic/SKILL.md:38` — "Opus (claude-opus-4-6)".
- `skills/cartograph/SKILL.md:31` — "Opus (claude-opus-4-6)"; `:32` — "Sonnet (claude-sonnet-4-5)".
- Actual dispatch uses aliases only: e.g. `skills/skeptic/SKILL.md:121-122`,
  `skills/cartograph/SKILL.md:363`.
- recon/tribunal/landscape/migrate name no IDs (e.g. `skills/tribunal/SKILL.md:38`).

---

## F9 — [MINOR] Recon "no report" error text differs between Step 1 and its failure table

The same failure has two different user-facing messages within one file.

- `skills/recon/SKILL.md:24` (Step 1) — "Run `/skeptic` first. Recon needs concept agent findings."
- `skills/recon/SKILL.md:425` (Failure Modes) — "No skeptic report found. Run `/skeptic` first."

---

## F10 — [MINOR] Cartograph Domain supplement mislabels its own dimension

The Domain agent's supplement opens with a phrase copy-pasted from a data-flow context —
it tells the Domain agent to act "Before mapping data flow", though this agent maps the
domain model, not data flow.

- `skills/cartograph/SKILL.md:153` — "Before mapping data flow, identify the core domain model:".
- Contradicts the agent's own identity: `skills/cartograph/SKILL.md:147` ("The Domain
  agent receives...") and roster `:27` (Domain lens = business logic/entities/workflows).

---

## F11 — [MINOR] Tribunal synthesis prompt says "two jobs" but defines four

The synthesis prompt announces two jobs, then enumerates Job 1–Job 4.

- `skills/tribunal/SKILL.md:102` — "**You have two jobs:**".
- Actual jobs: `:104` Job 1 (Comparison Matrix), `:118` Job 2 (Agreement & Divergence),
  `:125` Job 3 (Risk Pre-Mortem), `:131` Job 4 (Recommendation).

---

## F12 — [NITPICK] Miscellaneous structural / wording drifts

- **Duplicate step number**: `skills/skeptic/SKILL.md:785` and `:786` are both numbered `7.`
  ("Write output" and "Offer design doc persistence") inside plan Common Behavior.
- **Empty section**: `skills/recon/SKILL.md:419` — "### Step 10: Teardown" has no body.
- **Tribunal error-wording variants** across three locations for the option-count check:
  `skills/tribunal/SKILL.md:24` ("Tribunal requires 2-4 options. You provided X."),
  `:236-237` ("...at least 2 options. Please provide more." / "...maximum 4 options. You
  provided X. Pick the 4 most important."), `:358-359` ("...at least 2 options." / "...maximum
  4 options. Please pick 4."). Same semantics, three phrasings.

---

## Summary table

| ID | Sev | Area | One-line |
|----|-----|------|----------|
| F1 | MAJOR | namespace | recon documents `/skeptic:recon`; should be `/dispatch:recon` (README self-contradicts) |
| F2 | MAJOR | output path | skeptic globs `.landscape/report-*.md`; landscape writes `survey-*.md` — never matches |
| F3 | MAJOR | output path | README says all skills write `docs/designs/`; recon has no such step |
| F4 | MAJOR | model roster | skeptic Step 3 model-assignment omits the Testing agent |
| F5 | MAJOR | permissions | skeptic "only writes TODO/roadmap outside .skeptic/" but also writes docs/designs/ + CLAUDE.md |
| F6 | MINOR | commands | skeptic Invocation plan-type list drops `concept` |
| F7 | MINOR | namespace | bare `/skeptic`… vs CLAUDE.md `/dispatch:` convention (all skills + README) |
| F8 | MINOR | model roster | stale/inconsistent pinned model IDs (claude-opus-4-6, claude-sonnet-4-5) |
| F9 | MINOR | error table | recon "no report" message differs Step 1 vs failure table |
| F10 | MINOR | model roster | cartograph Domain supplement says "mapping data flow" (wrong dimension) |
| F11 | MINOR | error/structure | tribunal synthesis says "two jobs" but defines four |
| F12 | NITPICK | structural | duplicate Step 7, empty Teardown, tribunal error-wording variants |

**Verified clean** (no inconsistency found): tribunal output path `.tribunal/decision-*`
(README ↔ SKILL); cartograph output `docs/cartograph.md` (README:117 ↔ SKILL:272); migrate
output `docs/migration-*` (README:153 ↔ SKILL:286); skeptic report path `.skeptic/report-*`
(README:63 ↔ SKILL:494); agent counts (8 skeptic, 5 cartograph, 5 migrate, 4 landscape) match
across README/CLAUDE/SKILL; migrate & cartograph model rosters match README; agent tool-grant
statements internally consistent for cartograph, tribunal, landscape, recon.
