# Agent Roles and Orchestration

## Table of Contents
- [Architecture Overview](#architecture-overview)
- [Planner Agent](#planner-agent)
- [Review Agent](#review-agent)
- [Theory Agent](#theory-agent)
- [Experiment Agent](#experiment-agent)
- [Writing Agent](#writing-agent)
- [Layout Agent](#layout-agent)
- [Agent Communication Patterns](#agent-communication-patterns)

## Architecture Overview

```
                    +-----------------+
                    |  Planner Agent  |
                    |  (Orchestrator) |
                    +-------+---------+
                            |
              +-------------+-------------+
              |             |             |
        +-----+---+  +-----+---+  +-----+---+
        | Theory  |  |Experiment|  | Writing |
        |  Agent  |  |  Agent   |  |  Agent  |
        +---------+  +----+----+  +---------+
                          |
                    +-----+-----+
                    | parallel   |
                    | sub-agents |
                    +-----------+

   4 Review Agents (parallel per round)        +-----------+
   +---------------------------+               |  Layout   |
   | Historical Reviewer 1     | <-----------> |  Agent    |
   | Historical Reviewer 2     |               +-----------+
   | Fresh Reviewer 1          |
   | Fresh Reviewer 2          |
   +---------------------------+
   aggregated summary report feeds back
   into Historical Reviewers next round
```

Core loop: Plan -> Write/Derive/Experiment -> Review (x4 parallel) -> Aggregate -> Revise -> Review -> ...

## Planner Agent

**Role**: Orchestrate the full paper lifecycle — from-scratch writing or revision of an existing draft.

**Responsibilities**:
- Analyze user input: new paper requirements OR existing draft + revision goals
- Produce a structured plan with prioritized tasks
- Decide which agents to invoke, sequentially or in parallel
- Merge outputs from parallel agents into a coherent draft
- Trigger review cycles; translate review feedback into revision tasks
- Track progress; decide when the paper meets the quality bar

**Decision rules**:
- Independent theory and experiments -> dispatch in parallel
- Review finds >3 MAJOR issues -> full revision cycle
- Review finds only MINOR issues -> targeted fix via Writing/Layout Agent
- All 4 reviewers have median >= 4 AND no dimension below 3 -> paper is ready

**For revising existing papers**:
- First dispatch Review Agent on the current draft to produce a baseline assessment
- Categorize issues by severity and affected section
- Plan revision order: critical structural issues first, then content, then polish

## Review Agent

**Role**: Harsh, constructive academic reviewer. Produce structured review reports per `references/review-criteria.md`.

**Invocation**: 4 instances dispatched in parallel after each major revision, and as the first step when revising an existing draft. 2 are Historical Reviewers; 2 are Fresh Reviewers.

**Shared Behavior** (all 4 reviewers):
- Read the full LaTeX source (and compiled PDF if available)
- Evaluate against all criteria in review-criteria.md
- Produce a review report using the template in review-criteria.md
- Be specific: reference exact equations, figures, tables, sections
- Never say "looks good" without evidence
- Check every empirical claim against actual numbers in tables
- Verify abstract matches actual contributions and results
- Flag any notation inconsistency, overclaiming, or missing reference

### Historical Reviewer (2 per round)

**Additional context**: Receives the current draft AND the **aggregated summary report from the previous round only** (not the full history of all rounds).

**Additional responsibilities**:
- For each issue listed in the previous round's aggregated summary, explicitly check whether it has been genuinely resolved or only superficially addressed
- Fill in the `Unresolved from last round` section of the report template
- If a fix introduced a new problem (e.g., a rewritten proof now has a gap), flag it as a new CRITICAL issue
- Do not re-flag issues that are cleanly resolved; acknowledge them in the `Resolved` list

### Fresh Reviewer (2 per round)

**Additional context**: Receives only the current draft. Has no access to prior review reports.

**Additional responsibilities**:
- Provide a fully independent assessment unanchored to previous feedback
- Focus on issues that may have been overlooked or introduced in recent revisions
- Leave the `Unresolved from last round` section blank (N/A)

## Theory Agent

**Role**: Derive, verify, and strengthen the mathematical content.

**Capabilities**:
- Derive new results (theorems, propositions, corollaries, lemmas, bounds)
- Verify existing proofs for correctness and completeness
- Identify gaps where theory could strengthen the paper
- Ensure all assumptions stated explicitly
- Check notation consistency across all math content
- Suggest tighter bounds, stronger results, or cleaner formulations

**Output**: LaTeX-ready theorem/proof blocks with surrounding text.

## Experiment Agent

**Role**: Design, implement, run experiments. Spawn parallel sub-agents for independent experiments.

**Parallel sub-agent pattern**: When multiple independent experiments are needed (different scenarios, parameter sweeps, ablations), spawn as parallel Task agents:
```
Experiment Agent
  |-- Sub-agent: Scenario A
  |-- Sub-agent: Scenario B
  |-- Sub-agent: Sensitivity analysis
  +-- Sub-agent: Ablation study
```

**Output**:
- Reproducible scripts (deterministic seeds, logged parameters)
- Publication-quality figures (PDF format, readable at print size)
- LaTeX table code
- Brief text summarizing key findings

## Writing Agent

**Role**: Draft, revise, and polish prose.

**Capabilities**:
- Draft sections from outlines or bullet points
- Compress text to fit page limits while preserving information
- Ensure consistent terminology and notation references
- Adapt citation style to target venue
- Rewrite sections based on review feedback

**Quality targets**:
- Every paragraph has a clear purpose
- No redundant sentences
- Claims precise and quantified
- Transitions between sections explicit
- Abstract self-contained

## Layout Agent

**Role**: Formatting, typesetting, and visual quality of the compiled PDF.

**Capabilities**:
- Fix equation overflow (overfull hbox)
- Convert between document formats (article, IEEEtran, NeurIPS, ICML, etc.)
- Optimize table formatting for column width
- Adjust figure sizing and placement
- Ensure zero LaTeX compilation warnings

**Common techniques**:
- `align` -> `multline` for long equations with line breaks
- Introduce abbreviations for repeated long expressions
- `\footnotesize` + `\tabcolsep` for tight tables
- `bmatrix` for column vectors
- `\!` negative thin spaces for math tightening

## Agent Communication Patterns

### Pattern 1: Review-Revise Loop
```
last_round_summary = None

Planner -> [Agent(s)] ->
  parallel: [Historical R1(draft+prev) | Historical R2(draft+prev) |
             Fresh R1(draft)           | Fresh R2(draft)           ]
  -> Planner (aggregate all 4 reports, merge issues, prioritize unresolved)
  -> [Agent(s) revise]
  -> last_round_summary = aggregated summary report from this round
  -> last_round_summary = aggregated summary report from this round
  -> repeat until all 4 reviewers have median >= 4 AND no dimension below 3, or MAX_ROUNDS reached
```

### Pattern 2: Parallel Theory + Experiments
```
Planner -> [Theory Agent | Experiment Agent] (parallel)
        -> Planner (merge) -> Writing Agent (integrate)
```

### Pattern 3: Parallel Experiment Sub-agents
```
Experiment Agent -> [Scenario 1 | Scenario 2 | ... | Scenario N] (parallel)
                 -> Experiment Agent (aggregate)
```

### Pattern 4: Layout Fix Cycle
```
Layout Agent -> Compile -> Check warnings -> Fix -> Compile
            -> ... (until zero warnings)
```

### Pattern 5: Final Polish
```
Writing Agent -> Layout Agent -> Review Agent -> Planner (accept or iterate)
```
