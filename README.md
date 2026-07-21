# Dispatch

Multi-agent analysis and planning toolkit for Claude Code. Each skill fans out parallel specialists, collects their reports, and synthesizes a unified result.

## Installation

```bash
claude plugins add-marketplace KwanwooLee36/dispatch
claude plugins install dispatch
```

## Skills

| Skill | Purpose | Specialists | Key Output |
|-------|---------|-------------|-----------|
| **skeptic** | Adversarial review | 8 domain critics + synthesis | Scored report, findings by severity, improvement plan |
| **recon** | Competitive deep-dive | Per-competitor analysts | Gap matrix, differentiators, prioritized backlog |
| **tribunal** | Decision analysis | Per-option advocates + synthesis | Comparison matrix, recommendation with confidence |
| **cartograph** | Codebase mapping | 5 dimension specialists | Developer guide with architecture, data flow, conventions |
| **landscape** | Domain research | 4 research dimensions | Solutions catalog, opportunity ranking |
| **migrate** | Migration planning | 5 risk specialists + synthesis | Go/no-go, risk matrix, effort estimate |

---

## Skeptic

Every specialist is biased against the project. They hunt for flaws within their domain, argue the project is broken, and provide no fixes. Synthesis merges their reports, scores the project 0-100 per category, and triages findings.

```
/dispatch:skeptic                     # Interactive menu — choose agents
/dispatch:skeptic full                # All 8 agents — maximum coverage
/dispatch:skeptic quick               # All 8 agents — capped exploration for speed
/dispatch:skeptic fix                 # Auto-fix Actionable Now items from latest report
/dispatch:skeptic arch                # Architecture only
/dispatch:skeptic arch code security  # Multiple specific agents
/dispatch:skeptic plan                # Strategic improvement plan from findings
/dispatch:skeptic plan <type>         # Category plan: arch|design|code|security|perf|dx|test|concept|debt
```

### Agents

| Agent | Hunts For | Model |
|-------|-----------|-------|
| **Architecture** | Coupling, circular deps, wrong abstractions, scaling walls | Opus |
| **Design** | Leaky abstractions, pattern misuse, unclear intent, premature generalization | Opus |
| **Code Quality** | Deprecated deps, dead code, complexity, duplication, naming | Sonnet |
| **Security** | Injection, auth bypass, secrets, insecure defaults, OWASP top 10 | Opus |
| **Performance** | N+1 queries, allocations, missing caching, resource leaks | Sonnet |
| **DX/Ergonomics** | Confusing APIs, missing docs, bad error messages, onboarding friction, broken UX tooling | Sonnet |
| **Concept & Strategy** | Value prop, market fit, positioning, feasibility, differentiation | Opus |
| **Testing** | Test quality, coverage gaps, meaningless tests, flaky patterns, missing test types | Sonnet |

### Inline Recon

When the Concept & Strategy agent identifies competitors, skeptic prompts you: "Run recon?" If you say yes, competitive research agents dispatch immediately using the same research protocol as `/dispatch:recon`. Recon findings feed into synthesis, generating new findings from competitive gaps, missing moats, and positioning failures alongside the specialist critiques, and are written to `.skeptic/recon-YYYY-MM-DD.md` so a later `/dispatch:skeptic plan` can load them. Standalone `/dispatch:recon` additionally produces design notes in `.skeptic/recon-designs/`; the inline path does not.

### Plan with Competitive Intelligence

`/dispatch:skeptic plan` reads existing recon (`.skeptic/recon-*.md`) and landscape (`.landscape/survey-*.md`) reports when they exist. Plan agents reference competitor approaches, flag table-stakes gaps, and note moat opportunities. If no reports exist, it proceeds without them.

### Output

Reports land in `.skeptic/report-YYYY-MM-DD.md`. Console prints the verdict, scores (0-100 per category), and finding counts. Severity levels: FATAL, MAJOR, MINOR, NITPICK. Future Work items route to `docs/roadmap.md` milestones when a roadmap exists, falling back to `TODO.md` backlog otherwise.

### History

Reports accumulate in `.skeptic/`. Synthesis detects issues flagged in prior reviews, marks them as repeat offenders, escalates severity after 3+ appearances, and calculates score trends.

---

## Recon

Reads competitor names from a prior `/dispatch:skeptic` run, researches each in depth, and synthesizes into a gap matrix, differentiator map, and backlog items with design suggestions.

```
/dispatch:recon        # Full mode — one Opus agent per competitor, parallel
/dispatch:recon quick  # Quick mode — single Sonnet agent, all competitors sequential
```

Requires a skeptic report with Concept & Strategy findings that name competitors. Produces per-competitor deep dives (overview, strengths, weaknesses, feature inventory), a gap matrix tagged STEAL/CONSIDER/IRRELEVANT, differentiator analysis (existing moat, potential moat, table stakes), and prioritized recommendations. Design notes go to `.skeptic/recon-designs/`. Gap items route to `docs/roadmap.md` milestones when a roadmap exists, falling back to `TODO.md` backlog otherwise.

---

## Tribunal

Each advocate argues FOR one option with evidence. Synthesis identifies table stakes, conflicts, and recommends the strongest path forward.

```
/dispatch:tribunal "Postgres vs DynamoDB"
/dispatch:tribunal "monolith vs microservices vs modular monolith"
/dispatch:tribunal                              # Interactive prompt for decision and options
```

Requires 2-4 options. Project context (codebase) is mandatory so decisions stay grounded. Produces per-option advocate reports, a comparison matrix, and a recommendation with confidence assessment.

---

## Cartograph

Five dimension specialists analyze structure, data flow, conventions, infrastructure, and domain logic in parallel. Produces a single unified developer guide.

```
/dispatch:cartograph                  # Full analysis, all dimensions
/dispatch:cartograph quick            # Quick scan (all Sonnet, 15-file cap)
/dispatch:cartograph focus:auth       # Focus all dimensions on auth subsystem
/dispatch:cartograph focus:api quick  # Focus + quick mode
```

| Dimension | Lens | Model |
|-----------|------|-------|
| **Structure** | Module boundaries, entry points, dependency graph | Opus |
| **Data Flow** | APIs, state, storage, transformations, middleware chains | Opus |
| **Conventions** | Naming patterns, style, idioms, error handling, testing | Sonnet |
| **Infrastructure** | Build, deploy, CI/CD, containers, external services | Sonnet |
| **Domain** | Business logic, key entities, workflows, domain vocabulary | Sonnet |

Output: `docs/cartograph.md`.

---

## Landscape

Four parallel research agents map solutions, approaches, community insights, and business dynamics for a given domain. No codebase dependency.

```
/dispatch:landscape "real-time collaboration tools"  # Full mode — 4 agents, broad research
/dispatch:landscape quick "state management"         # Quick mode — focused research
/dispatch:landscape                                  # Interactive — prompts for domain
```

Domain string is 2-10 words; whole-category inputs ("technology", "database") are rejected with narrower suggestions, and a longer-than-10-word domain warns rather than fails. Produces a solution catalog with positioning and pricing, approaches ranked by adoption, and an opportunity ranking of gaps and emerging patterns. Opportunity items route to `docs/roadmap.md` milestones when a roadmap exists, falling back to `TODO.md` backlog otherwise.

---

## Migrate

Five parallel specialists investigate API compatibility, dependencies, tests, infrastructure, and data layer. Cross-cutting landmine detection with a go/no-go recommendation.

```
/dispatch:migrate "Express to Fastify"     # Full analysis
/dispatch:migrate quick "webpack to vite"  # Quick mode (Sonnet agents)
/dispatch:migrate                          # Interactive prompt for migration intent
```

| Dimension | Model | Investigates |
|-----------|-------|-------------|
| **API Surface** | Opus | Compatibility, transformation rules, breaking changes |
| **Dependencies** | Sonnet | Package ecosystem, replacement research, version alignment |
| **Tests** | Sonnet | Framework migration, coverage gaps, effort estimate |
| **Infrastructure** | Sonnet | CI/CD, deployment, monitoring changes |
| **Data** | Opus | Schema, ORM, data access patterns, rollback strategy |

Console prints go/no-go verdict, top 5 risks, and effort estimate. Full plan goes to `docs/migration-YYYY-MM-DD-{slug}.md`. Migration steps route to `docs/roadmap.md` milestones when a roadmap exists, falling back to `TODO.md` backlog otherwise — always after you confirm.

---

## Design Doc Persistence

Five skills — **skeptic** (via `/dispatch:skeptic plan`), **cartograph**, **landscape**, **migrate**, and **tribunal** — can persist their primary output as a design doc in `docs/designs/`. On the first time that directory is created in a project, they append a `## Design Docs` orientation block to the project's `CLAUDE.md`, so future sessions (switch-in, dian, or any session start) read every file in `docs/designs/` and inherit the context.

**Recon is the exception — it does not use `docs/designs/` and does not auto-inject into `CLAUDE.md`.** Recon's design notes are fragmentary: one note per gap tagged **STEAL** in its Inbound Gap Matrix. It writes each to `.skeptic/recon-designs/[gap-name].md` and links them from the main recon report (`.skeptic/recon-YYYY-MM-DD.md`). This keeps recon's many small, competitive-intelligence fragments co-located with the skeptic/recon reports they derive from, rather than scattering them across `docs/designs/` and bloating the always-read `CLAUDE.md` auto-inject. Recon surfaces these notes through its report and its routed backlog/roadmap items instead.

---

## Kerd Integration

Works standalone. Integrates with [Kerd](https://github.com/KwanwooLee36/kerd) when available: reviews noted in session logs (dian), findings persist to vault (kivna), and skeptic complements slainte's factual audits with adversarial interpretation.

---

## License

MIT
