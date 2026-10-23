---
type: template
title: KB Candidate Template
description: The copyable frontmatter shape for an inbox/session-extracts candidate produced by /kb-capture.
status: stable
related:
  - /schemas/ingestion-rules.md
---

# KB Candidate Template

This is the canonical definition of the **inbox candidate** shape. `/kb-capture` copies the frontmatter below when writing a candidate into `inbox/session-extracts/YYYY-MM-DD-<project>-<phase>.md`; `/kb-curate` reads candidates against it. It mirrors how `templates/page-template.md` anchors wiki pages: one authoritative, greppable shape rather than a shape buried inside a skill.

This file is itself a non-reserved page (`type: template`). It is an **exemplar, not real knowledge**; `/kb-lint` excludes `templates/` from broken-link and orphan checks.

## Prescribed candidate frontmatter

Copy this block as the **file-level** frontmatter of a candidate. `/kb-capture` writes it; `/kb-curate` reads against it.

```yaml
---
type: candidate
memory_type: concept | pattern | lesson | preference | decision   # target wiki category
title: <human-readable candidate title>
description: <one-line summary of what this capture run promoted>
generated: { by: kb-capture/1.0, at: YYYY-MM-DDTHH:MM:SSZ }
sources:                      # the paths the claims were captured from
  - resource: <path>
claim: <the durable, reusable assertion>
confidence: verified | probable | speculative
projects:                     # optional; project/repo tags if the claim is project-specific, else omit
  - <project-tag>
reuse_rationale: <why this recurs across projects, or why it matters for the tagged project>
possible_conflicting_pages:   # list of existing wiki pages this may contradict
  - <path or "none">
---
```

### Candidate field table

| Field | Required? | Meaning |
|-------|-----------|---------|
| `type` | **Required, non-empty** | The only field a consumer may rely on. For a candidate this is always `candidate`. |
| `memory_type` | **Required** | The **target wiki category**: `concept`, `pattern`, `lesson`, `preference`, or `decision`. It says where this is going. The file itself is always `type: candidate`. |
| `title` | Recommended | Human-readable candidate title. |
| `description` | Recommended | One-line summary, read before the full body in tiered retrieval. |
| `generated` | **Required** | `{ by, at }`: the capture run that produced the file, with an ISO 8601 datetime. |
| `sources` | **Required** | A list of `{ resource }` entries naming the project-relative or absolute paths that substantiate the claim. `/kb-curate` carries these onto the promoted page. |
| `claim` | **Required** | The durable, reusable assertion, stated so a future project can apply it. |
| `confidence` | **Required (extension)** | Separates fact from personal judgment: `verified`, `probable`, or `speculative`. |
| `projects` | Optional (extension) | Project/repo tags (lowercase kebab-case) when the claim is specific to those projects or products. Omit for general, cross-project knowledge. `/kb-curate` carries it onto the promoted page. |
| `reuse_rationale` | **Required** | Why this recurs across projects (or, for a `projects`-tagged item, across work on that project), which justifies promoting it past the promotion filter. |
| `possible_conflicting_pages` | **Required** | Existing `wiki/` pages this claim may contradict (so `/kb-curate` can apply supersession-over-deletion), or `none`. |

**Enum values (exact):**

- `memory_type`: `concept` | `pattern` | `lesson` | `preference` | `decision`
- `confidence`: `verified` | `probable` | `speculative`

A candidate carries no `verified` field. Verification is added only by `/kb-curate`, after a human approves the diff.

## One candidate file, multiple item-sections

A single capture run may promote **several** durable items from one phase's `LEARNINGS.md`. CAPTURE-03 locks **exactly ONE candidate file** per run — so multiple promotable items live as **repeatable body sections under one file-level frontmatter block**, not as multiple frontmatter-fenced blocks.

**Shape:** one file-level frontmatter block at the top (above), then one `## Item N — <short claim>` body section per promoted item. Each item-section **restates its own** `claim`, `memory_type`, `evidence`, `confidence`, `reuse_rationale`, and `possible_conflicting_pages` so `/kb-capture` writes and `/kb-curate` reads a consistent per-item block. The file-level frontmatter describes the capture run as a whole; each item-section carries the per-item contract.

```markdown
## Item 1 — <short summary of the claim>

- **claim:** <the durable, reusable assertion>
- **memory_type:** concept | pattern | lesson | preference | decision
- **evidence:**
  - <path>
- **confidence:** verified | probable | speculative
- **projects:** <project-tag, ... | omit if general>
- **reuse_rationale:** <why this recurs across projects>
- **possible_conflicting_pages:**
  - <path or "none">

## Item 2 — <short summary of the next claim>

- **claim:** ...
- **memory_type:** ...
- **evidence:**
  - <path>
- **confidence:** ...
- **reuse_rationale:** ...
- **possible_conflicting_pages:**
  - none
```

`/kb-curate` processes each item-section independently — proposing a merge target or a new-page decision per item — behind the human-reviewed diff.

## Cross-link rule

All cross-links — in the body — MUST be **absolute bundle-relative Markdown paths** with a leading `/`, e.g. `/wiki/concepts/x.md`. Never use relative `../` paths and never use `[[wikilinks]]`.
