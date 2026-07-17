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

## Workflow Preview First

Before Phase 0 Q&A, source-choice questions, or agent dispatch, the Planner must briefly show the user the intended path and then proceed. Keep this preview concise, but make the process visible:
- **Chosen mode and reason**: Mode A seed/full-paper writing, Mode B full-draft revision, or Mode C narrow task.
- **Quality target**: submit-ready, draft feedback, or quick skeleton; state what this means for review depth.
- **Next steps**: list the immediate 3-6 steps, including whether the workflow will run baseline review, related-paper calibration, paper-maturity expansion, specialist revisions, Review-Revise Loop, literature/novelty evidence, and final layout gate.
- **Automation policy**: state whether the Planner will continue automatically after review, or pause for user choices.
- **Stop conditions**: pass condition, round limit, quick-skeleton stop, review-only stop, PDF-only advisory stop, or explicit user stop.
- **Input/source status**: if only a PDF is available, say that precise source edits and true submit-ready sign-off require `.tex` / bibliography source; the PDF-only path is advisory.

Example preview for the common submit-ready revision case:
```text
Workflow preview:
- Mode B: full/near-full draft revision.
- Target: submit-ready.
- Steps: baseline review -> related-paper calibration -> maturity expansion/revision -> 4-reviewer loop -> final layout gate.
- I will continue after baseline review automatically unless you stop or limit scope.
```

## Entry Point: Phase 0 — Contextual Q&A

Before dispatching any agents, gather critical context from the user. Ask these questions upfront (use AskUserQuestion where available):

**Q1 — Starting point** (determines workflow mode):
- I have research results / ideas / notes, or only a very short seed draft → Mode A
- I have a full or near-full `.tex` / `.pdf` paper draft → Mode B
- I need a specific isolated task (fix equations, convert format, one-pass review, etc.) → Mode C

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

If Q1 answer is **Mode B**, also ask:

**Q5 — Main weakness** (multi-select, Mode B only):
- Theory is weak or incomplete
- Experiments are insufficient or missing ablations
- Writing is verbose or unclear
- Everything — run a full harsh review first

For **Mode C**, ask only the context needed for the narrow task, plus Q6 if the task affects claims, novelty, related work, or positioning.

**Q6 — Closest prior results / papers** (always ask for Mode A and Mode B unless already explicit):
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

If user skips Q&A or input is unambiguous (e.g., .tex file attached + "revise"), the Planner may auto-infer the workflow mode, but must not silently skip the core review context:
- Always ask or infer **Q2 target venue** and **Q4 quality target** before any review or revision loop. If the user refuses or asks to proceed immediately, record `unknown` or `user_skipped` in `session.yaml`.
- Always ask **Q6 closest prior results / papers** for Mode A and Mode B unless already explicit. For Mode C, ask Q6 only when the task affects claims, novelty, related work, or positioning. If the user does not provide it when needed, record `closest_prior_results: unknown` and lower the confidence of `References / Positioning` judgments rather than pretending the prior-work boundary is known.
- Q3 focus and Q5 weakness may be inferred from the request when clear, but inferred values must be written to `session.yaml` so later agents do not rely on unstated assumptions.

### Global Source Validation

Before choosing Mode A or Mode B for any existing draft, check whether editable source is available.

- **If `.tex` and bibliography/source assets are available**: proceed normally.
- **If only PDF is available**: show the PDF-only limitation in the Workflow Preview and ask whether to run an advisory PDF-only review or wait for source. Do not imply that the paper can be precisely revised or signed off as submit-ready without source.
- **If the user chooses advisory PDF-only review**: extract text and inspect the PDF, optionally run literature/novelty evidence and related-paper calibration with limited confidence, run an advisory baseline review, then stop with source-required next steps. Mark layout as `Limited / N/A`, mark source-dependent theory/evidence judgments as `limited evidence`, and do not enter the full source-editing Review-Revise Loop.
- **If the user provides source later**: rehydrate any advisory findings as context, then run the normal Mode A or Mode B workflow from source.

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
    calibration/
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
- Fresh Reviewers may receive the current round's literature/novelty evidence content from the Planner as a round-local non-history input. They must not browse `.paper-review/audit/reviews/*` to retrieve it themselves.
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
- `calibration/paper_maturity_profile.yaml`: compact guidance from 2-3 closest mature papers when the workflow is writing or substantially revising toward submit-ready quality

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
1. **Mode A — Write from Scratch or Seed Draft** → Use when there is no reviewable full paper yet: idea notes, method/results notes, a very short draft, a partial introduction, or a skeletal draft missing major paper sections. Treat the provided material as seed input and build it into a complete paper before review.
2. **Mode B — Revise Full or Near-Full Draft** → Use when an existing `.tex` / `.pdf` draft is complete enough to review as a paper, even if weak. Default goal is to revise, strengthen, and polish it toward submit-ready quality.
3. **Mode C — Narrow Isolated Task** → Use only when the user explicitly asks for a local task such as fixing one equation, rewriting one paragraph, converting format, making a skeleton from an existing draft, or doing a one-pass review. Dispatch directly to the relevant agent; run a single selected Fresh Reviewer pass afterward to check for regressions when the task changes paper content; ask user if they want a full review.
   - Choose the Mode C reviewer by task: technical/proof/experiment/equation edits use a Technical Fresh Reviewer; intro/story/writing/layout edits use a Non-Technical Fresh Reviewer; one-pass review runs the single most relevant reviewer and then stops.
   - If the user asks for a quick skeleton of an existing draft, treat it as a Mode C structural outline task and stop after the skeleton unless the user explicitly asks to continue into writing/revision.
   - If user requests full review after Mode C: first persist the raw Fresh Reviewer report under `.paper-review/audit/reviews/round-00-mode-c/`, then normalize the Mode C review into an **aggregated summary artifact**, assign stable issue IDs, write it to `.paper-review/audit/reviews/round-00-mode-c/aggregated-summary.md`, update `.paper-review/memory/review/last_round_summary.md`, `.paper-review/memory/review/issues.yaml`, and `.paper-review/memory/review/current_state.yaml`, and only then enter the standard 4-reviewer loop starting at round 1.

### Draft Completeness Triage

If the user provides a draft, the Planner must first decide whether it is seed material or a reviewable paper draft:
- Route to **Mode A** when the draft is an idea sketch, short note, partial intro, rough result log, or otherwise missing major sections such as related work, method, evidence/experiments, or conclusion.
- Route to **Mode B** when the draft has the major paper sections and a visible claim-evidence structure, even if the story, theory, experiments, or writing are weak.
- When uncertain, prefer **Mode A** if the work mainly requires completing the paper, and **Mode B** if the work mainly requires revising an already reviewable paper.

### Default Submit-Ready Revision Policy

Unless the user explicitly asks for review-only, quick skeleton, draft feedback, or a narrow isolated edit, interpret requests such as "revise", "improve", "polish", "fix this paper", "strengthen the paper", or "make it submit-ready" as `quality_target = Submit-ready`.

For submit-ready Mode B work:
- Do not stop after baseline review merely to ask which issues to address.
- Address all CRITICAL and MAJOR issues automatically, including protected Non-Technical Fresh Reviewer reader-impression issues affecting `Story / Logic` or `Writing / Structure`.
- Continue into the Review-Revise Loop until the pass condition, round limit, or an explicit user stop.

### Submit-Ready Continuation Rule

For any submit-ready workflow, the Planner should keep moving through review, expansion/revision, verification, and layout sign-off. Do not stop just because one review pass finished, one revision pass was applied, files were edited, or LaTeX compiled successfully.

Only pause for user input when the next action would change the core claim or method scope, require new experiments/resources the user has not provided, choose between incompatible paper directions, exceed the round limit, hit a real source/data blocker, or respond to an explicit user stop.

Before each stop, state the current phase, the reason stopping is necessary, and the next concrete action after the user responds.

Skip the full Review-Revise Loop only when:
- `quality_target == Quick skeleton`
- the user explicitly asks for review-only / one-pass feedback
- the user explicitly scopes the request to a narrow isolated edit
- only a PDF is available and the user chooses advisory PDF-only review

### Mandatory Full-Source Submit-Ready Pipeline

If `quality_target == Submit-ready` and editable source files are available, the Planner must execute this path in order:

1. Show Workflow Preview.
2. Run Global Source Validation.
3. Run Related-Paper Calibration using 2-3 closest mature papers.
4. Run Paper Maturity Audit against the calibration profile.
5. Execute an Initial Maturity Revision before the professional review loop:
   - expand thin sections
   - strengthen intro/story
   - add or improve related-work positioning
   - add missing citations
   - deepen method/theory/evidence explanation
   - improve figures/tables/captions/layout-facing prose where needed
   - preserve readability and story/logical flow so a technically adjacent or non-specialist reader can understand what the paper does, why it matters, what is technically strong, and how the evidence supports the claims
6. Enter the standard 4-reviewer Review-Revise Loop.
7. Continue review -> revise -> review until pass condition, round limit, explicit stop, or real blocker.
8. Run final Technical Layout Gate and PDF Aesthetic Layout Gate.

Do not skip calibration, maturity audit, initial maturity revision, or the Review-Revise Loop merely because the draft looks reasonable, compiles, or has already received one round of edits.

### Related-Paper Calibration and Paper Maturity

Before writing a new paper or making major submit-ready revisions, calibrate against 2-3 closest mature papers in the same area or venue style. This is not a full survey; it is a practical writing calibration pass to make the draft feel like a real paper rather than expanded notes.

Use the closest papers to form a compact `paper_maturity_profile`:

```yaml
paper_maturity_profile:
  anchor_papers:
    - paper:
      why_relevant:
      useful_pattern:
  expected_shape:
    story:
    section_depth:
    evidence:
    references:
    layout_density:
  gaps_to_fix:
    - ...
```

The profile should guide the Planner, Writing Agent, Experiment Agent, Theory Agent, and Layout Agent. It should cover story structure, section organization, expected depth, citation/related-work density, evidence or theorem support, figure/table use, and visual density. If network or literature search is unavailable, use the user's closest-prior answers and mark calibration confidence as limited.

For submit-ready work, treat draft-like maturity as a fix target before final prose polish. Typical maturity gaps include an overly short or generic introduction, thin related work, method sections that only sketch an idea, missing experiment setup/baseline/ablation/analysis, underdeveloped theorem assumptions or proof discussion, placeholder-like figures/tables, shallow limitations, and pages that look sparse or unbalanced versus the anchor papers.

## Mode A: Write from Scratch or Seed Draft

### Phase 0.5: Runtime State Setup
Before planning:

1. Initialize or rehydrate `.paper-review/` for this paper
2. Write or refresh `.paper-review/memory/session.yaml` with venue, focus, quality target, and the derived `review_profile`
3. If prior state exists, reload `.paper-review/memory/review/current_state.yaml`, `.paper-review/memory/review/last_round_summary.md`, and `.paper-review/memory/review/issues.yaml`
4. Use the rehydrated state as the source of truth instead of resetting in-memory loop variables
5. If the persisted state indicates planning, review, revision, or final layout is already complete or in progress, jump to the next expected phase instead of replaying earlier phases from scratch

### Phase 1: Planning
Use the Planner Agent role (see `references/agent-roles.md`).

1. Read user's input: problem description, method, results, target venue, and any seed draft / notes
2. Produce a paper outline:
   - Title, abstract sketch, section structure
   - Identify which sections need theory derivation vs. experiment design vs. prose
   - Identify what can be done in parallel
3. Create a TodoWrite plan tracking each section

If `quality_target == Quick skeleton`, stop after Phase 1. Output the title candidates, abstract sketch, contribution sketch, section outline, evidence/theory TODOs, and next-step plan. Do not run parallel content generation, review-revise loop, literature/novelty gate, or final layout gate unless the user explicitly asks to continue.

### Phase 1.5: Related-Paper Calibration and Maturity Profile

For submit-ready or full-draft targets, run the Related-Paper Calibration before drafting substantial prose. Select 2-3 closest mature papers from Q6, the bibliography, or a quick literature search. Read them at whole-paper level for writing and presentation patterns, not only for novelty.

Save `paper_maturity_profile` under `.paper-review/audit/calibration/paper_maturity_profile.yaml` and mirror the latest copy under `.paper-review/memory/calibration/paper_maturity_profile.yaml`. Use it to decide whether the paper needs expansion tasks before polish tasks.

### Phase 2: Parallel Content Generation
Dispatch agents in parallel where possible:

- **Theory Agent** (Task subagent): Derive all mathematical content — model, algorithm, theorems, proofs, bounds. Output LaTeX-ready blocks.
- **Experiment Agent** (Task subagent, may spawn parallel sub-agents per scenario): Design benchmarks, write experiment scripts, generate figures and tables. Output scripts + LaTeX figure/table code.
- **Writing Agent**: Draft related work, introduction framing, conclusion. Can begin once outline is set.

Use `paper_maturity_profile` to avoid producing a thin draft: expand underdeveloped sections with motivation, mechanism, evidence, theorem/proof context, related-work positioning, and analysis before doing compression or style polish.

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
  The Planner passes the evidence content directly to reviewers as round-local non-history input; Fresh Reviewers must not read `.paper-review/audit/reviews/*` themselves.

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
    For submit-ready work, also require no unresolved paper-maturity gaps that make the draft still feel skeletal, thin, citation-light, or visually unfinished.
    if PASS and maturity is acceptable: break

  if quality_target == "Draft for feedback" and round >= 2:
    Present aggregated issues to user; break (do not require pass condition)

  if round >= MAX_ROUNDS:
    Present current state to user: scores per reviewer, top unresolved issues
    Warn if any reviewer has median < 4 or any core dimension below 3
    Ask: "10 rounds completed without full convergence. Continue 10 more rounds, or stop and report the current draft as not yet submit-ready?"
    if continue: MAX_ROUNDS += 10  (do not reset round counter; preserve last_round_summary)
    else: break

  Planner creates revision task list from aggregated issues and paper-maturity gaps
  Planner writes the current fix plan to `.paper-review/memory/review/revision_plan.yaml`
  Planner logs major accept/reject decisions to `.paper-review/audit/decision-log.md`
  For submit-ready work, expansion and evidence-building tasks come before final polish when sections are visibly underdeveloped.
  Dispatch fixes to appropriate agent(s)
  Merge revisions
  Append completed fixes to `.paper-review/audit/revision-log.md`
  last_round_summary = aggregated summary report from this round
```

### Phase 4: Final Layout Gate
- Layout Agent: compile, check warnings, fix overflow/formatting
- Planner or final reviewer: verify both the Technical Layout Gate and PDF Aesthetic Layout Gate on the compiled PDF
- Technical checks: references resolve, equations fit, figures/tables readable, bibliography/style consistent, venue formatting respected
- Aesthetic checks: section lengths feel balanced, pages are neither sparse nor crowded, formula/text density is readable, figures/tables/captions support skimming, and the final pages look like a mature submission rather than notes squeezed into a template
- If either layout gate fails but content is intact, fix source/content/layout and re-run the gate
- If the layout fixes introduce content regressions, return to Phase 3

## Mode B: Revise Existing Paper

Mode B assumes the draft is full or near-full enough to review as a paper. If the draft is skeletal or mainly seed material, route to Mode A instead.

### Phase 0.5: Input Validation
Use the Global Source Validation result. If only a PDF is available and the user chooses advisory review, run the explicit advisory PDF-only branch and stop with source-required next steps. If source is available, proceed normally.

### Phase 1: Baseline Review
For full-source submit-ready Mode B, the Mandatory Full-Source Submit-Ready Pipeline overrides the default baseline-first flow. Run Related-Paper Calibration, Paper Maturity Audit, and one Initial Maturity Revision before any 4-reviewer baseline or iterative review pass. The first 4-reviewer pass after this initial revision becomes the baseline for the Review-Revise Loop.

1. Read the existing .tex source (and compiled PDF if available)
2. Initialize or rehydrate `.paper-review/` state for this paper
3. Write or refresh `.paper-review/memory/session.yaml` with venue, focus, quality target, and the derived `review_profile`
4. If prior state exists, reload `.paper-review/memory/review/current_state.yaml`, `.paper-review/memory/review/last_round_summary.md`, and `.paper-review/memory/review/issues.yaml` before deciding whether to start a new baseline round
5. If the persisted state indicates baseline review or later phases are already complete, continue from the recorded next expected phase instead of re-running baseline review
6. Run the References / Novelty Review Gate unless the user explicitly requested a no-search review.
7. Run Related-Paper Calibration using 2-3 closest mature papers and save `paper_maturity_profile`.
8. Dispatch **4 Review Agents in parallel** for a full baseline assessment. At baseline, none receive a prior round summary; use three technically oriented fresh reviewers and one Non-Technical Fresh Reviewer focused on readability, storyline, contribution clarity, and first-impression credibility.
9. Persist all 4 raw baseline reports under `.paper-review/audit/reviews/round-00-baseline/`
10. Aggregate into a single baseline `last_round_summary`
11. Write the baseline aggregated summary to `.paper-review/audit/reviews/round-00-baseline/aggregated-summary.md`
12. Update `.paper-review/memory/review/last_round_summary.md`, `.paper-review/memory/review/issues.yaml`, and `.paper-review/memory/review/current_state.yaml`
13. Present the aggregated baseline review to the user. If `quality_target == Submit-ready`, continue directly into revision planning and execution; pause here only for review-only, draft-feedback, user-limited scope, PDF-only advisory mode, or explicit user stop.

### Phase 2: Revision Planning
1. Categorize issues from the review by severity: CRITICAL > MAJOR > MINOR
2. Add maturity gaps from `paper_maturity_profile`, especially thin sections, weak citations, missing evidence/theorem support, shallow analysis, and visually unfinished pages
3. Group by type: theory, experiments, writing, layout
4. If `quality_target == Submit-ready`, address all CRITICAL and MAJOR issues automatically, including protected reader-impression issues and maturity gaps that make the draft feel non-submit-ready. Ask user which issues to address only for draft feedback, review-only, PDF-only advisory mode, or user-limited scope.
5. Create a prioritized revision plan via TodoWrite
6. Put expansion/evidence-building tasks before final polish when the draft is short, content-light, citation-light, or analysis-light
7. Write the active fix list to `.paper-review/memory/review/revision_plan.yaml`
8. Log any user-approved deferrals or scope cuts to `.paper-review/audit/decision-log.md`

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
round = rehydrated_round or 0
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
3. Save the bundle under `.paper-review/audit/reviews/round-XX/literature-novelty-evidence.md` for audit, and pass its content directly to reviewers as round-local non-history input.
4. Reviewers must base the `References / Positioning` score on both the draft and this evidence bundle, not only on the draft bibliography. Fresh Reviewers may use the Planner-provided evidence content, but must not browse audit history or prior review artifacts.
5. If fresh search is impossible, mark the evidence basis as `no fresh literature search`; in that case, `References / Positioning` should usually be capped at 3 unless the user explicitly requested an internal-only review.

The evidence bundle must include:
- search queries or paper-discovery routes used
- closest-prior table with citation, core result/method, formal overlap, substantive overlap, and remaining distinction
- novelty-check verdict (`clear`, `partial`, `weak`, or `unclear`)
- missing citations and positioning fixes

### Related-Paper Calibration Gate

For Mode A and submit-ready Mode B, the Planner should use 2-3 closest mature papers as anchors before substantial writing or revision. This gate is about paper-writing maturity, not only novelty.

Read the anchor papers at whole-paper level and summarize what the current paper should learn from them: how the introduction builds the problem/gap, how related work positions the contribution, how much method/theory detail is expected, how experiments or proofs are explained, how figures/tables support the story, how many references feel normal for the venue, and how dense or balanced the PDF pages look.

The result is the `paper_maturity_profile`. Specialist agents should use this profile as taste guidance. Do not turn it into a rigid template; use it to decide whether the draft feels underwritten, under-cited, under-evidenced, or visually immature.

Scoring dimensions: Story / Logic, Theory / Rigor, Evidence, Writing / Structure, References / Positioning. Each 1-5.

**Pass condition**: for each of the 4 reviewers individually, both must hold: (1) median score across the 5 core dimensions >= 4, and (2) no core dimension scored below 3. All 4 reviewers must pass this bar before final layout sign-off.

### Reviewer Types

Use the detailed reviewer definitions in `references/agent-roles.md` and scoring/comment guidance in `references/review-criteria.md`. The operational rule is:
- Full rounds use 2 Historical Reviewers, 1 Technical Fresh Reviewer, and 1 Non-Technical Fresh Reviewer.
- Historical Reviewers receive only the previous aggregated summary, not raw review history.
- Fresh Reviewers receive no prior review memory or audit history, but may receive the current round's literature/novelty evidence directly from the Planner.
- The Non-Technical Fresh Reviewer writes subjective reader-impression comments first and then translates that impression into the required 1-5 score table, with the strongest signal in `Story / Logic`.

## Theory Agent Protocol

Use `references/agent-roles.md` for full role details. In brief: strengthen model definitions, assumptions, theorems, proofs, bounds, notation consistency, and closest-prior distinctions. Output LaTeX-ready theorem/proof blocks plus the surrounding explanatory text needed to make the theory feel motivated and credible.

## Experiment Agent Protocol

Use `references/agent-roles.md` for full role details. In brief: map each claim to evidence, improve baselines/ablations/setup/analysis, ensure reproducibility, generate publication-quality figures/tables, and write the result interpretation needed for the paper to feel mature.

## Writing Agent Protocol

When drafting or revising prose:
- **Maturity before polish**: do not compress underdeveloped sections. If a section lacks motivation, mechanism, evidence, theorem/proof context, citation positioning, or analysis, expand it before polishing style.
- **Use calibration**: use `paper_maturity_profile` to judge whether section depth, citation density, evidence discussion, and page-level presentation feel comparable to the anchor papers.
- **Compression**: target the section length guidelines in review-criteria.md without removing the logic that helps the reader feel oriented.
- **Precision**: when a claim feels vague, revise toward concrete mechanism, condition, evidence, number, or limitation.
- **Honesty**: report limitations and cases where the method is not the best
- **Transitions**: make section-to-section movement feel natural rather than abrupt.
- **Abstract**: leave the reader with a concrete sense of the problem, gap, method idea, and key result.
- **Abstract progression**: move from background/motivation -> what the paper does -> how it does it -> evidence and trade-offs, so the reader understands the problem before the method.
- **Introduction progression**: move from existing setting -> difference from prior work -> why decisions must be made before measurement -> the proposed solution and contributions.
- **Contributions list**: make each item feel like a real technical contribution, not a broad promise.
- **Positioning**: make the closest-prior boundary feel honest, specific, and easy to understand.
- **Subject ownership and voice**: use `we` only for author actions such as studying, formulating, choosing, obtaining, or evaluating. Let background facts, physical mechanisms, assumptions, and results take their natural subjects instead of mechanically rewriting objective statements into first person.
- **Sentence rhythm**: alternate subjects across paragraphs; keep references clear; vary sentence length and structure; avoid chains of nominalized phrases or several consecutive sentences with the same opening pattern.
- **Preserve substance**: keep original equations, experimental numbers, comparison boundaries, and limitations intact unless the task explicitly asks to change them. Adjust narrative ownership, sentence rhythm, and transitions without weakening technical content.
- **Reader impression revision**: treat Non-Technical Fresh Reviewer comments as signals about reader experience, not binary defects.
- **Storyline smoothing**: shape the Abstract, Introduction, section openings, and conclusion so the reader feels naturally carried from problem to gap, from gap to key idea, and from key idea to mechanism and evidence.
- **Concept onboarding**: introduce concepts, acronyms, notation, assumptions, metrics, and prior-work names at the moment where they help the reader, before they become a source of friction.
- **Slogan-to-substance rewrite**: when prose feels like it is selling, including generic `we show` phrasing or defensive reviewer-response language, revise toward the concrete action, mechanism, evidence, condition, number, limitation, or design choice that makes the claim feel real.
- **Layout-facing prose**: write contribution bullets, captions, section openings, and overview-figure text so the page itself helps a skim reader follow the story.
- **Human taste pass**: after revising front matter, reread it as a tired but fair reviewer and revise anything that feels inflated, vague, rushed, or hard to trust.

## Layout Agent Protocol

When fixing formatting:
1. Compile the paper, capture all warnings
2. Run the Technical Layout Gate: unresolved references, serious overfull boxes, equation overflow, table width, figure readability, bibliography style, and venue format
3. Fix overfull hbox warnings systematically:
   - Long equations: `align` -> `multline` with strategic line breaks
   - Repeated expressions: introduce abbreviations (e.g., `\Hb := \Mb^\top\Mb`)
   - Wide tables: `\footnotesize`, reduce `\tabcolsep`, abbreviate headers
   - Column vectors: use `bmatrix` instead of inline notation
   - Math spacing: `\!` for tightening, `\,` for slight separation
4. Re-compile and verify serious technical layout problems are gone
5. Run the PDF Aesthetic Layout Gate by looking at the compiled PDF as a reviewer would: section balance, text/formula density, figure/table placement, caption usefulness, white space, crowded pages, sparse pages, and final-page polish
6. If the aesthetic problem is caused by missing content rather than formatting, return the issue to the Writing Agent or relevant specialist instead of only shrinking or spacing the page

## Resources

- **`references/review-criteria.md`**: Full review criteria, scoring rubric, section length guidelines, and review report template. Read this before any review pass.
- **`references/agent-roles.md`**: Detailed description of each agent's role, capabilities, and inter-agent communication patterns. Read this when planning complex multi-agent workflows.
