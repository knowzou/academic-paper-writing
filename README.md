# Academic Paper Writing

This repository contains an `academic-paper-writing` skill for Codex/Claude-style agent workflows focused on STEM paper writing and revision.

The skill is designed for cases such as:

- Writing a paper from scratch from research results or notes
- Revising an existing paper draft
- Strengthening theory, experiments, or writing quality
- Converting a paper to a target venue format
- Running a harsh multi-reviewer review loop before submission

## Repository Structure

```text
.
├── SKILL.md
└── references/
    ├── agent-roles.md
    └── review-criteria.md
```

## What Is Included

### `SKILL.md`

The main skill definition. It describes:

- Entry-point contextual Q&A
- Mode dispatch for writing from scratch, revising a draft, or handling a focused task
- Planner / Theory / Experiment / Writing / Layout agent responsibilities
- A multi-round review-revise loop with historical and fresh reviewers

### `references/agent-roles.md`

Defines the agent architecture and orchestration patterns, including:

- Planner-led coordination
- Parallel theory and experiment work
- Review aggregation
- Layout and final polish cycles

### `references/review-criteria.md`

Defines the review rubric used by review agents, including:

- Scoring dimensions
- Pass conditions
- Section-level writing and layout checks
- Review report template
- Aggregation rules for multi-reviewer feedback

## Typical Workflow

1. Collect context about the paper, venue, focus, and quality target.
2. Choose a mode:
   - write from scratch
   - revise an existing draft
   - handle a specific isolated task
3. Dispatch specialist agents for theory, experiments, writing, and layout as needed.
4. Run parallel review agents to assess the draft.
5. Aggregate issues, revise, and repeat until the quality bar is met.

## Use Cases

- NeurIPS / ICML / ICLR style ML papers
- CVPR / ICCV / ECCV style computer vision papers
- IEEE or Springer journal submissions
- Workshop papers or arXiv drafts

## Notes

- This repository currently contains the skill specification and reference documents only.
- It does not include paper source templates, experiment code, or LaTeX build tooling.
- The workflow is intended to be adapted inside an agentic writing environment that can read `SKILL.md` and the reference files.

