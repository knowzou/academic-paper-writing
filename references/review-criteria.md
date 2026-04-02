# Harsh Academic Reviewer Criteria

## Table of Contents
- [Scoring](#scoring)
- [Content & Novelty](#content--novelty)
- [Theory & Rigor](#theory--rigor)
- [Experiments](#experiments)
- [Reproducibility](#reproducibility)
- [Writing Quality](#writing-quality)
- [Layout & Formatting](#layout--formatting)
- [Review Report Template](#review-report-template)

## Scoring

Rate each dimension 1-5:
- 5 = Top-venue ready
- 4 = Minor revision
- 3 = Major revision needed
- 2 = Fundamental issues
- 1 = Reject

**Pass condition**: for each reviewer individually, **both** of the following must hold:
1. Median score across all 6 dimensions >= 4
2. No single dimension scored below 3

All 4 reviewers must pass this bar.

## Content & Novelty

- Is the core idea genuinely new, or a trivial extension?
- Does the paper clearly state what is new vs. borrowed?
- Are contributions listed explicitly with supporting evidence for each?
- Are contributions calibrated honestly? Overclaiming is worse than underclaiming.
- Is the related work comprehensive, fair, and clearly contrasted with this paper?
- Red flag: "applied method X to domain Y" with no new insight.

## Theory & Rigor

- Are all theorems/propositions stated with precise, complete assumptions?
- Are proofs logically sound and self-contained?
- Are there hidden assumptions not stated explicitly?
- Is the derivation chain clear (model -> surrogate -> algorithm)?
- Are approximations justified with error bounds or qualitative arguments?
- Is notation defined before first use and consistent throughout?
- Are there notation clashes between sections?

## Experiments

### Design
- Reproducible? (seeds, parameters, code/data availability statement)
- Baselines fair and representative of SOTA?
- Hyperparameters tuned with equal budget across methods?
- Enough test samples for statistical reliability?

### Results
- Appropriate metrics reported (mean, percentile, max, std)?
- Every claim supported by actual numbers in tables?
- Cases where the method is NOT the best reported honestly?
- Error bars or confidence intervals where appropriate?

### Analysis
- Ablation study showing contribution of each component?
- Sensitivity analysis for key hyperparameters?
- Comparison with theoretical bounds if available?
- Runtime comparison on fair hardware?

## Reproducibility

- Is there a code/data availability statement?
- Are random seeds and key hyperparameters reported?
- Is hardware and compute environment described?
- Can the main results be reproduced from the information provided?
- Are experiment scripts or notebooks included or referenced?

## Writing Quality

### Structure & Balance
For a typical 10-page paper (two-column):
| Section | Target length |
|---------|--------------|
| Abstract | 150-250 words |
| Introduction | 1-1.5 pages |
| Related Work | 0.75-1 page |
| Model/Problem | 1-1.5 pages |
| Method | 2-3 pages |
| Experiments | 2-3 pages |
| Conclusion | 0.5 page |

Adjust proportionally for other page limits. Flag sections that deviate >50% from target.

### Prose
- Is writing concise? Flag paragraphs that could lose 30% without losing info.
- Any redundant sentences saying the same thing twice?
- Abstract self-contained and informative (problem, method, key result)?
- Intro clearly states problem, gap, approach, contributions?
- No vague claims ("significantly better") without quantification?
- All acronyms defined at first use?

### Text Density (per page, two-column)
- Equations: 2-4 typical; >6 too dense, <1 too sparse
- Figures/tables: >= 1 visual per 2 pages in experiments
- Prose: no text wall >15 lines without a break

## Layout & Formatting

### PDF Quality
- Zero overfull hbox warnings?
- All equations fit within column/page width?
- No orphan lines or excessive whitespace?
- All references resolve (no "??" in PDF)?

### Figures
- Axes labeled with units?
- Legends readable and consistent?
- Captions self-contained?
- Readable at print size?

### Tables
- Headers clear with units?
- Best result highlighted (bold)?
- Numbers aligned, consistent decimal places?
- Caption above table (IEEE) or per venue convention?

### References
- Bibliography complete (years, venues, pages)?
- Citation style matches target venue?
- No duplicate or broken entries?

## Review Report Template

```
## Reviewer Info
- Type: [ ] Historical Reviewer  [ ] Fresh Reviewer
- Round: [N]

## Summary
[1-2 sentences: what the paper does]

## Scores
| Dimension | Score | Notes |
|-----------|-------|-------|
| Novelty | /5 | |
| Theory | /5 | |
| Experiments | /5 | |
| Reproducibility | /5 | |
| Writing | /5 | |
| Layout | /5 | |
| **Median** | **/5** | computed across the 6 dimensions above |

**This reviewer passes** if: median across all 6 dimensions >= 4 AND no single dimension below 3.

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
- **Abstract**: ...
- **Introduction**: ...
- **Related Work**: ...
- **Method/Theory**: ...
- **Experiments**: ...
- **Conclusion**: ...

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
1. **Unresolved issues** from Historical Reviewers → escalate one severity level (MINOR→MAJOR, MAJOR→CRITICAL, CRITICAL stays CRITICAL)
2. **Issues flagged by 3-4 reviewers** → CRITICAL priority regardless of original severity
3. **Issues flagged by 2 reviewers** → retain original severity
4. **Issues flagged by 1 reviewer** → downgrade one level (CRITICAL→MAJOR, MAJOR→MINOR, MINOR→low-priority); **low-priority issues are tracked but excluded from the required revision task list**

**Aggregation order**: apply rule 1 (escalation) first, then rules 2-4 (multi-reviewer weighting).

**Pass condition**: all 4 reviewers individually satisfy: median >= 4 AND no dimension below 3.

**Aggregated summary report format** (passed to Historical Reviewers next round):
```
## Aggregated Round N Summary

### Scores (per reviewer)
| Reviewer | Novelty | Theory | Experiments | Reproducibility | Writing | Layout | Median | Pass? |
|----------|---------|--------|-------------|-----------------|---------|--------|--------|-------|
| Historical R1 | | | | | | | | |
| Historical R2 | | | | | | | | |
| Fresh R1 | | | | | | | | |
| Fresh R2 | | | | | | | | |

### Top Priority Issues (after aggregation)
[CRITICAL] ...
[MAJOR] ...

### Resolved This Round
...
```
