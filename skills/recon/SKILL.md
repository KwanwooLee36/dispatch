---
name: recon
description: Use when the user says 'recon', 'competitor research', 'market research', 'competitive analysis', 'compare competitors', or wants deep competitive analysis building on skeptic's concept agent findings. Reads competitors from latest skeptic report, researches each in depth, produces gap matrix and differentiator analysis with design suggestions.
---

# Recon

Competitive deep-dive. Reads competitor names from a prior `/skeptic` run, researches each competitor in depth, synthesizes into gap analysis + differentiator map + actionable backlog items with design suggestions.

## Invocation

```
/skeptic:recon         # Full mode — one Opus agent per competitor, parallel
/skeptic:recon quick   # Quick mode — single Sonnet agent, all competitors sequential
```

## Input

**Target project:** The project where `/skeptic:recon` is invoked (cwd). The skeptic report must be from the same project.

### Step 1: Find Skeptic Report

Glob for `.skeptic/report-*.md` in cwd. Select the most recent by filename date. If none exists:

> **Error:** "Run `/skeptic` first. Recon needs concept agent findings."

### Step 2: Validate Project Identity

Read the target project's README (first heading or title). Compare against the report content. If the report references a different project name:

> **Error:** "Skeptic report appears to be from a different project. Run `/skeptic` in this project first."

### Step 3: Extract Competitor List

Parse the Concept & Strategy section of the report. Competitors appear in Market Saturation findings and throughout concept findings as named alternatives, incumbents, or competing tools/libraries.

**Extraction logic:** Scan for named products/tools/libraries mentioned as competitors, incumbents, or alternatives. For each, capture:
- **Name** (as written in the report)
- **Reference sentence** (why it was flagged or how it relates)

If no Concept & Strategy section exists, or findings contain no named alternatives:

> **Error:** "Could not extract competitor list from skeptic report. Ensure `/skeptic` was run with the concept agent included."

If competitors extracted successfully, print:

```
Competitors found: [Name1], [Name2], [Name3], ...
Proceeding with recon.
```

## Per-Competitor Research Agent

### Model Strategy

| Mode | Model | Dispatch |
|------|-------|----------|
| Full | opus | One agent per competitor, parallel (max 10 concurrent) |
| Quick | sonnet | Single agent, all competitors sequential |

**Full mode**: Dispatch one Opus agent per competitor in parallel via Agent tool. If >10 competitors, batch in waves of 10 (sequential waves, parallel within each wave). Log batch progress.

**Quick mode**: Dispatch one Sonnet agent that researches all competitors sequentially. Produces N individual per-competitor reports using the same schema as full mode, so synthesis works identically regardless of mode.

### Tools

- **WebSearch** — discovery (find competitor URLs, reviews, comparisons)
- **WebFetch** — deep reads of product pages, docs, pricing pages, GitHub READMEs
- **Read, Glob, Grep** — read target project for comparison context
- **No Bash** — research only, no execution

### Search Strategy

- **Full mode**: 5-10 WebSearches per competitor. Product page, docs, GitHub (if open source), reviews/comparisons, pricing page.
- **Quick mode**: 3-5 WebSearches total across all competitors.
- Use WebFetch to read discovered URLs.

### Agent Prompt Template

Each competitor agent receives this prompt (fill in `{COMPETITOR_NAME}`, `{COMPETITOR_CONTEXT}`, `{PROJECT_IDENTITY}`):

---

**BEGIN COMPETITOR AGENT PROMPT**

You are a competitive research analyst. Your job is to deeply research {COMPETITOR_NAME} and produce a comprehensive analysis comparing it against our project.

**Context from prior skeptic review:** {COMPETITOR_CONTEXT}

**Our project identity:** {PROJECT_IDENTITY}

**Research protocol:**

1. **Discover**: Use WebSearch to find {COMPETITOR_NAME}'s product page, documentation, GitHub repo (if open source), pricing page, and user reviews/comparisons.
2. **Deep read**: Use WebFetch to read key pages in full. Don't rely on search snippets alone.
3. **Compare**: Read our project (cwd) via Read/Glob/Grep to understand what we offer, so you can identify gaps accurately.

**Output this exact format:**

```
## {COMPETITOR_NAME}

### Overview
What it is, who it's for, pricing model, maturity/traction signals.

### Strengths
Best qualities — what they do well, what users praise. Be concrete and specific, not vague ("real-time collaboration with conflict resolution" not "good UX").

### Weaknesses
Where they fall short, common complaints, missing capabilities.

### Feature Inventory
| Feature | Description |
|---------|-------------|
| Feature name | Brief description of what it does |
(list all key features)

### Gap Analysis vs Our Project
Features/qualities they have that we don't. Tag each:

- **[STEAL]** Feature name — clearly valuable, we should incorporate. Why: ...
- **[CONSIDER]** Feature name — interesting but may not fit our audience/scope. Why: ...
- **[IRRELEVANT]** Feature name — they have it but it doesn't serve our use case. Why: ...
```

**Rules:**
1. Be thorough. Read actual docs, don't guess from landing page copy.
2. Be honest. If a competitor genuinely does something better, say so clearly.
3. Be specific. Name actual features, cite actual capabilities. No filler.
4. If you cannot find information about the competitor (product shut down, behind paywall, etc.), report what you could find and flag gaps in your research.
5. If WebFetch fails for a URL, note it in your report and continue with partial information.

**END COMPETITOR AGENT PROMPT**

---

## Synthesis Agent

After all competitor agents return, spawn one synthesis agent (`model: "opus"`, always — even in quick mode). Synthesis receives: (1) all per-competitor reports, (2) target project README + main docs (glob `docs/`, `doc/`, architecture files) capped at 50KB. If target project is large, prioritize: README > architecture docs > feature docs.

### Synthesis Agent Prompt

---

**BEGIN RECON SYNTHESIS PROMPT**

You are the Lead Analyst. You have received deep-dive research reports on competitors for a project. Your job is to synthesize these into actionable intelligence.

**You have two jobs:**

#### Job 1: Inbound Gap Matrix

Merge all gap analysis entries from all competitor reports into one unified matrix. Deduplicate: if multiple competitors have the same feature gap, merge into one row listing all competitors that have it.

**Priority logic:**
- **HIGH** — multiple competitors have it AND high user value
- **MED** — one competitor has it, or niche value
- **LOW** — nice-to-have, edge case

For each gap, write a **design suggestion**: 2-3 sentences on how this feature could be implemented in our project given our stack and architecture. Read our project (README, docs, source) to make suggestions grounded in reality.

**Output format:**

```
## Inbound Gap Matrix

| Gap | Competitor(s) | Tag | Priority | Design Suggestion |
|-----|---------------|-----|----------|-------------------|
| Feature name | Comp A, Comp C | STEAL | HIGH | How to implement... |
```

#### Job 2: Differentiator Analysis

Read our project thoroughly (README, docs, source code). Cross-reference our features against ALL competitor feature inventories.

Produce three lists:

1. **Existing Moat**: Features we have that ZERO competitors offer. For each: what it is and why it matters as a differentiator.
2. **Potential Moat**: Features we could build that NO competitor has — blue ocean opportunities identified from gaps across the entire competitive landscape. For each: what it would be, feasibility estimate (easy/medium/hard), and why it would differentiate us.
3. **Table Stakes**: Features we have that competitors also have. These are parity, not differentiators.

#### Final Output

After the two jobs, produce a **Prioritized Recommendations** section: a ranked list of what to build, what to steal, and what to ignore. Rank by impact × feasibility.

**Full output format:**

```
## Inbound Gap Matrix
[Table from Job 1]

## Differentiator Analysis

### Existing Moat
- **Feature name** — what it is, why it matters as differentiator

### Potential Moat
- **Feature name** — what it would be (feasibility: easy/medium/hard), why it differentiates

### Table Stakes
- **Feature name** — parity with [competitors]

## Prioritized Recommendations
1. **[STEAL/BUILD/IGNORE]** Feature name — reasoning (impact: high/med/low, feasibility: easy/med/hard)
```

**Rules:**
1. Every gap in the matrix needs a real design suggestion, not a placeholder.
2. Existing Moat must be verified — actually check that no competitor has it.
3. Potential Moat ideas must be grounded — don't suggest things wildly outside our stack's capability.
4. Prioritized Recommendations must be opinionated. Rank them. Don't hedge.

**END RECON SYNTHESIS PROMPT**

---

## Execution Flow

### Step 3: Read Target Project Identity

Read the target project to build a project identity summary for competitor agents. Same approach as skeptic's concept agent:

1. Read README — what the project claims to do, target audience, problem solved
2. Glob for `docs/`, `doc/` — read positioning/architecture docs
3. Read `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod` — description fields, keywords
4. Glob for `PITCH.md`, `ABOUT.md`, landing pages

Compile into a 1-2 paragraph project identity summary passed to each competitor agent as `{PROJECT_IDENTITY}`.

### Step 4: Dispatch Competitor Agents

**Full mode:**

Print: "Dispatching {N} competitor research agents (Opus)..."

Spawn one Agent per competitor in parallel:

```
Agent({
  description: "recon: research {COMPETITOR_NAME}",
  model: "opus",
  prompt: "... competitor agent prompt with {COMPETITOR_NAME}, {COMPETITOR_CONTEXT}, {PROJECT_IDENTITY} filled in ..."
})
```

If >10 competitors, batch in waves of 10. Print between waves: "Wave {X}/{Y}: dispatching next 10 competitors..."

**Quick mode:**

Print: "⚡ Quick mode: single Sonnet agent researching all {N} competitors..."

Spawn one Agent:

```
Agent({
  description: "recon: research all competitors (quick)",
  model: "sonnet",
  prompt: "... competitor agent prompt with ALL competitors listed, same output schema per competitor ..."
})
```

Add to quick mode prompt: "TOKEN BUDGET: Limit to 3-5 WebSearches total across all competitors. Prioritize breadth over depth. Focus on feature inventories and gaps."

### Step 5: Collect and Synthesize

After all competitor agents return, spawn the synthesis agent:

```
Agent({
  description: "recon: synthesize competitive intelligence",
  model: "opus",
  prompt: "... synthesis prompt with all competitor reports concatenated + project docs (capped at 50KB) ..."
})
```

If any competitor agent failed or timed out, pass partial results to synthesis. Note which competitors have incomplete research.

### Step 6: Write Report

Write the full synthesis output to `.skeptic/recon-YYYY-MM-DD.md`. If a recon report already exists for today, append counter: `recon-YYYY-MM-DD-2.md`, etc. Use Glob to find highest existing counter.

Create `.skeptic/` directory if it doesn't exist.

**Report format:**

```markdown
# Recon Report — YYYY-MM-DD

## Competitors Analyzed
- **Competitor Name** — one-line description
(repeated for each)

## Per-Competitor Deep Dives

### [Competitor Name]
**Overview:** ...
**Strengths:** ...
**Weaknesses:** ...
**Feature Inventory:** ...
**Gaps vs Us:** ...

(repeated per competitor — copied from individual agent reports)

## Inbound Gap Matrix
| Gap | Competitor(s) | Tag | Priority | Design Suggestion |
|-----|---------------|-----|----------|-------------------|
(from synthesis)

## Differentiator Analysis

### Existing Moat
(from synthesis)

### Potential Moat
(from synthesis)

### Table Stakes
(from synthesis)

## Prioritized Recommendations
(from synthesis)
```

### Step 7: Persist Design Notes

For each gap tagged **STEAL** in the Inbound Gap Matrix, write a design note to `.skeptic/recon-designs/[gap-name].md`.

Create `.skeptic/recon-designs/` directory if it doesn't exist.

**Design note format:**

```markdown
# [Gap Name]

**Source competitors:** Comp A, Comp C
**Priority:** HIGH
**Tag:** STEAL

## What It Is
[Description of the feature from competitor analysis]

## How Competitors Implement It
[Summary from competitor deep dives]

## How It Could Work In Our Project
[Design suggestion from synthesis, expanded with project-specific context]
```

Filename: slugified gap name (e.g., `real-time-collab.md`). Link each design note from the main report.

### Step 8: Persist Backlog Items

Auto-append to target project TODO.md under `### Recon Gaps — YYYY-MM-DD`.

**Items sourced from:** Inbound Gap Matrix rows tagged STEAL or CONSIDER with HIGH or MED priority.

**Format per item:**
```
- [TAG] Gap name — design suggestion (one sentence). See: .skeptic/recon-designs/[gap].md
```

**Deduplication:** Check existing TODO.md entries using >70% title-only word overlap. Items with identical titles but different design suggestions are considered duplicates — existing entry kept.

If no TODO.md exists: create with `# TODO\n\n## Backlog\n` header.
If TODO.md exists but has no `## Backlog` section: append one.

Print: "Added {N} items to TODO.md backlog ({M} duplicates skipped)." or "All gaps already in backlog. Nothing added."

### Step 9: Console Summary

Print:

```
═══════════════════════════════════════════════
  RECON — YYYY-MM-DD
═══════════════════════════════════════════════

  Competitors analyzed: N

  GAPS:
    STEAL:      X (HIGH: H, MED: M, LOW: L)
    CONSIDER:   X
    IRRELEVANT: X

  DIFFERENTIATORS:
    Existing moat:  X features no competitor has
    Potential moat:  X blue ocean opportunities
    Table stakes:   X parity features

  PERSISTENCE:
    Backlog items:  X added to TODO.md (Y deduped)
    Design notes:   X written to .skeptic/recon-designs/

  Full report: .skeptic/recon-YYYY-MM-DD.md
═══════════════════════════════════════════════
```

### Step 10: Teardown

## Failure Modes

| Failure | Behavior |
|---------|----------|
| No skeptic report exists | Error: "No skeptic report found. Run `/skeptic` first." |
| Report from different project | Error: "Skeptic report appears to be from a different project. Run `/skeptic` in this project first." |
| No Concept & Strategy section in report | Error: "Could not extract competitor list from skeptic report. Ensure `/skeptic` was run with the concept agent included." |
| No competitors found in report | Warn: "Concept agent found no competitors. Market may be genuinely open, or run skeptic with broader scope." |
| WebSearch unavailable | Degrade to WebFetch only. Note reduced discovery in console summary |
| Individual competitor agent fails | Report which competitor couldn't be researched. Continue with others. Synthesis flags as "incomplete research" |
| Competitor agent times out mid-research | Synthesis receives partial report. Flags competitor as "incomplete research" in report |
| All competitor agents fail | Error: "Recon could not research any competitors. Check network/tool availability." |
| No web tools available (WebSearch, WebFetch both unavailable) | Error: "Recon requires web access tools. Ensure WebSearch or WebFetch is available." |
| .skeptic/ directory doesn't exist | Create it |
| TODO.md write fails | Print backlog items to console only. Warn: "Could not write to TODO.md — items printed to console only" |

## Edge Cases

- **One competitor only**: Run normally. Gap matrix and differentiator analysis still valuable with single comparison point.
- **>10 competitors**: Batch in waves of 10. Log progress between waves.
- **Competitor is open source**: Agent should check GitHub for additional detail (stars, recent activity, contributor count, issue tracker).
- **Competitor product shut down or unreachable**: Agent reports what it could find and flags gaps. Synthesis notes incomplete coverage.
- **Our project has no README or docs**: Project identity summary will be minimal. Agents may produce less accurate gap analysis. Warn: "Limited project docs found — gap analysis may be less accurate."
