---
name: author-review-revise
description: Produce a hierarchical research corpus through parallel authoring, multi-perspective review, and iterative revision. Use when the user wants deep, cited, structured research on a topic.
argument-hint: "<topic> [output-dir]"
disable-model-invocation: true
---

# Author, Review, Revise

A skill for producing a hierarchical corpus of research documents through
parallel authoring, multi-perspective review, and iterative revision.

## Input

The user provides:
- **Topic or question** — the research subject
- **Output directory** — where to write the corpus (default: `./research/`)
- **Scope guidance** (optional) — depth, breadth, or specific subtopics to prioritize

## Corpus Structure

All output lives under the output directory. The structure is:

```
research/
  index.md              # Root document: overview, taxonomy, and links to all subtopics
  subtopic-a/
    index.md            # Main document for subtopic A
    detail-1.md         # Deeper dives linked from subtopic A
    data/               # Raw data, scripts, logs used to validate claims
      validate-claim.py
      output.txt
  subtopic-b/
    index.md
    ...
  sources.md            # Master bibliography with all primary sources
```

Rules:
- Every `.md` file must be reachable from `research/index.md` via relative links.
- Each document should be focused enough to stand alone but linked into the hierarchy.
- Prefer deeper hierarchy over long monolithic documents. A section that exceeds ~300 lines or covers a clearly separable subtopic should become its own linked document.
- `sources.md` at the root is the master bibliography — the single source of truth for all citations.

## Writing Standards

These rules apply to **all agents that write or edit corpus content** — whether authoring new documents or revising existing ones.

- **Cross-links**: every `index.md` must link to all its child detail documents. Do not create orphan documents.
- **Primary sources**: cite papers, official docs, RFCs, and authoritative references — not blog summaries or secondary accounts.
- **Citation key format**: follow the Inline Citation Format section below. Every agent must use the same convention.
- **Currency check**: verify that claims about projects, tools, and systems reflect the current state. If a project has been renamed, discontinued, or substantially changed, note the current status.
- **No fabrication**: do NOT invent specific version numbers, CVE identifiers, benchmark figures, or URLs that cannot be verified. If a specific detail is unknown, say so explicitly (e.g., "version X.x as of [date]; verify against [official source]") rather than guessing a plausible-looking value. Fabricated specifics are worse than acknowledged uncertainty.
- **Unknown URLs**: if a URL cannot be found or verified, write "(URL unknown — verify against [description of where to find it])" rather than guessing. A wrong URL is worse than no URL.
- **No direct writes to `sources.md`**: agents adding new citations must include them in a `## Local References` section at the bottom of the document they are editing, with full bibliographic details. These are compiled into `sources.md` during assembly steps.
- **Validation artifacts**: when a claim is non-obvious or quantitative, write a validation script or worked example into the relevant `data/` subfolder.

## Phase 1: Author

1. **Decompose the topic.** Before dispatching agents, draft a research plan: identify distinct subtopics, open questions, and known unknowns. Write this plan to `research/index.md` as an initial skeleton with TODOs.

2. **Dispatch parallel research agents.** For each subtopic, launch a subagent with a clear mandate:
   - What question(s) to answer
   - What format to write (the subtopic's `index.md` and any sub-documents)
   - The Writing Standards (above) — include or reference them in the agent prompt

   Each agent writes its documents directly into the corpus. Agents should use web search, code execution, and any other available tools to ground their findings.

3. **Assemble.** Once all authoring agents complete:
   - Update `research/index.md` to reflect the actual structure produced, including links to all documents and a "Supplementary Code" section listing data scripts.
   - **Compile `sources.md`** by collecting all Local References sections into a single deduplicated bibliography. Assign one canonical anchor per source — no aliases. Then **enforce key consistency end-to-end**: replace inline citation display text in all documents to match the canonical keys (e.g., if an agent wrote `[alglave-herd][alglave-herd]` but the canonical key is `[alglave14]`, rewrite the inline citation to `[alglave14][alglave14]`). Finally, replace Local References sections with bare link definitions using canonical keys (e.g., `[alglave14]: ../sources.md#alglave14`).

## Phase 2: Review

Launch parallel review agents, each examining the **entire corpus** through a specific lens. Each reviewer reads all documents and returns a structured list of findings (not saved to the corpus).

Each finding must include: **Location** (file + section), **Severity** (Error / Gap / Polish), **Finding** (what's wrong), **Suggested fix** (how to correct it).

### Review Lenses

| Lens | Focus |
|---|---|
| **Accuracy** | Are factual claims correct? Cross-check against primary sources. Run or re-run validation scripts. Flag anything unverified. |
| **Completeness** | Are there gaps — subtopics mentioned but not covered, questions raised but not answered, obvious related areas omitted? |
| **Coherence** | Does the hierarchy make sense? Are there contradictions between documents? Is terminology consistent? Are all cross-links present and working? Are citation keys consistent? |
| **Source quality** | Are citations present for all significant claims? Are sources primary, accessible, and correctly attributed? Do citation anchors match `sources.md` entries? |
| **Staleness** | Are any claims likely outdated given the current date? Flag version-specific information that may have changed. Flag projects that have been renamed, discontinued, or substantially changed. |

Reviewers must **validate concretely** — run scripts, fetch URLs to check they resolve, compare claims across documents, verify numbers. "Looks right" is not a review finding.

**Fabrication check**: Reviewers should be especially skeptical of specific version numbers, CVE identifiers, benchmark figures, and URLs that look plausible but may be invented. If a claim includes a specific identifier that cannot be cross-checked against a second source, flag it.

## Phase 3: Revise

1. **Triage.** Collect all review findings. Deduplicate across reviewers (multiple lenses often flag the same issue). Prioritize by severity:
   - **Error** — factually wrong, contradictory, or misleading → must fix
   - **Gap** — missing coverage or citation → fix if in scope, otherwise note as out-of-scope in the document
   - **Polish** — structural, stylistic, or consistency issues → fix

2. **Apply revisions.** Dispatch parallel revision agents grouped by file or area. Each agent edits corpus documents directly, following the Writing Standards. For each revision:
   - Fix the content
   - Add new citations as Local References in the edited document (not directly in `sources.md`)
   - Update or add validation artifacts if the fix involves a new or corrected claim
   - Remove validation artifacts that are no longer relevant

3. **Compile and update.** Once all revision agents complete:
   - **Compile `sources.md`**: collect any new Local References into the existing bibliography, deduplicate, enforce canonical key consistency in all inline citations (same as the Phase 1 assembly step), and strip Local References sections back to bare link definitions.
   - Update `research/index.md` if the hierarchy changed.

## Convergence

After revising, run another review phase. Continue the review-revise loop until:

- **No Error-severity findings** remain, AND
- **No Gap-severity findings** remain that are in scope, AND
- A maximum of **3 review-revise iterations** have been completed (to bound cost)

If the cap is hit with outstanding issues, note them in a `## Known Limitations` section in `research/index.md`.

## Inline Citation Format

Use Markdown reference-style links with **lowercase, short, mnemonic keys** — typically `[firstauthor-lastnameYY]` or a well-known abbreviation:

```markdown
The protocol guarantees exactly-once delivery under partial failure [cl83][cl83].

[cl83]: ../sources.md#cl83
```

Rules:
- **One canonical key per source.** No aliases. Every document uses the same key for the same source.
- Keys are always lowercase (e.g., `[lamport98]`, not `[Lamport98]` or `[LAM98]`).
- The inline display text and the link target must match: `[key][key]`, not `[Different Text][key]`.
- Each document includes bare link definitions at the bottom pointing to `sources.md` anchors.

`sources.md` entries use HTML anchor tags and should include: author(s), title, year, venue, and a URL or DOI. For documentation URLs, include an "(accessed YYYY-MM-DD)" date.

## Validation Artifacts

Scripts and data in `data/` subfolders serve as evidence. Rules:

- Include a comment or header explaining what claim the artifact validates
- Scripts should be runnable (include dependency notes if needed)
- Remove artifacts when the claim they validated is removed or substantially rewritten such that the artifact no longer applies
- Prefer small, self-contained scripts over large frameworks
- The top-level `index.md` should list all data scripts in a "Supplementary Code" section

## Version Control

The output directory is initialized as a git repository. Commit at each checkpoint:

| Checkpoint | Commit message format |
|---|---|
| After Phase 1 (Author) | `author: initial corpus for "<topic>"` |
| After each Phase 3 (Revise) | `revise: iteration N — <summary of changes>` |
| Final convergence | `final: research complete — <one-line summary>` |

Rules:
- Initialize the repo (`git init`) before writing any files.
- Stage all corpus files (`.md`, scripts, data) — do not commit anything outside the output directory.
- Each commit should represent a coherent snapshot of the corpus. Do not commit mid-phase.
- The commit history serves as a record of how the research evolved through review cycles.

## Output

The final deliverable is the corpus directory itself — a self-contained, interlinked set of markdown documents with citations and supporting evidence, rooted at `index.md`, with a git history showing the evolution through author-review-revise cycles.
