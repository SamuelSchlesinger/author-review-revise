# author-review-revise

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code) that produces hierarchical research corpora through parallel authoring, multi-perspective review, and iterative revision.

## Installation

Clone this repo into your Claude Code skills directory:

```bash
# Project-level (this project only)
cd your-project
mkdir -p .claude/skills
git clone https://github.com/SamuelSchlesinger/author-review-revise .claude/skills/author-review-revise

# Personal (all projects)
mkdir -p ~/.claude/skills
git clone https://github.com/SamuelSchlesinger/author-review-revise ~/.claude/skills/author-review-revise
```

## Usage

In Claude Code, run:

```
/author-review-revise <topic>
```

or with a custom output directory:

```
/author-review-revise <topic> ./my-output-dir
```

## What it does

The skill runs a three-phase research workflow:

1. **Author** — Decomposes the topic into subtopics and dispatches parallel research agents that write a structured, interlinked corpus with citations and validation scripts.
2. **Review** — Parallel reviewers examine the entire corpus through five lenses: accuracy, completeness, coherence, source quality, and staleness.
3. **Revise** — Triages review findings by severity and dispatches agents to fix issues, then loops back to review until convergence.

The output is a self-contained directory of interlinked Markdown documents with a master bibliography, validation artifacts, and a git history showing the evolution through review cycles.
