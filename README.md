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
/skeptic          # Interactive menu — choose agents
/skeptic full     # All 8 agents — maximum coverage
/skeptic quick    # All 8 agents — capped exploration for speed
/skeptic fix      # Auto-fix Actionable Now items from latest report
/skeptic arch     # Architecture only
/skeptic arch code security  # Multiple specific agents
/skeptic plan     # Strategic improvement plan from findings
/skeptic plan <type>  # Category plan: arch|design|code|security|perf|dx|test|concept|debt
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

When the Concept & Strategy agent identifies competitors, skeptic prompts you: "Run recon?" If you say yes, competitive research agents dispatch immediately using the same protocol as `/dispatch:recon`. Recon findings feed into synthesis, generating new findings from competitive gaps, missing moats, and positioning failures alongside the specialist critiques.

### Plan with Competitive Intelligence

`/skeptic plan` reads existing recon (`.skeptic/recon-*.md`) and landscape (`.landscape/report-*.md`) reports when they exist. Plan agents reference competitor approaches, flag table-stakes gaps, and note moat opportunities. If no reports exist, it proceeds without them.

### Output

Reports land in `.skeptic/report-YYYY-MM-DD.md`. Console prints the verdict, scores (0-100 per category), and finding counts. Severity levels: FATAL, MAJOR, MINOR, NITPICK. Future Work items route to `docs/roadmap.md` milestones when a roadmap exists, falling back to `TODO.md` backlog otherwise.

### History

Reports accumulate in `.skeptic/`. Synthesis detects issues flagged in prior reviews, marks them as repeat offenders, escalates severity after 3+ appearances, and calculates score trends.

---

## Recon

Reads competitor names from a prior `/skeptic` run, researches each in depth, and synthesizes into a gap matrix, differentiator map, and backlog items with design suggestions.

```
/skeptic:recon         # Full mode — one Opus agent per competitor, parallel
/skeptic:recon quick   # Quick mode — single Sonnet agent, all competitors sequential
```

Requires a skeptic report with Concept & Strategy findings that name competitors. Produces per-competitor deep dives (overview, strengths, weaknesses, feature inventory), a gap matrix tagged STEAL/CONSIDER/IRRELEVANT, differentiator analysis (existing moat, potential moat, table stakes), and prioritized recommendations. Design notes go to `.skeptic/recon-designs/`. Gap items route to `docs/roadmap.md` milestones when a roadmap exists, falling back to `TODO.md` backlog otherwise.

---

## Tribunal

Each advocate argues FOR one option with evidence. Synthesis identifies table stakes, conflicts, and recommends the strongest path forward.

```
/tribunal "Postgres vs DynamoDB"
/tribunal "monolith vs microservices vs modular monolith"
/tribunal                              # Interactive prompt for decision and options
```

Requires 2-4 options. Project context (codebase) is mandatory so decisions stay grounded. Produces per-option advocate reports, a comparison matrix, and a recommendation with confidence assessment.

---

## Cartograph

Five dimension specialists analyze structure, data flow, conventions, infrastructure, and domain logic in parallel. Produces a single unified developer guide.

```
/cartograph          # Full analysis, all dimensions
/cartograph quick    # Quick scan (all Sonnet, 15-file cap)
/cartograph focus:auth          # Focus all dimensions on auth subsystem
/cartograph focus:api quick     # Focus + quick mode
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
/landscape "real-time collaboration tools"     # Full mode — 4 agents, broad research
/landscape quick "state management"             # Quick mode — focused research
/landscape                                      # Interactive — prompts for domain
```

Domain string is 2-10 words. Produces a solution catalog with positioning and pricing, approaches ranked by adoption, and an opportunity ranking of gaps and emerging patterns.

---

## Migrate

Five parallel specialists investigate API compatibility, dependencies, tests, infrastructure, and data layer. Cross-cutting landmine detection with a go/no-go recommendation.

```
/migrate "Express to Fastify"          # Full analysis
/migrate quick "webpack to vite"        # Quick mode (Sonnet agents)
/migrate                                 # Interactive prompt for migration intent
```

| Dimension | Model | Investigates |
|-----------|-------|-------------|
| **API Surface** | Opus | Compatibility, transformation rules, breaking changes |
| **Dependencies** | Sonnet | Package ecosystem, replacement research, version alignment |
| **Tests** | Sonnet | Framework migration, coverage gaps, effort estimate |
| **Infrastructure** | Sonnet | CI/CD, deployment, monitoring changes |
| **Data** | Opus | Schema, ORM, data access patterns, rollback strategy |

Console prints go/no-go verdict, top 5 risks, and effort estimate. Full plan goes to `docs/migration-YYYY-MM-DD-{slug}.md`.

---

## Design Doc Persistence

All skills can persist their output as design docs in `docs/designs/`. These auto-inject into CLAUDE.md so future sessions have the context.

---

## Kerd Integration

Works standalone. Integrates with [Kerd](https://github.com/KwanwooLee36/kerd) when available: reviews noted in session logs (dian), findings persist to vault (kivna), and skeptic complements slainte's factual audits with adversarial interpretation.

---

## License

MIT
