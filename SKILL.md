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

**Q6 — Closest prior results / papers** (always ask unless already explicit):
- What are the closest prior papers or theoretical results?
- Which theorem / method / setting / experimental claim is closest to yours?
- What is formally different?
- Why does that difference matter?
- What limitations remain relative to the prior work?

After collecting answers, inject a **context block** into every subsequent agent call:
```
context:
  venue: [answer to Q2]
  page_limit: [derived from venue]
  focus: [answer to Q3]
  quality_target: [answer to Q4]
  closest_prior_results:
    - [paper/result 1]
    - [paper/result 2]
  claimed_difference: [formal and substantive difference from prior work]
  difference_significance: [why the difference matters]
  known_limitations_vs_prior: [remaining limitations or narrower scope]
```

Then derive and inject a **review profile** from the venue and paper focus:
```
review_profile:
  core_dimensions:
    - story_logic
    - theory_rigor
    - evidence
    - writing_structure
    - references_positioning
  focus_profile: [theory-heavy | experiment-heavy | balanced | application-case-study]
  venue_profile: [ml-conference | computer-vision | journal | workshop-arxiv]
  layout_gate: required for final acceptance when the source can be compiled
```

Profile rules:
- `Theory-heavy`: emphasize `theory_rigor` and `story_logic`; `evidence` should validate claims and assumptions, but need not be benchmark-heavy.
- `Experiment-heavy`: emphasize `evidence` and `story_logic`; `theory_rigor` still checks method soundness and honest scope.
- `Balanced theory + experiments`: emphasize alignment between theorem statements, empirical claims, and conclusions.
- `Application / case study`: emphasize problem realism, evidence quality, and references within the application domain.
- `ML conference`: prioritize contribution clarity, theory/evidence alignment, and honest comparison to strong prior work.
- `Computer vision`: strengthen empirical and visual evidence expectations.
- `IEEE / Springer journal`: strengthen completeness, literature coverage, and mature discussion of limitations.
- `Workshop / arXiv`: allow exploratory evidence, but require explicit calibration of claims and limitations.

If user skips Q&A or input is unambiguous (e.g., .tex file attached + "revise"), auto-infer mode and skip relevant questions. Always ask Q2 and Q4 if not clear.

## Runtime Local State and Audit Protocol

When this skill is used on a real paper workspace, maintain a local runtime state directory under the paper root:

```text
.paper-review/
  memory/
    session.yaml
    review/
      last_round_summary.md
      issues.yaml
      revision_plan.yaml
      current_state.yaml
  audit/
    reviews/
      round-00-baseline/
      round-01/
      round-02/
      ...
    revision-log.md
    decision-log.md
```

### `memory/` is for agent runtime state

Purpose: compact local memory for agents so work can resume across turns without replaying the whole chat.

Rules:
- `memory/` is machine-oriented and may be overwritten as the current state changes.
- Historical Reviewers may read `memory/session.yaml` and `memory/review/last_round_summary.md` only.
- Fresh Reviewers must not read prior review memory or audit files.
- Fresh Reviewers may read `memory/session.yaml` only for non-review-history metadata such as venue, focus, quality target, and `review_profile`.
- Specialist agents may read `memory/review/issues.yaml` and `memory/review/revision_plan.yaml` when performing revisions.
- `memory/review/last_round_summary.md` must contain only the previous round's aggregated review state, not the full review archive.
- On every new invocation of the skill, the Planner must first check whether `.paper-review/memory/` already exists for the current paper and rehydrate state from disk before creating any new rounds.
- `memory/session.yaml` is the source of truth for the persisted `review_profile` after initialization; later reviewer calls should read the profile from this file, not from stale in-memory prompt state.

Minimum contents:
- `session.yaml`: paper path, venue, page limit, focus, quality target, review profile, current round, workflow mode
- `review/last_round_summary.md`: previous round aggregated summary with scores, open issues, resolved issues, and pass/fail
- `review/issues.yaml`: stable issue ledger with issue IDs, severity, status, first seen round, last seen round, and owner
- `review/revision_plan.yaml`: current actionable fix list for specialist agents
- `review/current_state.yaml`: lightweight execution state with at least `workflow_mode`, `phase`, `round`, `round_type`, `latest_summary_path`, and `loop_status`

Resume semantics:
- `current_state.yaml` must record the last completed phase and the next expected phase.
- `round` in `current_state.yaml` means the latest completed review round number, not the next round to be created.
- On resume, the Planner must branch from `current_state.yaml` and continue from the next expected phase instead of replaying earlier phases unconditionally.
- If the persisted state and on-disk artifacts disagree, the Planner should stop and reconcile before proceeding.

### `audit/` is for human inspection

Purpose: preserve a complete, human-auditable record of the review and revision process.

Rules:
- `audit/` is append-oriented; do not silently overwrite old round artifacts.
- Store every raw reviewer report under `audit/reviews/round-XX/`.
- Store one aggregated summary per round under the same round directory.
- Store revision actions and key planning decisions in `audit/revision-log.md` and `audit/decision-log.md`.
- Audit artifacts are not the default input to Historical Reviewers; they exist for humans and for Planner-level traceability.

Required audit contents per round:
- all raw reviewer reports produced for that round
- per-reviewer score tables
- per-reviewer median and pass/fail
- aggregated round score table
- final layout gate status if evaluated
- resolved, unresolved, and regressed issues
- stable issue IDs carried across rounds

Round-type rules:
- Standard full-review rounds must contain 4 raw reviewer reports
- Single-review preflight rounds such as Mode C may contain fewer raw reviewer reports, but the round type must be labeled explicitly in the audit artifact

## Mode Dispatch

After Phase 0:
1. **Mode A** (no draft) → Write from Scratch workflow below
2. **Mode B** (existing draft) → Revise Existing Paper workflow below
3. **Mode C** (specific task) → Dispatch directly to the relevant agent; run a single Fresh Reviewer pass afterward to check for regressions; ask user if they want a full review.
   - If user requests full review after Mode C: first persist the raw Fresh Reviewer report under `.paper-review/audit/reviews/round-00-mode-c/`, then normalize the Mode C review into an **aggregated summary artifact**, assign stable issue IDs, write it to `.paper-review/audit/reviews/round-00-mode-c/aggregated-summary.md`, update `.paper-review/memory/review/last_round_summary.md`, `.paper-review/memory/review/issues.yaml`, and `.paper-review/memory/review/current_state.yaml`, and only then enter the standard 4-reviewer loop starting at round 1.

## Mode A: Write from Scratch

### Phase 0.5: Runtime State Setup
Before planning:

1. Initialize or rehydrate `.paper-review/` for this paper
2. Write or refresh `.paper-review/memory/session.yaml` with venue, focus, quality target, and the derived `review_profile`
3. If prior state exists, reload `.paper-review/memory/review/current_state.yaml`, `.paper-review/memory/review/last_round_summary.md`, and `.paper-review/memory/review/issues.yaml`
4. Use the rehydrated state as the source of truth instead of resetting in-memory loop variables
5. If the persisted state indicates planning, review, revision, or final layout is already complete or in progress, jump to the next expected phase instead of replaying earlier phases from scratch

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
last_round_summary = rehydrated_last_round_summary or None
round = rehydrated_round or 0   # latest completed round from current_state.yaml
MAX_ROUNDS = 10

loop:
  round += 1

  Use the latest References / Novelty evidence bundle for `References / Positioning` scoring.
  Re-run the References / Novelty Review Gate only if there is no prior bundle for this paper, the claims / positioning / related work changed materially, or the user explicitly asks for a fresh search.
  Otherwise, reuse the latest saved bundle and mark the evidence basis as `reused literature-novelty evidence`.

  Dispatch 4 Review Agents in parallel:
    - Historical Reviewer 1 (draft + memory/session.yaml + last_round_summary + literature/novelty evidence bundle)
    - Historical Reviewer 2 (draft + memory/session.yaml + last_round_summary + literature/novelty evidence bundle)
    - Technical Fresh Reviewer (draft + memory/session.yaml non-history metadata + literature/novelty evidence bundle)
    - Non-Technical Fresh Reviewer (draft + memory/session.yaml non-history metadata + literature/novelty evidence bundle; evaluate as a busy, technically adjacent reviewer who may not follow all technical details but must understand the paper's story, contribution, and why the method is credible)

  Aggregate reports:
    - Persist all 4 raw reviewer reports to `.paper-review/audit/reviews/round-XX/`
    - Collect all CRITICAL / MAJOR / MINOR issues across 4 reports
    - Merge duplicate issues, weight by number of reviewers who flagged them
    - Issues flagged as "Unresolved from last round" by Historical Reviewers escalate one severity level
    - Assign or reuse stable issue IDs in `.paper-review/memory/review/issues.yaml`
    - Produce a single aggregated summary report with per-reviewer core-dimension scores, median, pass/fail, and resolved/unresolved/regressed issues (see Aggregation Rules in review-criteria.md)
    - Write the aggregated summary to `.paper-review/audit/reviews/round-XX/aggregated-summary.md`
    - Update `.paper-review/memory/review/last_round_summary.md`
    - Update `.paper-review/memory/review/current_state.yaml`

  Planner checks pass condition:
    PASS if: for each of the 4 reviewers, median across the 5 core dimensions >= 4 AND no core dimension below 3
    if PASS: break

  if quality_target == "Draft for feedback" and round >= 2:
    Present aggregated issues to user; break (do not require pass condition)

  if round >= MAX_ROUNDS:
    Present current state to user: scores per reviewer, top unresolved issues
    Warn if any reviewer has median < 4 or any core dimension below 3
    Ask: "10 rounds completed without full convergence. Continue (10 more rounds) or accept current draft?"
    if continue: MAX_ROUNDS += 10  (do not reset round counter; preserve last_round_summary)
    else: break

  Planner creates revision task list from aggregated issues
  Planner writes the current fix plan to `.paper-review/memory/review/revision_plan.yaml`
  Planner logs major accept/reject decisions to `.paper-review/audit/decision-log.md`
  Dispatch fixes to appropriate agent(s)
  Merge revisions
  Append completed fixes to `.paper-review/audit/revision-log.md`
  last_round_summary = aggregated summary report from this round
```

### Phase 4: Final Layout Gate
- Layout Agent: compile, check warnings, fix overflow/formatting
- Planner or final reviewer: verify the layout gate on the compiled PDF
- Required checks: references resolve, equations fit, figures/tables readable, venue formatting respected
- If the layout gate fails but content is intact, fix layout and re-run the gate
- If the layout fixes introduce content regressions, return to Phase 3

## Mode B: Revise Existing Paper

### Phase 0.5: Input Validation
Before proceeding, verify the input format:

- **If .tex source available**: proceed normally.
- **If PDF only (no .tex)**:
  - Warn the user: "Only a PDF was provided. Revisions will be limited — I can extract text for review, but cannot make precise LaTeX edits without the source file."
  - Ask: "Do you want to (a) proceed with text-extraction-based review only, or (b) provide the .tex source?"
  - If (a): extract text via `pdftotext`, run review-only workflow (no Writing/Theory/Experiment agents), mark the final layout gate as `N/A`, treat theory/evidence findings that require source-level inspection as `limited evidence` rather than pretending full verification, and label any overall pass/fail as advisory only rather than equivalent to a full-source review.
  - If (b): wait for .tex before proceeding.

### Phase 1: Baseline Review
1. Read the existing .tex source (and compiled PDF if available)
2. Initialize or rehydrate `.paper-review/` state for this paper
3. Write or refresh `.paper-review/memory/session.yaml` with venue, focus, quality target, and the derived `review_profile`
4. If prior state exists, reload `.paper-review/memory/review/current_state.yaml`, `.paper-review/memory/review/last_round_summary.md`, and `.paper-review/memory/review/issues.yaml` before deciding whether to start a new baseline round
5. If the persisted state indicates baseline review or later phases are already complete, continue from the recorded next expected phase instead of re-running baseline review
6. Run the References / Novelty Review Gate unless the user explicitly requested a no-search review.
7. Dispatch **4 Review Agents in parallel** for a full baseline assessment. At baseline, none receive a prior round summary; use three technically oriented fresh reviewers and one Non-Technical Fresh Reviewer focused on readability, storyline, contribution clarity, and first-impression credibility.
8. Persist all 4 raw baseline reports under `.paper-review/audit/reviews/round-00-baseline/`
9. Aggregate into a single baseline `last_round_summary`
10. Write the baseline aggregated summary to `.paper-review/audit/reviews/round-00-baseline/aggregated-summary.md`
11. Update `.paper-review/memory/review/last_round_summary.md`, `.paper-review/memory/review/issues.yaml`, and `.paper-review/memory/review/current_state.yaml`
12. Present the aggregated baseline review to the user

### Phase 2: Revision Planning
1. Categorize issues from the review by severity: CRITICAL > MAJOR > MINOR
2. Group by type: theory, experiments, writing, layout
3. Ask user which issues to address (or address all if user says "fix everything")
4. Create a prioritized revision plan via TodoWrite
5. Write the active fix list to `.paper-review/memory/review/revision_plan.yaml`
6. Log any user-approved deferrals or scope cuts to `.paper-review/audit/decision-log.md`

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

### Phase 5: Final Layout Gate
- Run the same final layout gate used in Mode A
- If formatting-only fixes are needed, apply them and re-run the gate
- If layout fixes change content or claims, return to Phase 4

## Review Agent Protocol

Read `references/review-criteria.md` for full criteria and report template. Key principles:

- **Be harsh**: assume the reader is an expert reviewer at a top venue
- **Be specific**: cite exact equations, figures, tables, line numbers
- **Check every claim**: compare prose claims against actual numbers in tables
- **Check balance**: section lengths should match guidelines in review-criteria.md
- **Check positioning**: compare the paper against the closest prior results and judge whether the distinction is clear and honestly discussed
- **Check layout only as a gate note during iterative review**: serious layout blockers should be flagged, but layout is not a core scored dimension during the main loop
- **Never rubber-stamp**: always identify at least 3 items to improve

### References / Novelty Review Gate

For full review rounds where `References / Positioning` is scored, the Planner must ensure a literature/novelty evidence bundle is available. Run the gate at baseline, when no prior bundle exists, when claims / positioning / related work changed materially, or when the user explicitly asks for a fresh search. For ordinary revision rounds that only polish writing, layout, or minor exposition, reuse the latest bundle and mark the evidence basis as `reused literature-novelty evidence`.

1. Use the `research-lit` skill to search for related papers and organize the closest prior work by theme.
2. Use the `novelty-check` skill to test whether the claimed contribution is already present in recent or closest literature.
3. Save the bundle under `.paper-review/audit/reviews/round-XX/literature-novelty-evidence.md` and summarize it in the aggregated review.
4. Reviewers must base the `References / Positioning` score on both the draft and this evidence bundle, not only on the draft bibliography.
5. If fresh search is impossible, mark the evidence basis as `no fresh literature search`; in that case, `References / Positioning` should usually be capped at 3 unless the user explicitly requested an internal-only review.

The evidence bundle must include:
- search queries or paper-discovery routes used
- closest-prior table with citation, core result/method, formal overlap, substantive overlap, and remaining distinction
- novelty-check verdict (`clear`, `partial`, `weak`, or `unclear`)
- missing citations and positioning fixes

Scoring dimensions: Story / Logic, Theory / Rigor, Evidence, Writing / Structure, References / Positioning. Each 1-5.

**Pass condition**: for each of the 4 reviewers individually, both must hold: (1) median score across the 5 core dimensions >= 4, and (2) no core dimension scored below 3. All 4 reviewers must pass this bar before final layout sign-off.

### Reviewer Types

**Historical Reviewer** (2 instances per round):
- Receives the current draft, `memory/session.yaml`, and the **aggregated summary report from the previous round only** (not full history of all rounds)
- Focus: verify that issues flagged in the previous round are genuinely resolved, not superficially patched
- Must explicitly list any unresolved issues from the previous round in the `Unresolved from last round` section of the report template
- Must not read raw reviewer reports from older rounds unless the Planner explicitly overrides this for debugging

**Technical Fresh Reviewer** (1 instance per round):
- Receives the current draft plus `memory/session.yaml` metadata needed for the active `review_profile`, but no prior review context
- Focus: unbiased detection of new or overlooked issues
- Provides an independent perspective uncorrupted by anchoring to previous feedback
- Must not read `.paper-review/memory/review/*` or `.paper-review/audit/reviews/*`
- In PDF-only review mode, may still score the 5 core dimensions, but must explicitly mark any theory/evidence judgment that cannot be checked from the available artifact as `limited evidence`
- In PDF-only review mode, any overall pass/fail judgment is advisory only and must not be treated as equivalent to a full-source review pass

**Non-Technical Fresh Reviewer** (1 instance per round):
- Receives the same non-history inputs as the Technical Fresh Reviewer, but reviews as a busy, technically adjacent program-committee reviewer who may not fully follow the proof, derivation, or implementation details.
- Primary focus: Abstract, Introduction, contribution list, overview figure, section flow, visual first impression, and whether the paper's value is legible without deep technical parsing.
- The main score signal should be reflected in `Story / Logic`: can a non-specialist reviewer understand the problem, why it matters, what the gap is, what the key idea is, why the technique seems strong, and what evidence supports the claims?
- May still fill the other four core dimensions, but should mark technical judgments as surface-level or limited when they depend on details the reviewer did not deeply verify.
- The report should preserve both modes: write qualitative, reader-impression comments first, then translate that impression into the required 1-5 core-dimension scores. The score should be a compact summary of the felt clarity, trust, and friction described in the comments, not a replacement for them.
- Comments should read like a human reader-impression report, not a binary checklist: describe where the Abstract, Introduction, contribution list, and main-body reading path feel clear, confusing, concrete, vague, persuasive, or over-sold.
- While reading, note concepts, acronyms, notation, method names, assumptions, metrics, or prior-work references that create friction or make the reviewer feel they are missing context.
- When claims such as "novel framework", "effective", "robust", "general", "principled", "comprehensive", or "significant improvement" feel slogan-like, explain what makes them feel unsupported and what kind of concrete mechanism, condition, number, theorem, experiment, or implementation detail would make them feel real.
- Comments are most useful when they surface storyline breaks, vague or generic contributions, missing intuition, overclaiming, confusing section order, unclear figures/captions, and layout issues that reduce trust.
- Must not read `.paper-review/memory/review/*` or `.paper-review/audit/reviews/*`.
- In PDF-only review mode, the same advisory-only and `limited evidence` rules apply.

## Theory Agent Protocol

When strengthening theory:
1. Read the full method section to understand the mathematical framework
2. Identify opportunities: missing proofs, corollaries, bounds, convergence guarantees
3. For each new result: state assumptions precisely, write a complete proof, add connecting text
4. Verify notation consistency with the rest of the paper
5. If the result is close to prior work, explicitly compare assumptions, guarantees, scope, proof technique, and remaining limitations versus the closest prior results
6. Output: LaTeX theorem/proof environments ready for insertion, plus a short "difference from prior results" discussion when relevant

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
- **Compression**: target the section length guidelines in review-criteria.md without removing the logic that helps the reader feel oriented.
- **Precision**: when a claim feels vague, revise toward concrete mechanism, condition, evidence, number, or limitation.
- **Honesty**: report limitations and cases where the method is not the best
- **Transitions**: make section-to-section movement feel natural rather than abrupt.
- **Abstract**: leave the reader with a concrete sense of the problem, gap, method idea, and key result.
- **Contributions list**: make each item feel like a real technical contribution, not a broad promise.
- **Positioning**: make the closest-prior boundary feel honest, specific, and easy to understand.
- **Reader impression revision**: treat Non-Technical Fresh Reviewer comments as signals about reader experience, not binary defects.
- **Storyline smoothing**: shape the Abstract, Introduction, section openings, and conclusion so the reader feels naturally carried from problem to gap, from gap to key idea, and from key idea to mechanism and evidence.
- **Concept onboarding**: introduce concepts, acronyms, notation, assumptions, metrics, and prior-work names at the moment where they help the reader, before they become a source of friction.
- **Slogan-to-substance rewrite**: when prose feels like it is selling, revise toward the mechanism, evidence, condition, limitation, or design choice that makes the claim feel real.
- **Layout-facing prose**: write contribution bullets, captions, section openings, and overview-figure text so the page itself helps a skim reader follow the story.
- **Human taste pass**: after revising front matter, reread it as a tired but fair reviewer and revise anything that feels inflated, vague, rushed, or hard to trust.

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
