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
- For theory-heavy papers, is there enough evidence to validate the assumptions, boundary cases, or practical relevance of the theory?
- For experiment-heavy papers, are the benchmarks, baselines, ablations, and sensitivity analyses sufficient?
- Are cases where the method is not best reported honestly?

### Design quality

- Are baselines fair and representative?
- Are hyperparameters tuned with comparable effort across methods?
- Are metrics appropriate for the paper’s actual claims?
- Is the evidence scale appropriate to the venue and focus profile?

### Reproducibility

- Is there a code/data availability statement?
- Are random seeds, hyperparameters, and hardware/environment details reported where needed?
- Can a reasonable reader reproduce the main evidence from the information provided?
- Are experiment scripts, theorem-checking scripts, or notebooks included or referenced when relevant?

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

### Prose

- Is the abstract self-contained and informative?
- Does the introduction clearly state the problem, gap, approach, and contributions?
- Is the prose concise, or could major paragraphs lose 30% without losing information?
- Are there repeated claims across intro, method, and conclusion?
- Are all acronyms defined at first use?
- Are limitations and discussion written with enough clarity to be credible?

## References & Scholarly Positioning

- Is the related work comprehensive, fair, and clearly contrasted with this paper?
- Are the closest prior papers or theoretical results explicitly identified?
- Does the paper state both the **formal** difference and the **substantive** difference from the closest prior work?
- Does it explain why that difference matters?
- Does it discuss remaining limitations or narrower scope relative to prior work?
- Are citations used to support specific claims, rather than as decorative lists?
- Red flag: the paper claims novelty but never says what the nearest competing result actually proves or shows.

## Final Layout Gate

This is a required final gate for `submit-ready` workflows when the paper source can be compiled.

- Zero unresolved references (`??`) in the PDF
- No serious overfull hbox or equation overflow problems
- Figures and tables readable at print size
- Captions self-contained enough for the venue norm
- Bibliography complete and style-consistent
- No obvious venue-format violations

If only a PDF is available and source cannot be compiled, mark the layout gate as `Limited / N/A` and do not fold it into the core-score median.

## Review Report Template

```
## Reviewer Info
- Type: [ ] Historical Reviewer  [ ] Fresh Reviewer
- Round: [N]
- Review Profile: [focus profile + venue profile]
- Evidence Basis: [Full source | PDF-only / limited evidence]
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
4. **Issues flagged by 1 reviewer** -> downgrade one level (CRITICAL->MAJOR, MAJOR->MINOR, MINOR->low-priority); **low-priority issues are tracked but excluded from the required revision task list**

**Aggregation order**: apply rule 1 (escalation) first, then rules 2-4 (multi-reviewer weighting).

**Pass condition**: all 4 reviewers individually satisfy: median >= 4 across the 5 core dimensions AND no core dimension below 3.

**Aggregated summary report format** (passed to Historical Reviewers next round):
```
## Aggregated Round N Summary

### Scores (per reviewer)
| Reviewer | Story / Logic | Theory / Rigor | Evidence | Writing / Structure | References / Positioning | Median | Pass? |
|----------|---------------|----------------|----------|---------------------|--------------------------|--------|-------|
| Historical R1 | | | | | | | |
| Historical R2 | | | | | | | |
| Fresh R1 | | | | | | | |
| Fresh R2 | | | | | | | |

### Round Verdict
- Overall pass/fail: ...
- Final layout gate status (if evaluated): ...
- Evidence basis: ...

### Score Deltas vs Previous Round
- Historical R1: ...
- Historical R2: ...
- Fresh R1: ...
- Fresh R2: ...

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
