# Dispatch Consistency Audit — 2026-07-18

Cross-checked all 6 `skills/*/SKILL.md` against each other, `README.md`, and `CLAUDE.md`. Scope: slash-command references, model-roster claims, output file paths, error/failure tables, permission-grant statements. **Report only — nothing fixed.**

Files audited: `skills/{skeptic,recon,tribunal,cartograph,landscape,migrate}/SKILL.md`, `README.md`, `CLAUDE.md`, `.claude-plugin/{plugin,marketplace}.json`.

Ranked by severity. Each finding cites exact `file:line`.

---

## HIGH — broken command / silent-failure

### H1. Recon invocation uses wrong namespace `/skeptic:recon` (should be `/dispatch:recon`)
`recon` is its own skill (`skills/recon/SKILL.md:2` name: recon; registered `/dispatch:recon`). It is NOT a subcommand of skeptic, so `/skeptic:recon` is not a real command.

- `skills/recon/SKILL.md:13` — `/skeptic:recon         # Full mode`
- `skills/recon/SKILL.md:14` — `/skeptic:recon quick   # Quick mode`
- `skills/recon/SKILL.md:19` — "The project where `/skeptic:recon` is invoked (cwd)"
- `README.md:76` — `/skeptic:recon         # Full mode`
- `README.md:78` — `/skeptic:recon quick   # Quick mode`

Directly contradicted by:
- `CLAUDE.md:21` — "Skill references use `/dispatch:<skill>` namespace"
- `skills/skeptic/SKILL.md:369` — "Run `/dispatch:recon` separately later"
- `skills/skeptic/SKILL.md:1059` — "For deeper competitive research: /dispatch:recon"
- `README.md:55` — "using the same protocol as `/dispatch:recon`" (README contradicts *itself* — line 55 correct, lines 76/78 wrong)

### H2. Skeptic-plan globs `.landscape/report-*.md`, but landscape writes `.landscape/survey-*.md`
The competitive-intelligence loader will never match any landscape output — landscape data silently never loads into plan agents.

Landscape's actual output path:
- `skills/landscape/SKILL.md:324` — write node `.landscape/survey-YYYY-MM-DD-{slug}.md`
- `skills/landscape/SKILL.md:393` — "Write the full synthesis output to `.landscape/survey-{DATE}-{slug}.md`"
- `skills/landscape/SKILL.md:449` — console: `.landscape/survey-YYYY-MM-DD-{slug}.md`

Wrong pattern referenced by:
- `skills/skeptic/SKILL.md:780` — "Glob for `.skeptic/recon-*.md` and `.landscape/report-*.md`"
- `skills/skeptic/SKILL.md:781` — "Recon/landscape reports >14 days old…"
- `skills/skeptic/SKILL.md:957` — plan concept reads "landscape reports (`.landscape/report-*.md`)"
- `README.md:59` — "landscape (`.landscape/report-*.md`) reports when they exist"

(Recon's own pattern is fine: skeptic globs `.skeptic/recon-*.md` at `SKILL.md:780` and recon writes `.skeptic/recon-YYYY-MM-DD.md` at `skills/recon/SKILL.md:280` — matches.)

### H3. README "Design Doc Persistence" claims ALL skills write to `docs/designs/` — recon does not
- `README.md:157-159` — "All skills can persist their output as design docs in `docs/designs/`."

Recon has no `docs/designs/` persistence step. Its design output goes elsewhere:
- `skills/recon/SKILL.md:324-326` — Step 7 writes design notes to `.skeptic/recon-designs/[gap-name].md`
- `README.md:80` — README itself states recon "Design notes go to `.skeptic/recon-designs/`" (README line 80 contradicts README line 157-159)

Additionally, a plain skeptic *review* (`skills/skeptic/SKILL.md` Steps 1–7) never offers `docs/designs/` persistence — only skeptic *plan* does (`skills/skeptic/SKILL.md:786-815`). So "All skills … persist … as design docs" overstates for skeptic-review and is outright false for recon.

---

## MEDIUM — internal contradiction / missing spec

### M1. Skeptic Step 3 model-assignment list omits the Testing agent
The imperative dispatch list an implementer follows names no model for Testing:
- `skills/skeptic/SKILL.md:121-122` — `model: "opus"` → Architecture, Design, Security, Concept & Strategy; `model: "sonnet"` → Code Quality, Performance, DX. **Testing absent from both.**

Contradicts the roster and strategy which DO assign it Sonnet:
- `skills/skeptic/SKILL.md:34` — Testing … Model `sonnet`
- `skills/skeptic/SKILL.md:39` — "Sonnet: Code Quality, Performance, DX, Testing"
- `README.md:51` — Testing … Sonnet

### M2. Skeptic Invocation + fix-console omit `concept` from the plan subcommand list
- `skills/skeptic/SKILL.md:18` — `/skeptic plan <type>  # Category plan: arch|design|code|security|perf|dx|test|debt` (no `concept`)
- `skills/skeptic/SKILL.md:731` — fix console reprints `arch|design|code|security|perf|dx|test|debt` (no `concept`)

But `concept` is a real, documented plan subcommand everywhere else:
- `skills/skeptic/SKILL.md:762` — `/skeptic plan concept            # concept & strategy roadmap`
- `skills/skeptic/SKILL.md:955-971` — full "plan concept" section
- `README.md:37` — `/skeptic plan <type>  # Category plan: arch|design|code|security|perf|dx|test|concept|debt` (README correct; SKILL wrong)

### M3. Skeptic permission claim contradicts its own plan design-doc writes
- `skills/skeptic/SKILL.md:563` — "Write (TODO.md or docs/roadmap.md only — **these are the only places skeptic writes outside `.skeptic/`**)"

Contradicted by skeptic plan's design-doc persistence, which writes to `docs/designs/` and appends to `CLAUDE.md`:
- `skills/skeptic/SKILL.md:790` — "→ docs/designs/skeptic-plan-{type}-{slug}-YYYY-MM-DD.md"
- `skills/skeptic/SKILL.md:794` — "Copy the full plan to `docs/designs/…`"
- `skills/skeptic/SKILL.md:806-811` — appends `## Design Docs` block to project `CLAUDE.md`

---

## LOW / NITPICK — cosmetic, numbering, staleness

### L1. Stale/inconsistently-specified model IDs
Two skills pin outdated version IDs (current roster: Opus 4.8 = `claude-opus-4-8`, Sonnet 5 = `claude-sonnet-5`); the other four use bare "Opus"/"opus":
- `skills/skeptic/SKILL.md:38` — "Opus (claude-opus-4-6)"
- `skills/cartograph/SKILL.md:31` — "Opus (claude-opus-4-6)"
- `skills/cartograph/SKILL.md:32` — "Sonnet (claude-sonnet-4-5)"

Cosmetic only — all Agent calls pass bare `model: "opus"`/`"sonnet"`, which is the valid parameter form. But the parenthetical IDs are both stale and inconsistent with the other skills' unversioned style.

### L2. Recon step numbering is broken
- `skills/recon/SKILL.md:33` — "### Step 3: Extract Competitor List"
- `skills/recon/SKILL.md:219` — "### Step 3: Read Target Project Identity" (second, conflicting Step 3; execution-flow section restarts mid-sequence rather than continuing from Step 3)
- `skills/recon/SKILL.md:419` — "### Step 10: Teardown" is an empty header (no body before `## Failure Modes` at line 421)

### L3. Skeptic plan Common Behavior has duplicate step number "7"
- `skills/skeptic/SKILL.md:785` — "7. **Write output**"
- `skills/skeptic/SKILL.md:786` — "7. **Offer design doc persistence**" (should be 8)

### L4. Tribunal synthesis says "two jobs" but defines four
- `skills/tribunal/SKILL.md:102` — "**You have two jobs:**"
- `skills/tribunal/SKILL.md:104,117,125,131` — Job 1 (Comparison Matrix), Job 2 (Agreement & Divergence), Job 3 (Risk Pre-Mortem), Job 4 (Recommendation) — four jobs.

### L5. Cartograph Domain-agent supplement mislabels its own task
- `skills/cartograph/SKILL.md:154` — "Before mapping **data flow**, identify the core domain model:" — copy-paste from the Data Flow dimension; the Domain agent maps domain, not data flow.

### L6. Systemic: bare `/skeptic`, `/tribunal`, `/cartograph`, `/landscape`, `/migrate` vs CLAUDE.md's `/dispatch:` convention
`CLAUDE.md:21` states references "use `/dispatch:<skill>` namespace". Every SKILL invocation block and README section uses the bare form instead (e.g. `skills/skeptic/SKILL.md:13`, `skills/tribunal/SKILL.md:13`, `skills/cartograph/SKILL.md:13`, `skills/landscape/SKILL.md:13`, `skills/migrate/SKILL.md:13`, `README.md:30,89,103,126,139`). Consistent with each other and clearly intentional user-facing shorthand, so low-impact — but it does deviate from the stated convention. (Distinct from H1, where `/skeptic:recon` points at the *wrong* skill.)

---

## Checked and CONSISTENT (no finding)

- Skeptic agent-model roster (`SKILL.md:27-34`) vs README agents table (`README.md:44-51`) — match (8 agents, models agree).
- Cartograph dimension models (`SKILL.md:23-27`) vs README (`README.md:110-115`) and output `docs/cartograph.md` (`SKILL.md:272`, `README.md:117`) — match.
- Migrate dimension models (`SKILL.md:30-34`) vs README (`README.md:147-151`) and output `docs/migration-YYYY-MM-DD-{slug}.md` (`SKILL.md:286`, `README.md:153`) — match.
- Severity vocab FATAL/MAJOR/MINOR/NITPICK — `skeptic SKILL.md:149-152` vs `README.md:63` — match.
- Recon report path `.skeptic/recon-*.md` — recon write (`SKILL.md:280`) vs skeptic-plan glob (`SKILL.md:780`) vs `README.md:59` — match.
- Landscape "6 jobs" (`SKILL.md:192`), recon "2 jobs" (`SKILL.md:149`), migrate "3 jobs" (`SKILL.md:168`) — counts correct.
- Skills tables in `CLAUDE.md:7-15` and `README.md:14-21` — same 6 skills.
