# Dispatch Skill-Behavior Verification — 2026-07-21

Behavioral trace of all six `skills/*/SKILL.md` files against themselves, `README.md`, and
`CLAUDE.md`. Unlike the two prior consistency audits (`.scorched/consistency-audit-2026-07-18.md`,
`.scorched/consistency-audit-2026-07-19.md`), which cross-checked *stated facts* (paths, rosters,
namespaces), this pass walks each **Step-by-step Invocation flow as an executor would** and asks:
can an agent follow this literally and produce the promised artifact?

**Scope traced per skill:** every numbered step and its ordering; every file-I/O path (read, write,
glob); every model / agent-roster claim; every tool-grant and permission statement; every example
command.

**Prior findings are NOT re-reported.** F1–F12 (2026-07-19) and H1–L6 (2026-07-18) are excluded;
their current status is summarised in §4 for reference only.

Findings are cited as `file:line` **plus a content excerpt**, because a later change may shift the
line numbers — match on the excerpt if the line no longer lines up.

---

## 1. Findings — executability

### B1 — [MAJOR] Landscape's "too broad" guidance is unreachable: every example it gives is rejected first

Landscape validates domain length **before** it checks breadth, and both of its worked broadness
examples are single words that the length check rejects. The broadness branch can therefore never
fire for the inputs it documents.

- `skills/landscape/SKILL.md:23` — *"Length: 2-10 words. Reject strings with <2 or >10 words."*
- `skills/landscape/SKILL.md:25` — *"Example: `❌ \"technology\"` → \"Too broad. Try: 'web framework', …\""* — `technology` is **1 word**.
- `skills/landscape/SKILL.md:348` — Step 1 ordering: *"If < 2 words: Error. \"Domain too short. Provide 2-10 words…\""*
- `skills/landscape/SKILL.md:352-353` — *"Example input: `\"technology\"` → \"❌ Too broad…\""* / *"Example input: `\"database\"` → \"❌ Too broad…\""* — both 1 word.
- `README.md:131` — *"Domain string is 2-10 words."* (repeats the length rule only)

**Impact**: the user-facing error for the most common misuse (a one-word domain) is the wrong one —
"too short, give me 2-10 words" instead of "too broad, try a narrower category". Fixing means either
allowing 1-word input into the broadness check or rewriting the examples as ≥2-word strings.

### B2 — [MAJOR] Landscape rejects vs. warns on >10 words, in the same file

Three statements of the same rule disagree on whether a long domain is fatal.

- `skills/landscape/SKILL.md:23` — *"**Reject** strings with <2 or >10 words."*
- `skills/landscape/SKILL.md:349` — *"If > 10 words: **Warn**. \"Domain string is long ({N} words). … Proceed? (y/n)\""*
- `skills/landscape/SKILL.md:514` — Failure Modes: *"Domain string too long (>10 words) | Warn and ask to narrow. If user declines, exit."*

**Impact**: an executor reading Input Parsing aborts; one reading Step 1 or the failure table
prompts and continues. Two of three say warn — line 23 is the outlier.

### B3 — [MAJOR] Migrate Step 8 writes the backlog before it offers to

The step is ordered write-then-ask; the confirmation prompt is the last line of the step, after the
write instruction and the format spec.

- `skills/migrate/SKILL.md:320` — *"**After user confirms**, write migration steps to `TODO.md` under a `### Migration {SOURCE} → {TARGET} — YYYY-MM-DD` section."*
- `skills/migrate/SKILL.md:329-331` — *"If no `TODO.md` exists: create with …"* / *"If `TODO.md` exists but has no `## Backlog` section: append one."*
- `skills/migrate/SKILL.md:333` — *"Offer: \"Add these steps to TODO.md backlog? (Y/n)\""* ← the ask, positioned last

The flow digraph agrees with the *intent*, not the text: `skills/migrate/SKILL.md:58` —
`todo [label="Offer TODO.md items\n(user confirms)"]`.

**Impact**: a literal executor creates `TODO.md` and appends a section before the user has been
asked. The "After user confirms" clause and the trailing "Offer:" line are the same gate stated
twice, out of order.

### B4 — [MAJOR] Tribunal's interactive path specifies a tool capability that does not exist

Tribunal's no-args path asks for free-text input via `AskUserQuestion` with a `placeholder` and five
numbered fields. `AskUserQuestion` presents multiple-choice options — it has no text-input or
placeholder affordance, and no notion of "fields 2–5".

- `skills/tribunal/SKILL.md:225-231`:
  - *"Use **AskUserQuestion** with text inputs:"*
  - *"`placeholder`: \"e.g., 'Postgres vs DynamoDB'\" or \"Database choice\""*
  - *"Collect decision title in field 1"* / *"Field 2-5: Up to 4 options (fields 2-3 required, 4-5 optional)"*
- Contrast the correct usage pattern elsewhere: `skills/skeptic/SKILL.md:85-110` (`AskUserQuestion`
  with `multiSelect` + an `options:` list), `skills/skeptic/SKILL.md:362-369` (same shape).
- `README.md:91` — *"`/tribunal`  # Interactive prompt for decision and options"* (promises the path).
- `skills/tribunal/SKILL.md:26` — *"Use **AskUserQuestion** to prompt for the decision title and 2-4 options interactively"* (states it without the impossible parameters).

**Impact**: the documented interactive entry point cannot be executed as written. The executor must
improvise (plain text prompt, or a synthesised options list), so behavior is undefined for
`/tribunal` with no args — one of three invocation forms the README advertises.

### B5 — [MAJOR] Inline recon persists nothing, so `/skeptic plan` can never see it

Skeptic's Step 3.5 runs recon's competitor agents inline and folds results into synthesis, but
never writes `.skeptic/recon-*.md` and never writes design notes. Plan mode's competitive-intelligence
loader reads only from disk.

- Inline recon dispatch and collection: `skills/skeptic/SKILL.md:378-383` — *"**Dispatch competitor research agents** in parallel"* … *"**Collect recon results** — hold until all competitor agents return."* — no write step follows.
- Consumed only in memory: `skills/skeptic/SKILL.md:391` — *"**If recon ran**: Append a `{RECON_INTELLIGENCE_BLOCK}` to the synthesis prompt…"*
- The disk-only loader: `skills/skeptic/SKILL.md:780` — *"Glob for `.skeptic/recon-*.md` … If either exists, read the most recent of each."*
- Standalone recon *does* persist: `skills/recon/SKILL.md:280` — *"Write the full synthesis output to `.skeptic/recon-YYYY-MM-DD.md`"*; `skills/recon/SKILL.md:326` — design notes to `.skeptic/recon-designs/[gap-name].md`.
- `README.md:55` — *"competitive research agents dispatch immediately using the same protocol as `/dispatch:recon`"* — overstates: the *research* protocol is shared, the *persistence* protocol is not.

**Impact**: a user who answers "Yes — run recon" inside `/skeptic` pays for the research, then gets
none of it in a later `/skeptic plan`. This is distinct from the prior F2 glob mismatch — even with
the glob corrected, inline recon leaves no file to match.

### B6 — [MAJOR] Roadmap-first routing is implemented in 2 of 5 persistence steps

Skeptic's review step and recon both implement the "roadmap preferred, TODO.md fallback" contract in
full. Landscape, migrate, and skeptic's overarching plan write straight to `TODO.md` with no
`docs/roadmap.md` branch at all.

- Implements roadmap-first:
  - `skills/skeptic/SKILL.md:537` — *"**A. Roadmap routing (preferred):** Check if `docs/roadmap.md` exists…"*; `:551` — *"**B. TODO.md fallback:**"*
  - `skills/recon/SKILL.md:359` / `:373` — same A/B structure.
- TODO.md only, no roadmap branch:
  - `skills/landscape/SKILL.md:462` — *"Append to `TODO.md` under `### Landscape Opportunities — YYYY-MM-DD`"*
  - `skills/migrate/SKILL.md:320` — *"write migration steps to `TODO.md` under a `### Migration …` section"*
  - `skills/skeptic/SKILL.md:1035` — overarching plan Execution Flow step 8: *"Offer TODO.md Phase 1 items (user confirms)"*
- `README.md:63` and `README.md:80` advertise roadmap-first for skeptic and recon only — so the
  README is accurate; the gap is that three persistence surfaces were never brought up to the contract.

**Impact**: in a project with `docs/roadmap.md`, three of the five artifact-producing skills bypass
it and drop dated sub-headings into `TODO.md`, which then need manual routing.

### B7 — [MINOR] Recon's console summary hardcodes TODO.md, contradicting its own Step 8

- `skills/recon/SKILL.md:412` — console block prints *"Backlog items:  X added to TODO.md (Y deduped)"*
- `skills/recon/SKILL.md:359-371` — Step 8 branch A routes to `docs/roadmap.md` and explicitly says
  *"Do NOT also write these items to TODO.md backlog — roadmap is the destination."*

**Impact**: after a roadmap-routed run the console reports a destination that was deliberately not
written to.

### B8 — [MINOR] Skeptic's Step 3.5 claims "all 8 agents dispatch" regardless of selection

- `skills/skeptic/SKILL.md:359` — *"All 8 agents dispatch in parallel (Step 3 unchanged). **Only applies when Concept & Strategy agent was selected.**"*
- Contradicted by Step 1: `skills/skeptic/SKILL.md:83` — *"If user passed args (`/skeptic full`, `/skeptic arch code`), skip menu"*, and Step 3 `:118` — *"Spawn **all selected agents** in parallel"*.
- `README.md:35` — *"`/skeptic arch code security`  # Multiple specific agents"*.

Self-correcting in practice (the bolded guard covers the case), but the leading sentence is false for
every selective invocation.

---

## 2. Findings — tool grants and permissions

### B9 — [MAJOR] Migrate states no tool grant or prohibition anywhere

Every other skill has an explicit agent capability statement. Migrate has none — its agents' tools
are implied only by imperative sentences inside the agent prompt.

- Present elsewhere: `skills/skeptic/SKILL.md:623` / `:627`; `skills/tribunal/SKILL.md:39`, `:414` / `:416`;
  `skills/cartograph/SKILL.md:371` / `:372`; `skills/landscape/SKILL.md:370`; `skills/recon/SKILL.md:65-70`
  (*"**No Bash** — research only, no execution"*).
- Migrate's only signal: `skills/migrate/SKILL.md:117-118` — *"Use Read, Glob, and Grep to understand the current {SOURCE} usage."* / *"Use WebSearch or WebFetch to research {TARGET}'s API…"*
- No "must NOT do" line exists in `skills/migrate/SKILL.md` — so nothing forbids Write/Edit/Bash for
  migrate's five dimension agents, unlike the read-only guarantee every sibling skill gives.

**Impact**: migrate is the only analysis skill without a stated read-only boundary. Since its agents
analyse a live codebase before a risky migration, an unbounded agent is the worst case to leave
unstated.

### B10 — [MINOR] Skeptic grants Bash to all agents, then re-grants it as "extended" and revokes it

Four adjacent lines describe three different Bash policies.

- `skills/skeptic/SKILL.md:623` — *"**What agents can use**: Read, Glob, Grep, **Bash** (read-only commands like `git log`, `npm ls`, etc.)"* — universal grant.
- `skills/skeptic/SKILL.md:624` — *"**DX agent extended access**: Bash for running project test/UX tools…"* — framed as an extension of a grant that line 623 already gave everyone.
- `skills/skeptic/SKILL.md:625` — *"**Testing agent extended access**: Bash for running test discovery commands…"* — same.
- `skills/skeptic/SKILL.md:626` — *"**Concept & Strategy agent access**: Read, Glob, Grep, WebSearch. **No Bash**…"* — revokes the universal grant for one agent.

Either line 623 should not list Bash (making 624/625 genuine extensions), or 624/625 should not say
"extended". As written, a Performance or Architecture agent's Bash access is ambiguous.

### B11 — [MINOR] Tribunal's own console block undercuts its design-doc step

Tribunal implements full `docs/designs/` persistence at Step 8, but its second console block tells
the user design-doc persistence is a manual kerd action.

- `skills/tribunal/SKILL.md:318-352` — Step 8 *"Offer Design Doc Persistence"* → `docs/designs/tribunal-{slug}-YYYY-MM-DD.md`, incl. the CLAUDE.md `## Design Docs` inject.
- `skills/tribunal/SKILL.md:441` — console line: *"Design doc persistence: Optional — use /kerd:kivna save"*
- `README.md:159` lists tribunal among the five skills that *do* persist to `docs/designs/`.

### B12 — [MINOR] CLAUDE.md's release checklist names a version field that does not exist

- `CLAUDE.md` Release Checklist step 1 — *"Bump version in `.claude-plugin/plugin.json` **and `.claude-plugin/marketplace.json`**…"*
- `.claude-plugin/marketplace.json` has **no `version` key** — its `plugins[]` entry carries only
  `name`, `description`, `source`, `category`. Only `.claude-plugin/plugin.json` has `"version": "2.2.3"`.

**Impact**: half of the first release step is unexecutable; anyone following it either invents a
field or silently skips it.

---

## 3. Findings — command examples, ordering, and diagrams

### B13 — [MINOR] Tribunal parses different separators in Input Parsing vs. Step 1

- `skills/tribunal/SKILL.md:21` — *"Split on `vs` / `or` / commas (case-insensitive split boundaries, trim whitespace)"*
- `skills/tribunal/SKILL.md:220` — Step 1: *"Split on `vs` (case-insensitive)"* — `or` and commas dropped.

`README.md:89-90` only shows `vs` examples, so a comma-separated invocation
(`/tribunal "Postgres, DynamoDB"`) is accepted by one section and unparsed by the other.

### B14 — [MINOR] Tribunal specifies two different console summaries

Two blocks describe the same post-synthesis console output with different labels and content.

- `skills/tribunal/SKILL.md:280-302` (Step 6) — *"OPTIONS EVALUATED: {N}"* with a per-option `✓` list, and *"KEY FACTORS:"* filled with actual findings.
- `skills/tribunal/SKILL.md:422-443` ("Console Output Format") — *"OPTIONS ANALYZED:"* / *"{N} options submitted to advocate review"*, and *"KEY FACTORS:"* as three static category labels; adds the `/kerd:kivna save` line (see B11).

No statement says one supersedes the other.

### B15 — [MINOR] "Same Step 3" cross-references from skeptic resolve ambiguously in recon

Skeptic points at recon's Step 3 twice for two *different* procedures. Recon has two sections
numbered Step 3, one for each — so each pointer happens to be satisfiable, but only by guessing.

- `skills/skeptic/SKILL.md:376` — *"Parse named products/tools/libraries … (same extraction logic as **recon SKILL.md Step 3**)"* → matches `skills/recon/SKILL.md:33` *"### Step 3: Extract Competitor List"*.
- `skills/skeptic/SKILL.md:377` — *"**Read project identity** — README, docs/, config files (same as **recon SKILL.md Step 3**)"* → matches `skills/recon/SKILL.md:219` *"### Step 3: Read Target Project Identity"*.

The duplicate numbering itself was reported before (F12 / L2); what is new is that a **second file
depends on it**, so renumbering recon requires updating `skills/skeptic/SKILL.md:376-377` in the same
change.

### B16 — [MINOR] Skeptic's overarching plan output path breaks its own "all subcommands" rule

- `skills/skeptic/SKILL.md:785` — Common Behavior, stated as applying to *"All plan subcommands"*: *"**Write output**: `.skeptic/plan-{type}-YYYY-MM-DD.md`."*
- `skills/skeptic/SKILL.md:1033` — overarching Execution Flow: *"Write to `.skeptic/plan-YYYY-MM-DD.md`"* — no `{type}` segment.

Harmless in isolation, but any tooling that globs `plan-{type}-*` (or a future dogfood contract, §5)
will not match the overarching plan.

### B17 — [NITPICK] Four of five execution-flow digraphs stop before the persistence steps

The `dot` diagrams are the fastest read of each skill's flow and they omit the final steps.

- `skills/skeptic/SKILL.md:44-77` — ends at `console` / `file`; no node for Step 6 (Persist Future Work, `:530`) or Step 7 (Suggest Plan, `:565`).
- `skills/cartograph/SKILL.md:37-62` — ends at `console`; no node for Step 6 design-doc persistence (`:301`).
- `skills/migrate/SKILL.md:44-70` — ends at `todo`; no node for Step 9 design-doc persistence (`:335`).
- `skills/landscape/SKILL.md:311-342` — ends at `backlog` → `end`; no node for Step 9 design-doc persistence (`:471`).
- Correct: `skills/tribunal/SKILL.md:186-213` — includes `offer [label="Offer design doc\npersistence"]`.

Recon has no digraph at all, so five skills have diagrams; four are stale.

---

## 4. Prior findings — current status (reference only, not re-reported)

| Prior ID | Status today | Evidence |
|----------|--------------|----------|
| F1 / H1 — `/skeptic:recon` namespace | **FIXED** | `skills/recon/SKILL.md:13-14,19` and `README.md:76-77` now read `/dispatch:recon` |
| F3 / H3 — README "all skills write docs/designs/" | **FIXED** | `README.md:159-161` now names the five skills and documents recon as the explicit exception |
| F2 / H2 — `.landscape/report-*.md` glob | **OPEN** | `skills/skeptic/SKILL.md:780`, `:957`, `README.md:59` still glob `report-*`; landscape still writes `survey-*` (`skills/landscape/SKILL.md:393`) |
| F4 / M1 — Testing agent has no model in Step 3 | **OPEN** | `skills/skeptic/SKILL.md:121-122` still lists 7 of 8 |
| F5 / M3 — skeptic write-scope claim | **OPEN** | `skills/skeptic/SKILL.md:563` still says TODO/roadmap are the only writes outside `.skeptic/` |
| F6 / M2 — `concept` missing from plan-type list | **OPEN** | `skills/skeptic/SKILL.md:18`, `:731` |
| F7 / L6 — bare `/skeptic` vs `/dispatch:` | **OPEN** | all five remaining SKILL invocation blocks + README |
| F8 / L1 — stale pinned model IDs | **OPEN** | `skills/skeptic/SKILL.md:38`, `skills/cartograph/SKILL.md:31-32` |
| F9 — recon "no report" wording | **OPEN** | `skills/recon/SKILL.md:25` vs `:425` |
| F10 / L5 — cartograph Domain supplement says "data flow" | **OPEN** | `skills/cartograph/SKILL.md:155` |
| F11 / L4 — tribunal "two jobs", defines four | **OPEN** | `skills/tribunal/SKILL.md:102` vs `:104,117,125,131` |
| F12 / L3 — duplicate Step 7, empty Teardown | **OPEN** | `skills/skeptic/SKILL.md:785-786`; `skills/recon/SKILL.md:419` |

---

## 5. Branch review — `scorched/dispatch-dogfood-verification-harness`

**Commit**: `af4bd3c` — *"test: add dogfood harness for the six dispatch skills"*, Kwanwoo Lee,
2026-07-19. Single commit off `main`. Diff vs `main`: **56 files, +1477 / −1**.

### What it contains

| Group | Files | Purpose |
|-------|-------|---------|
| `tests/dogfood/run.sh` | 1 (90 ln) | driver: `plan` \| `stage` \| `verify` \| `selftest` |
| `tests/dogfood/verify.py` | 1 (186 ln) | structural diff engine (path / section / console drift classes) |
| `tests/dogfood/expected/*.json` | 6 | machine-readable extract of each skill's promised output path, sections, console labels |
| `tests/dogfood/fixtures/**` | 11 | tiny inputs: buggy app (skeptic), URL shortener (cartograph), Express app (migrate), a decision doc (tribunal), a domain string (landscape), a synthetic prior skeptic report (recon) |
| `tests/dogfood/actual/_samples`, `_drift` | 24 | committed conformant + drifted captures that self-test the verifier |
| `tests/dogfood/README.md` | 1 (91 ln) | harness contract and usage |
| `.scorched/dogfood-report*.md` | 2 | the two summary reports produced by the self-test |
| `.claude-plugin/plugin.json` | 1 | version `2.2.0 → 2.2.1` |

### Verification performed

Ran the harness self-test from the existing worktree
(`.scorched/wt/dispatch-dogfood-verification-harness`), foreground:

```
bash tests/dogfood/run.sh selftest
== conformant samples ==
  [PASS ] cartograph / landscape / migrate / recon / skeptic / tribunal — 0 findings each
== drift samples (expected to report drift) ==
  [MISS ] cartograph, landscape, recon — 1 finding each
  [DRIFT] migrate, skeptic, tribunal — 2 findings each
```

**The verifier works.** 6/6 conformant captures pass, 6/6 drifted captures are correctly flagged, and
the run is fully offline — no Claude fan-out, no network, no tokens.

### Public-repo divergence scan

`git grep -iE 'kerd|kivna|pullmd|slainte|dian'` over the branch's added paths
(`tests/dogfood`, `.scorched`, `.claude-plugin`) → **no matches**. The branch is clean for
publication. (Note: `kerd` references already exist in the *published* surface — `README.md:165-167`
"Kerd Integration", plus Kerd Integration sections in skeptic/tribunal/cartograph — so kerd is a
documented public integration, not a divergence leak. The branch adds none.)

### Correctness spot-check of `expected/*.json` against current SKILL.md

Each contract's `output_file` matches the SKILL.md it cites, including the two paths prior audits
found contested:

- `landscape.json` → `.landscape` / `survey-*.md`, citing `SKILL.md:393` — matches
  `skills/landscape/SKILL.md:393`, i.e. the contract encodes the **correct** `survey-` prefix, not
  skeptic's broken `report-*` glob (prior F2). Good.
- `recon.json` → `.skeptic` / `recon-*.md` (`SKILL.md:280`) — matches.
- `skeptic.json` → `.skeptic` / `report-*.md` (`SKILL.md:494`), `tribunal.json` → `.tribunal` /
  `decision-*.md` (`SKILL.md:309`), `cartograph.json` → `docs/cartograph.md` (`SKILL.md:272`),
  `migrate.json` → `docs` / `migration-*.md` (`SKILL.md:286`) — all match.
- `recon.json` carries a `stage` block copying the fixture skeptic report into `.skeptic/` because
  `.skeptic/` is git-ignored — correct handling of this repo's `.gitignore`.

The invocations in every contract use the **`/dispatch:<skill>` namespace**, which is `CLAUDE.md`'s
stated convention — i.e. the harness is more convention-correct than the SKILL.md invocation blocks
it tests (prior F7).

### Problems the branch carries

1. **`plugin.json` conflicts.** Branch bumps `2.2.0 → 2.2.1`; `main` is already at `2.2.3`. A merge
   conflicts on that one line. Resolution is trivially "keep main's 2.2.3" — the harness is
   test-only and does not change user-facing skill behavior, so per `CLAUDE.md` Release Checklist
   step 1 it needs no bump at all.
2. **Two report files land inside a git-ignored directory.** `.scorched/dogfood-report-2026-07-19.md`
   and `.scorched/dogfood-report-drift-2026-07-19.md` are tracked on the branch, but `.gitignore`
   ignores `.scorched/` — they were force-added. This is the same wart already flagged for the
   consistency audits ("audit reports live in tracked `.scorched/*.md` inside the now-ignored dir —
   relocate to `docs/` if that offends"). Both are *generated* reports and the self-test regenerates
   them on demand.
3. **`.scorched/` is the harness's default report destination.** `run.sh verify` writes to
   `../../.scorched/`, so on a fresh clone (where `.scorched/` may not exist and is ignored) reports
   land in an ignored dir by design. Acceptable for a scratch artifact; worth pointing at
   `docs/audits/` if these reports are meant to be reviewed later.
4. **Depends on `python3`.** `verify.py` is Python; the repo is otherwise documentation-only
   (`CLAUDE.md`: *"Documentation-only plugin (no runtime code)"*). Merging makes that statement
   partially false and adds an interpreter dependency for anyone running the harness.

### Recommendation: **MERGE**, with three fixes first (do not merge as-is)

Rationale for merge:

- It is the only *executable* check that exists against the SKILL.md contracts. Three consecutive
  audits (2026-07-18, 2026-07-19, this one) have been hand-traced; every one of them found path and
  section drift that this verifier detects mechanically. The `expected/*.json` files also give the
  audits a durable machine-readable baseline, which is exactly what §4 above has to reconstruct by
  hand each time.
- It is self-proving: the committed `_samples` / `_drift` captures mean the diff engine's correctness
  does not depend on ever running a real fan-out. Self-test is offline and free.
- Risk is contained: it adds only `tests/`, touches no `skills/*/SKILL.md`, and cannot affect plugin
  behavior for users (the published plugin ships skills; `tests/` is inert).
- Its contracts are already *more* correct than the docs they test (correct `survey-*` path, correct
  `/dispatch:` namespace), so it will not entrench any of the open findings.

Required before merge:

1. **Drop the `plugin.json` bump** — take `main`'s `2.2.3`. Test-only change; no user-facing behavior
   changed, so the Release Checklist does not require a bump.
2. **Move the two `.scorched/dogfood-report-*.md` files out of the commit**, or relocate them (and
   `run.sh`'s output path) to `docs/audits/`. Do not merge tracked files into a git-ignored dir; they
   are regenerable by `run.sh selftest` anyway.
3. **Amend `CLAUDE.md`** — *"Documentation-only plugin (no runtime code)"* becomes false. Add a line
   noting `tests/dogfood/` is a Python-based structural verifier, not shipped with the plugin, plus
   the `python3` prerequisite.

Optional follow-up (post-merge, not blocking): wire `run.sh verify` into CI as a gate once real
captures exist, per `tests/dogfood/README.md` — *"`verify.py` exits `0` only when every checked skill
PASSes … usable as a CI gate once real captures are wired in."*

Explicitly **not** done in this session: the branch was **not merged**. Recommendation only, per the
task brief.

---

## 6. Summary table

| ID | Sev | Skill | One-line |
|----|-----|-------|----------|
| B1 | MAJOR | landscape | every "too broad" example is 1 word and is rejected by the 2-word minimum first |
| B2 | MAJOR | landscape | >10 words = reject (`:23`) vs warn-and-continue (`:349`, `:514`) |
| B3 | MAJOR | migrate | Step 8 writes/creates TODO.md before the confirmation prompt it ends with |
| B4 | MAJOR | tribunal | interactive path needs AskUserQuestion text inputs + `placeholder` + 5 fields — not a capability of that tool |
| B5 | MAJOR | skeptic | inline recon writes no `.skeptic/recon-*.md`, so `/skeptic plan` can never load it |
| B6 | MAJOR | cross-skill | roadmap-first routing implemented in skeptic-review + recon only; landscape, migrate, plan-overarching go straight to TODO.md |
| B7 | MINOR | recon | console prints "added to TODO.md" even on the roadmap-routed path that forbids TODO.md |
| B8 | MINOR | skeptic | Step 3.5 says "all 8 agents dispatch" — false for any selective invocation |
| B9 | MAJOR | migrate | no tool-grant or "must NOT do" statement anywhere; the only skill with no read-only boundary |
| B10 | MINOR | skeptic | Bash granted to all agents (`:623`), then re-granted as "extended" (`:624-625`), then revoked (`:626`) |
| B11 | MINOR | tribunal | console says design-doc persistence is manual `/kerd:kivna save`, contradicting its own Step 8 |
| B12 | MINOR | CLAUDE.md | Release Checklist says bump `marketplace.json`'s version; that file has no version field |
| B13 | MINOR | tribunal | splits on `vs`/`or`/commas (`:21`) vs `vs` only (`:220`) |
| B14 | MINOR | tribunal | two conflicting console-summary specs (`:280-302` vs `:422-443`) |
| B15 | MINOR | skeptic→recon | two cross-file "recon SKILL.md Step 3" pointers rely on recon's duplicate Step 3 numbering |
| B16 | MINOR | skeptic | overarching plan writes `plan-YYYY-MM-DD.md`, breaking the `plan-{type}-…` rule stated for "all subcommands" |
| B17 | NITPICK | 4 skills | execution-flow digraphs stop before the design-doc / persistence steps (tribunal's is correct) |

**Branch verdict**: `scorched/dispatch-dogfood-verification-harness` — **merge after 3 fixes**
(drop the plugin.json bump, relocate the two `.scorched/` reports, amend the "documentation-only"
line in CLAUDE.md). Verified working: 6/6 conformant PASS, 6/6 drift detected, no kerd/kivna/PullMD
leakage. Not merged in this session.
