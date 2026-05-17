# Dispatch

Multi-agent analysis and planning toolkit for Claude Code.

Fan-out parallel specialists for adversarial review, competitive research, decision analysis, codebase mapping, domain surveys, and migration planning.

**Philosophy**: Structured parallelism. One skill per analysis type. Expertise through specialization.

## Installation

```bash
claude plugins add-marketplace KwanwooLee36/dispatch
claude plugins install dispatch
```

## Skill Overview

| Skill | Purpose | Specialists | Key Output |
|-------|---------|-------------|-----------|
| **skeptic** | Adversarial review | 8 domain critics + synthesis | Scored report, findings by severity, improvement plan |
| **recon** | Competitive deep-dive | Per-competitor analysts | Gap matrix, differentiators, prioritized backlog |
| **tribunal** | Decision analysis | Per-option advocates + synthesis | Comparison matrix, recommendation with confidence |
| **cartograph** | Codebase mapping | 5 dimension specialists | Developer guide with architecture, data flow, conventions |
| **landscape** | Domain research | 4 research dimensions | Landscape map, solutions catalog, opportunity ranking |
| **migrate** | Migration planning | 5 risk specialists + synthesis | Go/no-go, risk matrix, effort estimate |

---

## Skeptic

Adversarial multi-agent review. Every specialist is biased against the project and will find flaws within their domain. Maximum hostility. Exposure first, optional fixes second.

```
/skeptic          # Interactive menu — choose agents
/skeptic full     # All 8 agents — maximum coverage
/skeptic quick    # All 8 agents — capped exploration for speed
/skeptic fix      # Auto-fix Actionable Now items from latest report
/skeptic arch     # Architecture only
/skeptic arch code security  # Multiple specific agents
/skeptic plan     # Strategic improvement plan from findings
/skeptic plan <type>  # Category plan: arch|design|code|security|perf|dx|test|debt
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

### Output

- **Console**: Verdict, scores (0-100 per category), finding counts by severity
- **Report**: `.skeptic/report-YYYY-MM-DD.md` with full findings, repeat offenders, trends
- **Severity levels**: FATAL, MAJOR, MINOR, NITPICK
- **Plan**: `/skeptic plan` reads latest report, produces strategic improvement plan with neutral arbiter framing

### History & Repeat Offenders

Reports accumulate in `.skeptic/`. Synthesis agent detects issues flagged in prior reviews, marks as **REPEAT OFFENDER**, escalates severity after 3+ appearances, calculates score trends.

---

## Recon

Competitive deep-dive. Reads competitor names from a prior `/skeptic` run, researches each in depth, synthesizes into gap analysis, differentiator map, and actionable backlog items with design suggestions.

```
/skeptic:recon         # Full mode — one Opus agent per competitor, parallel
/skeptic:recon quick   # Quick mode — single Sonnet agent, all competitors sequential
```

### Input

Reads `.skeptic/report-*.md` (most recent). Extracts competitor list from Concept & Strategy findings. Errors if report missing or no competitors identified.

### What It Produces

- **Per-competitor deep dives**: Overview, strengths, weaknesses, feature inventory
- **Gap matrix**: Features competitors have that you don't, tagged STEAL/CONSIDER/IRRELEVANT
- **Differentiator analysis**: Existing moat, potential moat, table stakes
- **Prioritized recommendations**: Ranked list of what to build, steal, or ignore
- **Design notes**: Per-gap design docs written to `.skeptic/recon-designs/`

---

## Tribunal

Structured decision analysis via competing advocates. Each advocate argues FOR one option with evidence. Synthesis agent identifies table stakes, conflicts, and recommends the strongest path forward.

```
/tribunal "Postgres vs DynamoDB"
/tribunal "monolith vs microservices vs modular monolith"
/tribunal                              # Interactive prompt for decision and options
```

### Input

Requires 2-4 options. Project context (codebase) is mandatory — decisions are grounded in reality, not abstract.

### What It Produces

- **Per-option advocate reports**: Case summary, evidence, counterargument preemption, closing argument
- **Comparison matrix**: Overlaps, conflicts, table stakes across options
- **Recommendation**: Strongest path forward with confidence assessment

---

## Cartograph

Parallel codebase mapper and developer guide generator. Five dimension specialists analyze structure, data flow, conventions, infrastructure, and domain logic.

```
/cartograph          # Full analysis, all dimensions
/cartograph quick    # Quick scan (all Sonnet, 15-file cap)
/cartograph focus:auth          # Focus all dimensions on auth subsystem
/cartograph focus:api quick     # Focus + quick mode
```

### Dimensions

| Dimension | Lens | Model |
|-----------|------|-------|
| **Structure** | Module boundaries, entry points, dependency graph | Opus |
| **Data Flow** | APIs, state, storage, transformations, middleware chains | Opus |
| **Conventions** | Naming patterns, style, idioms, error handling, testing | Sonnet |
| **Infrastructure** | Build, deploy, CI/CD, containers, external services | Sonnet |
| **Domain** | Business logic, key entities, workflows, domain vocabulary | Sonnet |

### Output

Single unified guide: `docs/cartograph.md` with architecture map, data flow narrative, conventions reference, infrastructure guide, and domain glossary.

---

## Landscape

Domain and market research survey. Four parallel research agents map solutions, approaches, community insights, and business dynamics. Works standalone — no codebase dependency.

```
/landscape "real-time collaboration tools"     # Full mode — 4 agents, broad research
/landscape quick "state management"             # Quick mode — focused research
/landscape                                      # Interactive — prompts for domain
```

### Input

Domain string (2-10 words): "state management", "real-time collaboration tools", "database migration".

### Research Dimensions

- **Solutions**: Existing tools, products, libraries, SaaS, frameworks
- **Approaches**: Technical patterns, architectural styles, design patterns, research papers
- **Community**: Developer discussions, forums, GitHub activity, trends, sentiment
- **Business**: Funding, company dynamics, market consolidation, sustainability signals

### Output

- **Landscape map**: Solution catalog with positioning, features, pricing, maturity signals
- **Comparison matrix**: Approaches ranked by adoption and innovation
- **Opportunity ranking**: Gaps, emerging patterns, business trends

---

## Migrate

Multi-dimensional migration risk analysis. Five parallel specialists investigate API compatibility, dependencies, tests, infrastructure, and data layer. Cross-cutting landmine detection with go/no-go recommendation.

```
/migrate "Express to Fastify"          # Full analysis
/migrate quick "webpack to vite"        # Quick mode (Sonnet agents)
/migrate                                 # Interactive prompt for migration intent
```

### Dimensions

| Dimension | Model | Investigates |
|-----------|-------|-------------|
| **API Surface** | Opus | Compatibility, transformation rules, breaking changes |
| **Dependencies** | Sonnet | Package ecosystem, replacement research, version alignment |
| **Tests** | Sonnet | Framework migration, coverage gaps, effort estimate |
| **Infrastructure** | Sonnet | CI/CD, deployment, monitoring changes |
| **Data** | Opus | Schema, ORM, data access patterns, rollback strategy |

### Output

- **Console**: Go/no-go verdict, top 5 risks, effort estimate
- **Plan**: `docs/migration-YYYY-MM-DD-{slug}.md` with ranked migration path, risk matrix, cross-cutting landmines
- **Todo**: Auto-generated backlog items (user confirms before adding)

---

## Design Doc Persistence

All skills support persistent design docs in `docs/designs/`:

- **Skeptic plans** → `docs/designs/skeptic-plan-*.md`
- **Recon gaps** → `docs/designs/recon-{gap}-*.md`
- **Tribunal decisions** → `docs/designs/tribunal-decision-*.md`
- **Landscape insights** → `docs/designs/landscape-*.md`
- **Migration plans** → `docs/designs/migration-*.md`

Design docs auto-inject into CLAUDE.md for cross-session context. Session memory preserves analysis state across tool invocations.

---

## Kerd Integration

Works standalone, but integrates with the [Kerd](https://github.com/KwanwooLee36/kerd) ecosystem when available:

- **dian**: Reviews and analyses noted in session logs
- **kivna**: Design docs and findings persist to vault for cross-session access
- **slainte**: Complementary — slainte reports facts; skeptic argues against you

---

## License

MIT
