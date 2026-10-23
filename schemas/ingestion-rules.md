---
type: schema
title: Ingestion Rules
description: The promotion filter, capture→curate lifecycle, page-creation heuristic, and supersession-over-deletion rule that govern what enters the wiki and how.
status: stable
related:
  - /templates/page-template.md
  - /wiki/patterns/supersession-over-deletion.md
---

# Ingestion Rules

How candidate knowledge becomes canonical wiki knowledge. These rules are the contract `/kb-capture` and `/kb-curate` implement; they are stated here as the intended lifecycle.

## 1. Promotion filter

The gate for whether something belongs in the KB at all:

> session → project artifact → is this reusable? → candidate → integrated

**PROMOTABLE** — durable, reusable, generalizable knowledge with evidence citations that will recur across projects: concepts, patterns, lessons, preferences, and decisions.

**EXCLUDED** — one-off session detail, unverified speculation, raw chat/history transcripts, and project-specific ephemera. If it won't help future work, it does not get promoted.

**Project scope.** The wiki serves many projects and products. Durable knowledge that is specific to one (an architecture decision, a product constraint, a repo convention) IS promotable, but MUST carry a `projects:` tag list naming the repo/project(s) it applies to. Knowledge with no `projects` tag is general and cross-project. Ephemeral per-project detail (task status, one-off debugging) stays excluded, tagged or not.

## 2. Capture → curate lifecycle

- **`/kb-capture`** reads a completed phase's `LEARNINGS.md`, applies the promotion filter, and writes ONE candidate into [`/inbox/session-extracts/`](/inbox/session-extracts/). It never touches `wiki/`.
- **`/kb-curate`** processes `inbox/` candidates on a `kb-update/*` branch, behind a human-reviewed diff, and integrates the approved knowledge into `wiki/` — stamping each page's `generated` and `verified` (human actor) fields, updating the relevant `index.md`, and archiving the processed candidate into [`/log.md`](/log.md).

Nothing reaches canonical `wiki/` without passing through the human-reviewed curate step.

## 3. Page-creation heuristic

Create a NEW page only for a distinct, linkable concept. For an attribute update or refinement of an existing concept, EDIT the existing page in place rather than spawning a near-duplicate. Prefer one authoritative page per concept over many overlapping ones.

## 4. Supersession over deletion

When new evidence replaces an old claim, mark the old claim `superseded` with the supersession DATE and a POINTER to the replacing page or claim. NEVER silently delete. Forgetting errors is how they get repeated — the superseded trail is what keeps the KB honest and auditable.

## 5. Project tags

- `projects` is a list of lowercase kebab-case slugs, one per repo/project/product (use the repo name; stay consistent with tags already in the wiki).
- Capture sets `projects` when the claim depends on a specific project; omit it when the claim would hold anywhere. When unsure, tag — curate can widen it.
- Curate preserves tags, unions them when merging into an existing page, and asks the human to confirm any tag change in the diff.
- Retrieval: when working in project `X`, read pages tagged `X` plus untagged (general) pages; pages tagged only for other projects are skipped unless the topic is directly relevant. `rg -l "^  - X$" wiki/` or `rg "^projects:"` finds tagged pages.
- A `projects` tag narrows applicability; it never changes trust. Conflicts follow the usual rule: current-project evidence wins.
