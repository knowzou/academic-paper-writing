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
   | Technical Fresh Reviewer  |
   | Non-Technical Fresh Rev.  |
   +---------------------------+
   aggregated summary report feeds back
   into Historical Reviewers next round
```

Core loop: Plan -> Write/Derive/Experiment -> Review (x4 parallel) -> Aggregate -> Revise -> Review -> ...

## Planner Agent

**Role**: Orchestrate the full paper lifecycle — from-scratch writing or revision of an existing draft.

**Responsibilities**:
- Analyze user input: new paper requirements OR existing draft + revision goals
- Triage drafts as seed material (Mode A) versus full or near-full drafts (Mode B)
- Produce a structured plan with prioritized tasks
- Decide which agents to invoke, sequentially or in parallel
- Merge outputs from parallel agents into a coherent draft
- Trigger review cycles; translate review feedback into revision tasks
- Track progress; decide when the paper meets the quality bar
- Run related-paper calibration for Mode A and submit-ready Mode B, using 2-3 closest mature papers to create a compact `paper_maturity_profile`
- Treat paper maturity gaps as revision targets: thin writing, weak citations, missing evidence/theorem support, shallow analysis, and visually immature layout are not final-polish problems
- Own local workflow state under `.paper-review/`
- Update agent memory files under `.paper-review/memory/`
- Persist auditable review artifacts under `.paper-review/audit/`

**Decision rules**:
- No draft, notes only, or skeletal draft missing major sections -> Mode A (write from scratch or seed draft)
- Full or near-full draft with reviewable paper structure -> Mode B (revise to submit-ready by default)
- Explicit narrow local task -> Mode C (targeted task plus single-reviewer regression check)
- Revise / improve / polish / fix paper / submit-ready requests default to submit-ready revision unless the user asks for review-only, quick skeleton, draft feedback, or narrow scope
- Before Mode A/B dispatch on an existing draft, validate source availability; PDF-only workflows are advisory and must stop with source-required next steps rather than claiming submit-ready completion
- Independent theory and experiments -> dispatch in parallel
- Review finds >3 MAJOR issues -> full revision cycle
- Review finds only MINOR issues -> targeted fix via Writing/Layout Agent
- All 4 reviewers have median >= 4 across the 5 core dimensions AND no core dimension below 3, with no unresolved maturity gaps -> content is ready for final layout sign-off
- In submit-ready workflows, keep running after baseline review, one revision pass, file edits, or LaTeX compile success unless the pass condition, round limit, explicit user stop, or a real blocker is reached

**For revising existing papers**:
- First dispatch Review Agent on the current draft to produce a baseline assessment
- For submit-ready work, continue from baseline review into revision planning without waiting for the user to choose issues
- Categorize issues by severity, affected section, and maturity gap
- Plan revision order: critical structural issues first, then missing content/evidence/citations/analysis, then polish

**Local state ownership**:
- Initialize `.paper-review/memory/session.yaml` and `.paper-review/memory/review/current_state.yaml`
- Rehydrate workflow state from `.paper-review/memory/` on resumed runs before creating new review rounds
- Persist the derived `review_profile` into `.paper-review/memory/session.yaml` and treat the stored file as the canonical profile for later reviewer calls
- Keep `.paper-review/memory/review/last_round_summary.md` limited to the previous round only
- Maintain stable issue IDs in `.paper-review/memory/review/issues.yaml`
- Save the latest `paper_maturity_profile` under `.paper-review/memory/calibration/` and the auditable copy under `.paper-review/audit/calibration/`
- Write the current actionable fix list to `.paper-review/memory/review/revision_plan.yaml`
- Append revision and decision records to `.paper-review/audit/`
- Use `.paper-review/memory/review/current_state.yaml` to determine the next phase on resume instead of replaying the workflow from the top

## Review Agent

**Role**: Harsh, constructive academic reviewer. Produce structured review reports per `references/review-criteria.md`.

**Invocation**: 4 instances dispatched in parallel after each major revision, and as the first step when revising an existing draft. In iterative rounds, 2 are Historical Reviewers, 1 is a Technical Fresh Reviewer, and 1 is a Non-Technical Fresh Reviewer. At baseline for an existing draft, no reviewer receives prior-round history; use three technically oriented fresh reviewers and one Non-Technical Fresh Reviewer.

**Shared Behavior** (all 4 reviewers):
- Read the full LaTeX source (and compiled PDF if available)
- Read the persisted `review_profile` from `.paper-review/memory/session.yaml` and apply its emphasis when weighting comments
- Use the current round's literature/novelty evidence when provided by the Planner as round-local non-history input; Fresh Reviewers must not browse `.paper-review/audit/reviews/*` to retrieve it themselves
- Evaluate against all criteria in review-criteria.md
- Produce a review report using the template in review-criteria.md
- Be specific: reference exact equations, figures, tables, sections
- Never say "looks good" without evidence
- Check every empirical claim against actual numbers in tables
- Verify abstract matches actual contributions and results
- Flag any notation inconsistency, overclaiming, or missing reference
- Explicitly test whether the paper clearly distinguishes itself from the closest prior results and discusses that distinction honestly
- Include a complete core-dimension score table, median, and explicit pass/fail in every report so the result can be archived under `.paper-review/audit/`

### Historical Reviewer (2 per round)

**Additional context**: Receives the current draft, `.paper-review/memory/session.yaml`, and the **aggregated summary report from the previous round only** (not the full history of all rounds).

**Additional responsibilities**:
- For each issue listed in the previous round's aggregated summary, explicitly check whether it has been genuinely resolved or only superficially addressed
- Fill in the `Unresolved from last round` section of the report template
- If a fix introduced a new problem (e.g., a rewritten proof now has a gap), flag it as a new CRITICAL issue
- Do not re-flag issues that are cleanly resolved; acknowledge them in the `Resolved` list
- Do not read raw reports from earlier rounds by default; rely on the prior aggregated summary only

### Technical Fresh Reviewer (1 per round)

**Additional context**: Receives the current draft, the review rubric, non-review-history metadata from `.paper-review/memory/session.yaml`, and any current round literature/novelty evidence content passed directly by the Planner. Has no access to prior review reports, `.paper-review/memory/review/*`, or audit artifacts.

**Additional responsibilities**:
- Provide a fully independent assessment unanchored to previous feedback
- Focus on issues that may have been overlooked or introduced in recent revisions
- Leave the `Unresolved from last round` section blank (N/A)
- In PDF-only review mode, label any theory/evidence conclusion that cannot be checked from the available artifact as `limited evidence`
- In PDF-only review mode, treat any overall pass/fail as advisory only

### Non-Technical Fresh Reviewer (1 per round)

**Additional context**: Receives the same non-history inputs as the Technical Fresh Reviewer, but reviews as a busy, technically adjacent reviewer who may not understand every theorem, derivation, implementation detail, or domain-specific trick.

**Additional responsibilities**:
- Judge whether the paper is understandable and credible from the front door: title, abstract, introduction, contribution list, overview figure, section flow, figure/table captions, and visual first impression
- Put the strongest scoring signal in `Story / Logic`: can the reviewer understand the problem, stakes, prior-work gap, key idea, technical strength, evidence preview, and contribution boundaries without deep technical parsing?
- Still fill all 5 core-dimension scores, but label technical judgments as surface-level or `limited evidence` when they depend on details the reviewer did not deeply verify
- Preserve compatibility between subjective comments and structured scoring: first describe the reader impression, then assign the required scores as a compressed judgment of the clarity, trust, friction, and persuasiveness described in those comments
- Write comments as a subjective reader-impression report, not a pass/fail checklist: describe where the Abstract, Introduction, contribution list, and main-body reading path feel smooth, confusing, concrete, vague, trustworthy, or over-sold
- Note comprehension friction from concepts, acronyms, notation, method names, assumptions, metrics, or prior-work references that appear before the paper has made them feel understandable
- Distinguish concrete technical description from slogan-like writing by explaining which claims feel inflated or generic, and what kind of mechanism, condition, number, theorem, experiment, or implementation detail would make them more convincing
- Emphasize storyline breaks, generic contribution bullets, missing intuition, unclear novelty framing, overclaiming, confusing section order, weak figure/caption communication, and layout problems that reduce trust
- Leave the `Unresolved from last round` section blank (N/A)
- In PDF-only review mode, treat any overall pass/fail as advisory only

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
- Expand underdeveloped sections before polishing when the paper lacks motivation, mechanism, evidence, proof context, citation positioning, or analysis
- Compress text to fit page limits while preserving information
- Ensure consistent terminology and notation references
- Adapt citation style to target venue
- Rewrite sections based on review feedback
- Use `paper_maturity_profile` to match the expected depth, citation density, section balance, and story shape of close mature papers without copying their wording
- Translate Non-Technical Fresh Reviewer impressions into prose revisions that improve the reader's felt clarity and trust
- During initial maturity revision, preserve readability and story/logical flow so technically adjacent or non-specialist readers can understand the problem, contribution, technical strength, and evidence chain
- When a claim feels vague or slogan-like, revise toward concrete mechanism, condition, number, theorem, experiment, implementation detail, or limitation
- Add concept onboarding where the reader may feel lost, introducing terms, acronyms, notation, metrics, assumptions, and prior-work context at the point where they become useful

**Quality targets**:
- Every paragraph has a clear purpose
- No redundant sentences
- Claims feel concrete and grounded rather than inflated
- Transitions between sections feel natural
- Abstract gives a concrete sense of the problem, gap, idea, and result
- The story feels smoothly guided from problem -> gap -> key idea -> technical mechanism -> evidence -> contribution
- A skim reader can recover the paper's value from the Abstract, Introduction, contribution bullets, section openings, overview figure text, and captions
- Front matter passes a tired-but-fair reviewer read: concrete, guided, credible, and not over-sold
- The paper no longer feels like a short draft: related work, method/theory, evidence, analysis, and limitations have enough substance for the target venue

## Layout Agent

**Role**: Formatting, typesetting, and visual quality of the compiled PDF.

**Capabilities**:
- Fix equation overflow (overfull hbox)
- Convert between document formats (article, IEEEtran, NeurIPS, ICML, etc.)
- Optimize table formatting for column width
- Adjust figure sizing and placement
- Ensure no unresolved or serious LaTeX warnings that affect correctness, venue format, or readability
- Run a PDF Aesthetic Layout Gate after compile: section length balance, text/formula density, figure/table readability, caption usefulness, sparse/crowded pages, and final-page polish
- If a page looks bad because content is missing or a section is underdeveloped, route the issue back to Writing/Theory/Experiment rather than only shrinking spacing

**Common techniques**:
- `align` -> `multline` for long equations with line breaks
- Introduce abbreviations for repeated long expressions
- `\footnotesize` + `\tabcolsep` for tight tables
- `bmatrix` for column vectors
- `\!` negative thin spaces for math tightening

## Agent Communication Patterns

### Pattern 1: Review-Revise Loop
```
last_round_summary = rehydrated_last_round_summary or None
round = rehydrated_round or 0

Planner -> [Agent(s)] ->
  Planner (run or reuse References / Novelty Review Gate and save evidence bundle)
  Planner (run related-paper calibration when writing or substantially revising toward submit-ready)
  -> parallel: [Historical R1(draft+prev+evidence) | Historical R2(draft+prev+evidence) |
                Technical Fresh(draft+profile+evidence) | Non-Technical Fresh(draft+profile+evidence)]
  -> Planner (persist raw reports to audit, aggregate all 4 reports, merge issues, prioritize unresolved)
  -> Planner (write aggregated summary to audit and update memory/last_round_summary + issues ledger + maturity gaps)
  -> [Agent(s) expand/revise]
  -> Planner (append revision-log + decision-log entries)
  -> last_round_summary = aggregated summary report from this round
  -> repeat until all 4 reviewers have median >= 4 across the 5 core dimensions AND no core dimension below 3, no maturity gaps remain, or MAX_ROUNDS reached
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
Layout Agent -> Compile -> Technical Layout Gate -> PDF Aesthetic Layout Gate
            -> Fix source/content/layout -> Recompile -> Recheck
```

### Pattern 5: Final Polish
```
Writing Agent -> Layout Agent -> Final layout gate -> Planner (accept or iterate)
```
