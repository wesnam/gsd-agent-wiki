---
name: kb-curate
description: "Integrate inbox candidates into wiki pages on a branch, behind a human-reviewed diff"
argument-hint: "[candidate-path]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

<objective>
Process `inbox/` candidates into canonical `wiki/` knowledge — but ONLY on a `kb-update/*` git branch (never `main`), and ONLY behind an explicit human review of the diff. For each candidate item-section, `/kb-curate` proposes a merge target or a new-page decision per the page-creation heuristic, produces or edits an OKF-conformant page matching `/wiki/concepts/okf-page-shape.md`, maintains absolute cross-links and backlinks, applies supersession-over-deletion, records contradictions rather than silently choosing a winner, updates the relevant `index.md`, archives each processed candidate into `log.md` newest-first, and then presents the full diff for human approval before any `wiki/` commit. `/kb-curate` is the ONLY path that writes to canonical `wiki/` — it is the human-in-the-loop safety gate the entire KB safety model depends on.
</objective>

<non_negotiable_gates>
Three gates are absolute. None may be skipped, reordered so a write precedes them, or auto-satisfied by the tool. They exist because curate is the sole promotion path from staged candidate to canonical `wiki/` — the highest-integrity boundary in the KB.

1. **`kb-update/*` branch, never `main` (CURATE-02).** Before ANY `wiki/` write, curate creates or switches to a `kb-update/*` branch. Curate NEVER writes to `wiki/` on `main`. The exact branch pattern is `kb-update/*` (e.g. `kb-update/2026-07-11-okf-page-shape`).

2. **Human diff-approval before any `wiki/` commit (CURATE-05).** Curate ALWAYS proposes and shows; the human ALWAYS approves before commit. Before any `wiki/` commit, curate presents the full diff of the staged changes and requires explicit human approval via `AskUserQuestion`. There is no silent auto-merge to `wiki/` — no canonical write is committed without the human approving the shown diff.

3. **Supersession over deletion, never silent overwrite (CURATE-04).** When new evidence replaces an old claim, curate marks the old claim `superseded` with the supersession DATE and a POINTER to the replacing page/claim — it NEVER silently deletes or overwrites. Uncertain or contradictory claims are RECORDED for human review, never adjudicated by the tool.
</non_negotiable_gates>

<read_time_contract>
Curate parses each candidate against `/templates/candidate-template.md` — the read-time contract mirroring capture's write-time contract. A candidate carries **file-level frontmatter** (`type: candidate`, `memory_type`, `title`, `description`, `generated`, `sources`) plus **one repeatable `## Item N — <short claim>` body section per promoted item**. Each item-section restates its own `claim`, `memory_type`, `evidence`, `confidence`, `reuse_rationale`, and `possible_conflicting_pages`. Curate processes each item-section **independently** — one merge/new-page proposal per item — all surfaced together in the single human-reviewed diff.
</read_time_contract>

<process>
1. **Branch gate (CURATE-02) — do this FIRST, before any `wiki/` write.** Create or switch to a `kb-update/*` branch via `Bash` git (e.g. `kb-update/<date>-<slug>`, where `<slug>` names the candidate or the concept being integrated). Confirm the working branch is a `kb-update/*` branch before touching a single `wiki/` file. **Non-negotiable: curate NEVER writes to `wiki/` on `main` — all candidate integration happens on a `kb-update/*` branch.** If `git status` shows the branch is `main` (or any non-`kb-update/*` branch), stop and switch before proceeding.

2. **Read candidates.** Load `inbox/session-extracts/` candidates (the argument may name a specific candidate path, else process the pending candidates in that directory). Permissively tolerate `inbox/manual-notes/` — do NOT choke on manually-authored notes if present (permissive-consumer behavior; a dedicated manual-notes workflow is out of scope). Parse each candidate's item-sections against `/templates/candidate-template.md` — extracting per item its `claim`, `memory_type`, `evidence`, `confidence`, `reuse_rationale`, and `possible_conflicting_pages`.

3. **Route per item using the page-creation heuristic (CURATE-03; `/schemas/ingestion-rules.md` §3).** For EACH item-section, PROPOSE a target — curate always proposes a merge target or a new-page decision and surfaces it in the reviewed diff; it never auto-merges silently.
   - **Map `memory_type` to the wiki category directory** — `concept | pattern | lesson | preference | decision` → `wiki/<memory_type>/`.
   - **Merge / edit-in-place** into an existing `wiki/<memory_type>/` page when one fits: for an attribute update or refinement of an existing concept, EDIT the existing page in place rather than spawning a near-duplicate. Prefer one authoritative page per concept.
   - **Create a NEW page** only for a distinct, linkable concept that no existing page covers.
   - **Output must MATCH `/wiki/concepts/okf-page-shape.md` exactly.** Frontmatter: a non-empty `type` equal to the wiki category, plus `title`, `description`, `sources` (carried from the candidate's `sources`), `generated` (`{ by, at }`, `at` = now as ISO 8601 with offset), `status: stable`, `confidence`, and an absolute `/`-prefixed `related:` list. Do NOT write `verified` yet: it is stamped only in step 6, after the human approves the diff. Body: a `#` H1 title then `##` sections. **All cross-links are absolute bundle-relative (`/wiki/...`) — never `../`, never `[[wikilinks]]`.** A path with no trailing `.md` (e.g. `/inbox/`) is a directory pointer, not a page cross-link.
   - **Maintain BOTH link directions — backlinks.** When a new/edited page adds a forward `related:` link to another page, add the reciprocal backlink on the page pointed to, so cross-links stay bidirectional.

4. **Supersession / contradiction gate (CURATE-04; `/schemas/ingestion-rules.md` §4).** When a new candidate's evidence replaces an old claim, mark the old claim `superseded` with the supersession DATE and a POINTER to the replacing page/claim — **NEVER silently delete or overwrite.** The superseded trail is what keeps the KB honest and auditable. When a candidate's `possible_conflicting_pages` flags an uncertain contradiction, **RECORD the contradiction for human review** in the diff rather than silently choosing a winner — the human diff-review gate is the backstop that adjudicates.

5. **Index + log maintenance (CURATE-04).** Update the relevant reserved files — they carry **NO frontmatter** (first line never `---`):
   - **`wiki/<category>/index.md`** — add or refresh the one-liner for the page, in the exact format `- [Title](/wiki/<category>/<slug>.md) — one-line description` (lifted from the page's `description`).
   - **`log.md`** — archive each processed candidate by **PREPENDING a NEWEST-FIRST `## YYYY-MM-DD` section** above older ones, with a bullet naming what was processed and where it landed. Never append below older entries; newest is always at the top.

6. **Human diff-review gate (CURATE-05) — the backstop, before any commit.** BEFORE any `wiki/` commit, present the FULL diff of the staged changes (via `Bash` `git diff` / `git diff --staged`) and require EXPLICIT human approval via `AskUserQuestion`. **Non-negotiable: no canonical `wiki/` write is committed without the human approving the shown diff.** Curate ALWAYS proposes and shows; the human ALWAYS approves before commit; there is no silent auto-merge to `wiki/`. Only after explicit approval does curate commit on the `kb-update/*` branch. If the human declines, curate does not commit — it revises or stops per the human's direction.
   - **Stamp trust only after approval.** Once the human approves, add `verified: [{ by: human:<id>, at: <now, ISO 8601 with offset> }]` to every page written or materially edited in this run (append to an existing list rather than replacing it), then commit. `<id>` comes from `git config user.name`. Never stamp a `human:` actor before approval and never on a rejected diff: trust tiers derive from this field.
</process>

<gates>
- **Branch isolation (CURATE-02)** — every `wiki/` write happens on a `kb-update/*` branch; curate NEVER writes to `wiki/` on `main`.
- **OKF output fidelity (CURATE-03)** — every page curate writes matches `/wiki/concepts/okf-page-shape.md` frontmatter + body shape, with `type` == the category dir, absolute `/wiki/...` cross-links, and reciprocal backlinks.
- **Supersession over deletion (CURATE-04)** — old claims are marked `superseded` with date + pointer, never deleted; contradictions are recorded for human review, never adjudicated by the tool.
- **Reserved-file maintenance (CURATE-04)** — the relevant `index.md` one-liner is updated and each candidate is archived into `log.md` newest-first; reserved files carry no frontmatter.
- **Human diff-approval (CURATE-05)** — the full diff is presented and explicit human approval obtained via `AskUserQuestion` before ANY `wiki/` commit.
</gates>
