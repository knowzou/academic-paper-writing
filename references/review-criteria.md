# Harsh Academic Reviewer Criteria

## Table of Contents
- [Scoring](#scoring)
- [Review Profiles](#review-profiles)
- [Audit Persistence Requirements](#audit-persistence-requirements)
- [Story & Logic](#story--logic)
- [Theory & Rigor](#theory--rigor)
- [Evidence](#evidence)
- [Writing & Structure](#writing--structure)
- [References & Scholarly Positioning](#references--scholarly-positioning)
- [Paper Maturity Signals](#paper-maturity-signals)
- [Final Layout Gate](#final-layout-gate)
- [Review Report Template](#review-report-template)
- [Aggregation Rules (for Planner)](#aggregation-rules-for-planner)

## Scoring

Rate each **core dimension** 1-5:
- 5 = Top-venue ready
- 4 = Minor revision
- 3 = Major revision needed
- 2 = Fundamental issues
- 1 = Reject

Core dimensions:
- Story / Logic
- Theory / Rigor
- Evidence
- Writing / Structure
- References / Scholarly Positioning

**Pass condition for the iterative content-review loop**: for each reviewer individually, **both** of the following must hold:
1. Median score across all 5 core dimensions >= 4
2. No single core dimension scored below 3

All 4 reviewers must pass this bar.

`Layout` is **not** a core scored dimension during the main review loop. For `submit-ready` quality targets, the paper must also pass the **Final Layout Gate** before acceptance.

For `submit-ready` quality targets, passing scores are not enough if the paper still feels like an expanded draft. The Planner should also judge paper maturity using the anchor-paper calibration and the signals below.

## Review Profiles

All reviewers use the same 5 core dimensions, but emphasis changes with the paper focus and venue profile supplied by the skill.

### Focus profiles

- `Theory-heavy`: prioritize `Theory / Rigor` and `Story / Logic`. `Evidence` should validate the core claims and assumptions, but need not be benchmark-heavy.
- `Experiment-heavy`: prioritize `Evidence` and `Story / Logic`. `Theory / Rigor` still checks whether the method definition and scope are sound.
- `Balanced theory + experiments`: require strong alignment between theorem statements, empirical claims, and conclusions.
- `Application / case study`: prioritize realistic problem framing, application-specific evidence, and fair positioning against domain-specific prior work.

### Venue modifiers

- `ML conference`: strong emphasis on contribution clarity, rigorous claim support, and honest comparison to the strongest prior work.
- `Computer vision`: stronger expectations on empirical evidence, visual clarity, and comparison quality.
- `IEEE / Springer journal`: stronger expectations on completeness, literature coverage, and mature discussion of scope and limitations.
- `Workshop / arXiv`: exploratory evidence is acceptable, but the paper must calibrate claims clearly and discuss limitations explicitly.

### PDF-only review mode

When only a PDF (or extracted text) is available and source files are unavailable:
- `Final Layout Gate` is `N/A`
- `Story / Logic`, `Writing / Structure`, and `References / Scholarly Positioning` may still be scored normally from the available artifact
- `Theory / Rigor` and `Evidence` may still be scored, but reviewers must explicitly mark unsupported judgments as `limited evidence`
- Reviewers must not claim source-level verification they could not actually perform
- Any overall pass/fail in this mode is advisory only and must not be treated as equivalent to a full-source review pass

## Audit Persistence Requirements

When this review protocol is used inside the skill workflow, the Planner must persist review artifacts under `.paper-review/audit/`.

### Raw reviewer report requirements

Every raw reviewer report archived in `audit/reviews/round-XX/` must include:
- reviewer type
- round number
- the active review profile
- literature/novelty evidence basis for `References / Positioning` (`research-lit + novelty-check`, `reused literature-novelty evidence`, `no fresh literature search`, or `PDF-only / limited evidence`)
- all 5 core-dimension scores
- median score
- explicit pass/fail
- final layout gate status if evaluated (`Pass`, `Fail`, or `N/A`)
- whether the review used full source access or `PDF-only / limited evidence`
- strengths
- weaknesses by severity
- required revisions

Round shape:
- Standard full-review rounds contain 4 raw reviewer reports
- Single-review preflight rounds may contain fewer reports if the workflow explicitly labels them as preflight / Mode C rounds

### Aggregated summary requirements

Every aggregated round summary archived in `audit/reviews/round-XX/aggregated-summary.md` must include:
- per-reviewer score table across the 5 core dimensions
- per-reviewer median and pass/fail
- round-level pass/fail decision
- final layout gate status if evaluated
- evidence basis (`full source` or `PDF-only / limited evidence`)
- literature/novelty evidence basis, including path to the saved search/novelty bundle when available
- paper maturity status (`acceptable`, `needs expansion`, or `limited evidence`) with the main anchor-paper gaps
- score deltas versus the previous round when available
- stable issue IDs
- resolved issues
- unresolved issues
- regressed issues
- top priority revision targets

## Story & Logic

- Is the paper’s core problem clear and well motivated?
- Does the paper explain the gap in prior work precisely rather than rhetorically?
- Is there a clean logical chain from problem -> approach -> theory/evidence -> conclusions?
- Are the listed contributions actually supported later in the paper?
- Are assumptions, limitations, and scope boundaries surfaced early enough?
- Red flag: interesting theorem or experiment appears, but it does not support the claimed story of the paper.

### Non-Technical Fresh Reviewer emphasis

One Fresh Reviewer in each full review round reviews as a busy, technically adjacent reader who may not understand every proof, derivation, or implementation detail. This reviewer still completes the full score table. The compatibility rule is: comments should capture the human reading impression, and scores should summarize that impression in the required rubric format.

For this reviewer, the `Story / Logic` score should be grounded in impressions such as:
- Can the paper be understood from the title, abstract, introduction, contribution list, overview figure, and section flow?
- Is the problem -> gap -> key idea -> technical strength -> evidence chain clear without deep technical parsing?
- Are the contributions concrete and checkable rather than generic claims of novelty, generality, or extensive experiments?
- Does the paper make the method feel technically strong through intuition, comparison, evidence preview, and clear limitations?
- Do figure captions, section openings, and layout choices help a tired reviewer follow the story?
- Would a reviewer who does not fully understand the technical details still know what the paper contributes and why it matters?

Scoring guidance for this reviewer:
- `Story / Logic` is the main score channel for reader trust, storyline smoothness, contribution clarity, and whether the paper feels concrete rather than slogan-like.
- `Writing / Structure` should reflect prose flow, section order, caption helpfulness, and skim readability.
- `Theory / Rigor`, `Evidence`, and `References / Positioning` should still be scored when possible, but the reviewer should label judgments as surface-level or limited when they are based on legibility and plausibility rather than deep technical verification.
- The numeric scores should be consistent with the comments: a paper described as confusing, vague, or over-sold should not receive a high `Story / Logic` score even if the technical content may be strong.

The Non-Technical Fresh Reviewer comments should be qualitative and reader-centered rather than binary. They should describe the reviewer's subjective sense of:
- Abstract: whether the problem, gap, method idea, and key result feel concrete, memorable, and credible, or instead feel generic and slogan-like.
- Introduction: whether the story feels naturally guided from problem -> gap -> idea -> contribution, and where the reader starts to feel lost, rushed, or unconvinced.
- Contribution list: whether the bullets feel like real technical contributions, or like broad claims of novelty, generality, or effectiveness.
- Main-body reading path: where concepts, acronyms, notation, assumptions, metrics, or prior-work names create friction because the reader does not yet feel oriented.
- Vagueness impression: phrases that sound impressive but leave the reviewer wondering what the actual mechanism, condition, number, theorem, experiment, implementation detail, or limitation is.
- Slogan-vs-substance impression: places where the paper feels like it is selling a claim instead of showing the actual process, technical design choice, or evidence chain.

## Theory & Rigor

- Are all theorems, propositions, and lemmas stated with precise, complete assumptions?
- Are proofs logically sound and self-contained enough for the target venue?
- Are there hidden assumptions not stated explicitly?
- Is the derivation chain clear (problem -> model -> surrogate -> algorithm -> guarantee)?
- Is notation defined before first use and consistent throughout?
- Are approximations justified with error bounds or clear qualitative arguments?
- If the main result is close to prior work, does the paper state exactly how the assumptions, guarantees, scope, or proof technique differ?
- Does the paper discuss whether the difference from prior theory is substantive or merely formal?

## Evidence

This dimension combines experimental support and reproducibility expectations.

### Claim support

- Does the evidence actually test the claims made in the paper?
- Do the main figures and tables support the method, theory, and storyline claims they are used to support?
- Could a skeptical reviewer use any result, missing comparison, weak ablation, or trade-off in the figures/tables to attack the paper's core story?
- For theory-heavy papers, is there enough evidence to validate the assumptions, boundary cases, or practical relevance of the theory?
- For experiment-heavy papers, are the benchmarks, baselines, ablations, and sensitivity analyses sufficient?
- Are cases where the method is not best reported honestly?

### Design quality

- Are baselines fair and representative?
- Are hyperparameters tuned with comparable effort across methods?
- Are metrics appropriate for the paper’s actual claims?
- Are figure/table captions and surrounding text honest about what the result proves, partially supports, or does not support?
- Is the evidence scale appropriate to the venue and focus profile?

### Reproducibility

- Is there a code/data availability statement?
- Are random seeds, hyperparameters, and hardware/environment details reported where needed?
- Can a reasonable reader reproduce the main evidence from the information provided?
- Are experiment scripts, theorem-checking scripts, or notebooks included or referenced when relevant?
- Are reproducibility details concise and paper-like, rather than a dump of run logs, shell traces, or file-system interaction records?

## Writing & Structure

### Structure & balance

For a typical 10-page paper (two-column):
| Section | Target length |
|---------|--------------|
| Abstract | 150-250 words |
| Introduction | 1-1.5 pages |
| Related Work | 0.75-1 page |
| Model/Problem | 1-1.5 pages |
| Method | 2-3 pages |
| Evidence / Experiments | 2-3 pages |
| Conclusion | 0.5 page |

Adjust proportionally for other page limits. Flag sections that deviate >50% from target unless the venue/profile clearly justifies it.

Treat these targets as reviewer-taste guidance, not rigid word-count rules. A section can be short and still mature if it is dense and well supported; it can also be long and still feel draft-like if it repeats claims without mechanism, evidence, citations, or analysis.

### Prose

- Is the abstract self-contained and informative?
- Does the abstract progress from background/motivation -> what the paper does -> how it does it -> evidence and trade-offs?
- Does the introduction clearly state the problem, gap, approach, and contributions?
- Does the introduction progress from existing setting -> difference from prior work -> why decisions must be made before measurement -> solution and contributions?
- Is the prose concise, or could major paragraphs lose 30% without losing information?
- Are there repeated claims across intro, method, and conclusion?
- Are all acronyms defined at first use?
- Are limitations and discussion written with enough clarity to be credible?
- Does the paper read like a paper rather than a developer document, avoiding command-by-command workflow narration or file/path/log talk in the main text?
- Does the paper use `we` mainly for author actions, while letting background facts, mechanisms, assumptions, and results use their natural subjects?
- Are concrete actions, mechanisms, numbers, theorems, experiments, and limitations used instead of generic `we show` claims or defensive reviewer-response language?
- Are equations, experimental numbers, comparison boundaries, and limitations preserved while the narration is made clearer?
- Are references clear and sentence openings varied, avoiding long runs of nominalizations or repeated sentence patterns?

## References & Scholarly Positioning

This score should be based on the draft plus a fresh literature/novelty evidence bundle produced with `research-lit` and `novelty-check`, unless the user explicitly requested a no-search/internal-only review. If no fresh search was run, mark the basis clearly and normally cap the score at 3.

- Is the related work comprehensive, fair, and clearly contrasted with this paper?
- Are the closest prior papers or theoretical results explicitly identified?
- Does the paper state both the **formal** difference and the **substantive** difference from the closest prior work?
- Does it explain why that difference matters?
- Does it discuss remaining limitations or narrower scope relative to prior work?
- Are citations used to support specific claims, rather than as decorative lists?
- Red flag: the paper claims novelty but never says what the nearest competing result actually proves or shows.

### Literature / novelty evidence requirements

The evidence bundle should include:
- search queries or discovery route used by `research-lit`
- closest-prior table: citation, core claim/result, formal overlap, substantive overlap, remaining distinction
- `novelty-check` verdict: `clear`, `partial`, `weak`, or `unclear`
- missing citations and concrete positioning edits

### Score anchors for References / Positioning

- **5**: Fresh search found the closest prior work; the paper explicitly contrasts against it with formal and substantive differences, explains why the differences matter, and states remaining limitations.
- **4**: Fresh search found no fatal overlap; closest-prior comparison is mostly clear, with minor missing citations or wording fixes.
- **3**: Related work is plausible but incomplete, or no fresh search was run; closest-prior boundary needs major sharpening.
- **2**: Literature search finds substantial overlap that the paper does not address, or the draft risks reading as a repackaging of known work.
- **1**: Novelty-check indicates the core contribution is already present in prior work and the paper does not provide a credible distinction.

## Paper Maturity Signals

Use these signals when the workflow target is submit-ready. This is a subjective maturity judgment informed by 2-3 closest mature papers, not a hard checklist.

Draft-like signals:
- Introduction is short, generic, or jumps from motivation to claims without a clear gap and method idea.
- Related work has too few close citations, reads like a list, or avoids the strongest neighboring papers.
- Method/theory sections describe the idea at slogan level but do not explain mechanism, assumptions, algorithmic choices, proof shape, or failure cases.
- Experiments lack enough setup, baseline explanation, ablation, sensitivity, dataset/task detail, or interpretation of what the numbers mean.
- Figures/tables look placeholder-like, are weakly captioned, or do not carry the story.
- Limitations and discussion feel shallow or defensive.
- Section lengths and page density feel visibly unlike the calibrated anchor papers: too sparse, too crowded, too formula-heavy without intuition, or too prose-heavy without technical substance.

When these signals appear, the revision plan should create expansion, citation, evidence, theory, figure/table, or analysis tasks before final polish. Do not solve a maturity problem by only tightening prose.

## Final Layout Gate

This is a required final gate for `submit-ready` workflows when the paper source can be compiled.

### Technical Layout Gate

- Zero unresolved references (`??`) in the PDF
- No serious overfull hbox or equation overflow problems
- Figures and tables readable at print size
- Captions self-contained enough for the venue norm
- Bibliography complete and style-consistent
- No obvious venue-format violations

### PDF Aesthetic Layout Gate

Judge the compiled PDF as a reviewer would, using the calibrated anchor papers as taste references:
- Section lengths feel balanced for the venue and paper type.
- Text, formulas, figures, and tables have a mature density; pages are not visibly sparse or overcrowded.
- Equations are readable and surrounded by enough intuition.
- Figures/tables are placed where they help the story and are legible without zooming.
- Captions carry useful information rather than merely naming the object.
- The last page and bibliography area do not look careless, empty, or squeezed.
- The visual first impression supports trust in the work.

If the aesthetic gate fails because the paper lacks content, analysis, or figures, route the issue to Writing/Theory/Experiment as a maturity problem rather than only changing spacing.

If only a PDF is available and source cannot be compiled, mark the layout gate as `Limited / N/A` and do not fold it into the core-score median.

## Review Report Template

```
## Reviewer Info
- Type: [ ] Historical Reviewer  [ ] Technical Fresh Reviewer  [ ] Non-Technical Fresh Reviewer
- Round: [N]
- Review Profile: [focus profile + venue profile]
- Evidence Basis: [Full source | PDF-only / limited evidence]
- Literature / Novelty Basis: [research-lit + novelty-check | reused literature-novelty evidence | no fresh literature search | PDF-only / limited evidence]
- Decision Strength: [Full-review decision | Advisory-only decision]

## Summary
[1-2 sentences: what the paper does]

## Scores
| Dimension | Score | Notes |
|-----------|-------|-------|
| Story / Logic | /5 | |
| Theory / Rigor | /5 | |
| Evidence | /5 | |
| Writing / Structure | /5 | |
| References / Positioning | /5 | |
| **Median** | **/5** | computed across the 5 core dimensions above |

**This reviewer passes** if: median across all 5 core dimensions >= 4 AND no single core dimension below 3.
**Pass/Fail**: [Pass | Fail]
**Final Layout Gate**: [Pass | Fail | N/A]

## Literature / Novelty Basis
- Evidence bundle path: [...]
- Closest prior work checked: [...]
- Novelty-check verdict: [clear | partial | weak | unclear]
- How this affects References / Positioning score: [...]

## Unresolved from Last Round (Historical Reviewers only; Fresh Reviewers: N/A)
Based on the aggregated summary from the previous round, list issues that are NOT yet resolved:
1. [CRITICAL/MAJOR] Description of unresolved issue — why the current fix is insufficient
2. ...

## Resolved from Last Round (Historical Reviewers only; Fresh Reviewers: N/A)
Issues from prior rounds that are cleanly resolved (acknowledge briefly):
1. [Round N-X] Description — confirmed resolved

## Strengths
1. ...
2. ...

## Weaknesses (by severity)
1. [CRITICAL] ...
2. [MAJOR] ...
3. [MINOR] ...

## Section-by-Section Comments
- **Abstract / Introduction**: ...
- **Related Work / Positioning**: ...
- **Method / Theory**: ...
- **Evidence / Experiments**: ...
- **Conclusion / Limitations**: ...

## Specific Issues
- Eq.(X) / Fig.Y / Table Z / Line N: [issue]

## Required Revisions
1. ...
2. ...

## Verdict
[ ] Ready to submit
[ ] Minor revision then submit
[ ] Major revision needed
[ ] Fundamental restructuring needed
```

## Aggregation Rules (for Planner)

After collecting all 4 reports each round, produce a single **aggregated summary report** to pass to Historical Reviewers next round:

**Issue priority rules:**
1. **Unresolved issues** from Historical Reviewers -> escalate one severity level (MINOR->MAJOR, MAJOR->CRITICAL, CRITICAL stays CRITICAL)
2. **Issues flagged by 3-4 reviewers** -> CRITICAL priority regardless of original severity
3. **Issues flagged by 2 reviewers** -> retain original severity
4. **Reader-impression exception**: issues raised only by the Non-Technical Fresh Reviewer are not automatically downgraded when they affect `Story / Logic` or `Writing / Structure` through reader friction, storyline breaks, vague contributions, weak skim readability, confusing captions, or layout choices that reduce trust. Keep these in the required revision task list as Writing Agent or Layout Agent work unless the Planner explicitly defers them with a rationale.
5. **Other issues flagged by 1 reviewer** -> downgrade one level (CRITICAL->MAJOR, MAJOR->MINOR, MINOR->low-priority); **low-priority issues are tracked but excluded from the required revision task list**

**Aggregation order**: apply rule 1 (escalation) first, then rules 2-3 (multi-reviewer weighting), then rule 4 (reader-impression exception), then rule 5 (single-reviewer downgrade).

**Pass condition**: all 4 reviewers individually satisfy: median >= 4 across the 5 core dimensions AND no core dimension below 3.

**Aggregated summary report format** (passed to Historical Reviewers next round):
```
## Aggregated Round N Summary

### Scores (per reviewer)
| Reviewer | Story / Logic | Theory / Rigor | Evidence | Writing / Structure | References / Positioning | Median | Pass? |
|----------|---------------|----------------|----------|---------------------|--------------------------|--------|-------|
| Historical R1 | | | | | | | |
| Historical R2 | | | | | | | |
| Technical Fresh | | | | | | | |
| Non-Technical Fresh | | | | | | | |

### Round Verdict
- Overall pass/fail: ...
- Final layout gate status (if evaluated): ...
- Paper maturity status: ...
- Evidence basis: ...

### Score Deltas vs Previous Round
- Historical R1: ...
- Historical R2: ...
- Technical Fresh: ...
- Non-Technical Fresh: ...

### Top Priority Issues (after aggregation)
[CRITICAL] ISSUE-### ...
[MAJOR] ISSUE-### ...

### Resolved This Round
...

### Still Open
- ISSUE-### ...

### Regressed
- ISSUE-### ...
```
