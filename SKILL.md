---
name: academic-paper-writing
description: >
  Multi-agent workflow for writing and revising academic research papers (STEM focus).
  Use when the user wants to: (1) write a new paper from scratch given research results,
  (2) revise/improve an existing paper draft, (3) strengthen theory, experiments, or writing
  quality of a paper, (4) convert a paper to a different venue format (e.g., article to IEEE/NeurIPS),
  (5) get a harsh academic review of a draft. Triggers include: "write a paper", "revise this paper",
  "review my draft", "improve the writing", "strengthen the theory", "fix the experiments",
  "convert to IEEE format", "polish for submission", or similar academic paper requests.
---

# Academic Paper Writing & Revision

Multi-agent system for writing new and revising existing academic papers. Uses a Review-Revise loop with specialized agents for theory, experiments, writing, and layout.

## Entry Point: Phase 0 — Contextual Q&A

Before dispatching any agents, gather critical context from the user. Ask these questions upfront (use AskUserQuestion where available):

**Q1 — Starting point** (determines workflow mode):
- I have research results / ideas, no draft yet → Mode A
- I have an existing .tex / .pdf draft → Mode B
- I need a specific isolated task (fix equations, convert format, etc.) → Mode C

**Q2 — Target venue** (determines page budget, style, rigor bar):
- ML conference (NeurIPS / ICML / ICLR) — 9 pages, theory + experiments
- Computer vision (CVPR / ICCV / ECCV) — 8 pages, strong visuals
- IEEE / Springer journal — 12-14 pages, comprehensive
- Workshop / arXiv — flexible, exploratory OK

**Q3 — Paper focus** (determines agent effort allocation):
- Theory-heavy (proofs, bounds, convergence)
- Experiment-heavy (benchmarks, ablations, SOTA comparisons)
- Balanced theory + experiments
- Application / case study

**Q4 — Quality target** (determines review loop depth):
- Submit-ready: iterate until all reviewers pass
- Draft for feedback: 1-2 review rounds, report issues to user
- Quick skeleton: structure only, minimal content generation

If Q1 answer is **Mode B or C**, also ask:

**Q5 — Main weakness** (multi-select, Mode B only):
- Theory is weak or incomplete
- Experiments are insufficient or missing ablations
- Writing is verbose or unclear
- Everything — run a full harsh review first

After collecting answers, inject a **context block** into every subsequent agent call:
```
context:
  venue: [answer to Q2]
  page_limit: [derived from venue]
  focus: [answer to Q3]
  quality_target: [answer to Q4]
```

If user skips Q&A or input is unambiguous (e.g., .tex file attached + "revise"), auto-infer mode and skip relevant questions. Always ask Q2 and Q4 if not clear.

## Mode Dispatch

After Phase 0:
1. **Mode A** (no draft) → Write from Scratch workflow below
2. **Mode B** (existing draft) → Revise Existing Paper workflow below
3. **Mode C** (specific task) → Dispatch directly to the relevant agent; run a single Fresh Reviewer pass afterward to check for regressions; ask user if they want a full review.
   - If user requests full review after Mode C: initialize `last_round_summary` from the Mode C Fresh Reviewer report, then enter the standard 4-reviewer loop starting at round 1.

## Mode A: Write from Scratch

### Phase 1: Planning
Use the Planner Agent role (see `references/agent-roles.md`).

1. Read user's input: problem description, method, results, target venue
2. Produce a paper outline:
   - Title, abstract sketch, section structure
   - Identify which sections need theory derivation vs. experiment design vs. prose
   - Identify what can be done in parallel
3. Create a TodoWrite plan tracking each section

### Phase 2: Parallel Content Generation
Dispatch agents in parallel where possible:

- **Theory Agent** (Task subagent): Derive all mathematical content — model, algorithm, theorems, proofs, bounds. Output LaTeX-ready blocks.
- **Experiment Agent** (Task subagent, may spawn parallel sub-agents per scenario): Design benchmarks, write experiment scripts, generate figures and tables. Output scripts + LaTeX figure/table code.
- **Writing Agent**: Draft related work, introduction framing, conclusion. Can begin once outline is set.

Merge results into a single .tex file.

### Phase 2.5: Cross-Agent Consistency Check
Before proceeding to review, Planner verifies consistency across parallel agent outputs:

1. **Assumption alignment**: check that Theory Agent's assumptions (e.g., Lipschitz constant, convexity, distribution) match the conditions used in Experiment Agent's setups
2. **Notation consistency**: verify symbols defined by Theory Agent are used consistently in Writing Agent's prose
3. **Claim alignment**: ensure claims in Writing Agent's introduction/abstract match actual results in Theory and Experiment outputs

If conflicts detected:
- Log each conflict as a MAJOR issue
- Dispatch the conflicting agent(s) sequentially to reconcile (Theory Agent or Experiment Agent, depending on which side needs updating)
- Re-merge and re-check before proceeding to Phase 3

### Phase 3: Review-Revise Loop
Run iteratively until the pass condition is met or the round limit is reached:

```
last_round_summary = None   # aggregated summary report from the previous round
round = 0
MAX_ROUNDS = 10

loop:
  round += 1

  Dispatch 4 Review Agents in parallel:
    - Historical Reviewer 1 (draft + last_round_summary)
    - Historical Reviewer 2 (draft + last_round_summary)
    - Fresh Reviewer 1     (draft only)
    - Fresh Reviewer 2     (draft only)

  Aggregate reports:
    - Collect all CRITICAL / MAJOR / MINOR issues across 4 reports
    - Merge duplicate issues, weight by number of reviewers who flagged them
    - Issues flagged as "Unresolved from last round" by Historical Reviewers escalate one severity level
    - Produce a single aggregated summary report (see Aggregation Rules in review-criteria.md)

  Planner checks pass condition:
    PASS if: for each of the 4 reviewers, median >= 4 AND no dimension below 3
    if PASS: break

  if quality_target == "Draft for feedback" and round >= 2:
    Present aggregated issues to user; break (do not require pass condition)

  if round >= MAX_ROUNDS:
    Present current state to user: scores per reviewer, top unresolved issues
    Warn if any reviewer has median < 4 or any dimension below 3
    Ask: "10 rounds completed without full convergence. Continue (10 more rounds) or accept current draft?"
    if continue: MAX_ROUNDS += 10  (do not reset round counter; preserve last_round_summary)
    else: break

  Planner creates revision task list from aggregated issues
  Dispatch fixes to appropriate agent(s)
  Merge revisions
  last_round_summary = aggregated summary report from this round
```

### Phase 4: Layout Polish
- Layout Agent: compile, check for warnings, fix overflow/formatting
- Final Review Agent pass on the compiled PDF

## Mode B: Revise Existing Paper

### Phase 0.5: Input Validation
Before proceeding, verify the input format:

- **If .tex source available**: proceed normally.
- **If PDF only (no .tex)**:
  - Warn the user: "Only a PDF was provided. Revisions will be limited — I can extract text for review, but cannot make precise LaTeX edits without the source file."
  - Ask: "Do you want to (a) proceed with text-extraction-based review only, or (b) provide the .tex source?"
  - If (a): extract text via `pdftotext`, run review-only workflow (no Writing/Theory/Experiment agents), present review report and stop.
  - If (b): wait for .tex before proceeding.

### Phase 1: Baseline Review
1. Read the existing .tex source (and compiled PDF if available)
2. Dispatch **4 Review Agents in parallel** (2 Historical with `last_round_summary = None`, 2 Fresh) to produce a full baseline assessment using the template in `references/review-criteria.md`
3. Aggregate into a single baseline `last_round_summary`
4. Present the aggregated baseline review to the user

### Phase 2: Revision Planning
1. Categorize issues from the review by severity: CRITICAL > MAJOR > MINOR
2. Group by type: theory, experiments, writing, layout
3. Ask user which issues to address (or address all if user says "fix everything")
4. Create a prioritized revision plan via TodoWrite

### Phase 3: Execute Revisions
Dispatch to specialist agents based on issue type:

| Issue type | Agent | Parallel? |
|-----------|-------|-----------|
| Missing/weak theorem | Theory Agent | Yes, with other theory tasks |
| Incorrect proof | Theory Agent | Sequential (depends on fix) |
| Missing ablation | Experiment Agent | Yes, parallel sub-agents per experiment |
| Weak baselines | Experiment Agent | Yes |
| Verbose prose | Writing Agent | Yes, per section |
| Overclaiming | Writing Agent | Sequential (needs global view) |
| Equation overflow | Layout Agent | After content changes done |
| Format conversion | Layout Agent | After content changes done |

### Phase 4: Verify
Re-run the 4-reviewer parallel loop using the same structure as Mode A Phase 3.

```
last_round_summary = aggregated summary from Mode B Phase 1 baseline review
round = 0
MAX_ROUNDS = 10

(same loop body as Mode A Phase 3)
```

`last_round_summary` is initialized from the baseline review produced in Phase 1, so Historical Reviewers start with context from that initial assessment. If new issues found after convergence, loop back to Phase 3.

## Review Agent Protocol

Read `references/review-criteria.md` for full criteria and report template. Key principles:

- **Be harsh**: assume the reader is an expert reviewer at a top venue
- **Be specific**: cite exact equations, figures, tables, line numbers
- **Check every claim**: compare prose claims against actual numbers in tables
- **Check balance**: section lengths should match guidelines in review-criteria.md
- **Check layout**: compile and verify zero warnings, readable figures, proper formatting
- **Never rubber-stamp**: always identify at least 3 items to improve

Scoring dimensions: Novelty, Theory/Rigor, Experiments, Reproducibility, Writing, Layout. Each 1-5.

**Pass condition**: for each of the 4 reviewers individually, both must hold: (1) median score across all 6 dimensions >= 4, and (2) no single dimension scored below 3. All 4 reviewers must pass this bar.

### Two Reviewer Types

**Historical Reviewer** (2 instances per round):
- Receives the current draft AND the **aggregated summary report from the previous round only** (not full history of all rounds)
- Focus: verify that issues flagged in the previous round are genuinely resolved, not superficially patched
- Must explicitly list any unresolved issues from the previous round in the `Unresolved from last round` section of the report template

**Fresh Reviewer** (2 instances per round):
- Receives only the current draft, no prior review context
- Focus: unbiased detection of new or overlooked issues
- Provides an independent perspective uncorrupted by anchoring to previous feedback

## Theory Agent Protocol

When strengthening theory:
1. Read the full method section to understand the mathematical framework
2. Identify opportunities: missing proofs, corollaries, bounds, convergence guarantees
3. For each new result: state assumptions precisely, write a complete proof, add connecting text
4. Verify notation consistency with the rest of the paper
5. Output: LaTeX theorem/proof environments ready for insertion

Common theory improvements:
- Add corollaries showing limiting behavior of key parameters
- Prove convergence or monotonicity properties of iterative algorithms
- Derive information-theoretic bounds (CRLB, mutual information) for benchmarking
- Add lemmas that break long proofs into digestible pieces

## Experiment Agent Protocol

When designing or improving experiments:
1. Read claims in the paper -> design experiments that test each claim
2. Ensure reproducibility: deterministic seeds, logged parameters, stated hardware
3. For multiple independent scenarios, spawn parallel sub-agents:
   ```
   Task(subagent_type="general-purpose", prompt="Run scenario X...")
   Task(subagent_type="general-purpose", prompt="Run scenario Y...")
   ```
4. Generate figures as PDF (matplotlib, seaborn, or pgfplots)
5. Format tables in LaTeX with proper highlighting (bold best results)
6. Write a brief results summary paragraph for each experiment

## Writing Agent Protocol

When drafting or revising prose:
- **Compression**: target the section length guidelines in review-criteria.md
- **Precision**: replace vague claims with quantified statements
- **Honesty**: report limitations and cases where the method is not the best
- **Transitions**: every section should connect to the next
- **Abstract**: must be self-contained (problem, gap, method, key quantitative result)
- **Contributions list**: each item must have a supporting theorem/experiment/section reference

## Layout Agent Protocol

When fixing formatting:
1. Compile the paper, capture all warnings
2. Fix overfull hbox warnings systematically:
   - Long equations: `align` -> `multline` with strategic line breaks
   - Repeated expressions: introduce abbreviations (e.g., `\Hb := \Mb^\top\Mb`)
   - Wide tables: `\footnotesize`, reduce `\tabcolsep`, abbreviate headers
   - Column vectors: use `bmatrix` instead of inline notation
   - Math spacing: `\!` for tightening, `\,` for slight separation
3. Re-compile and verify zero warnings
4. Check figure placement, page breaks, orphan lines

## Resources

- **`references/review-criteria.md`**: Full review criteria, scoring rubric, section length guidelines, and review report template. Read this before any review pass.
- **`references/agent-roles.md`**: Detailed description of each agent's role, capabilities, and inter-agent communication patterns. Read this when planning complex multi-agent workflows.
