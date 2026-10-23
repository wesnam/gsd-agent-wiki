---
name: kb-capture
description: "Promote durable knowledge from a completed phase's LEARNINGS.md into one inbox candidate"
argument-hint: "<phase-number-or-LEARNINGS-path>"
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  - Glob
---

<objective>
Read a completed GSD phase's `LEARNINGS.md`, apply the promotion filter from `/schemas/ingestion-rules.md`, and write EXACTLY ONE conformant candidate into `inbox/session-extracts/YYYY-MM-DD-<project>-<phase>.md` — copying the candidate frontmatter and per-item shape prescribed by `/templates/candidate-template.md`. `/kb-capture` is the ingress that turns a verified phase's durable learnings into staged candidate knowledge, isolated in `inbox/` behind the human-reviewed curate gate (`/kb-curate`, a separate step). It NEVER writes to `wiki/` (CAPTURE-04) — capture stops at inbox.
</objective>

<write_path_confinement>
This skill carries `Write` in its `allowed-tools` because it MUST create the candidate file. That capability is confined by a **non-negotiable gate**: every write this skill performs goes to exactly one path under `inbox/session-extracts/`. **`/kb-capture` NEVER writes to `wiki/` (CAPTURE-04); it never creates, edits, or deletes any canonical page.** Promoting a candidate to a canonical `wiki/` page is the job of the separate, human-gated `/kb-curate` skill (Plan 03). Capture's entire output surface is a single inbox candidate file — nothing else in the bundle is touched.
</write_path_confinement>

<input_contract>
`LEARNINGS.md` (produced by GSD's `gsd-extract-learnings`) is the input this skill parses. It carries:
- **YAML frontmatter** — per-category counts and `missing_artifacts`.
- **Four `##`-heading categories** — `Decisions`, `Lessons`, `Patterns`, `Surprises`. Each item under a category carries a `Source:` attribution line pointing at the artifact it came from (e.g. `Source: 03-01-SUMMARY.md`).

Capture reads these four categories and their per-item `Source:` lines; the `Source:` paths become the candidate's `evidence`.
</input_contract>

<process>
1. **Resolve the input `LEARNINGS.md`.** The argument is either a phase number or a direct path to a `LEARNINGS.md`. For a phase number `N`, the phase directory is the SLUGGED directory (e.g. `.planning/phases/02-capture-curate-lint-skills/`), not a bare-number directory — resolve it by globbing `.planning/phases/<padded-N>-*/` (padded to two digits) and reading `<padded-N>-LEARNINGS.md` inside the matched directory (e.g. `.planning/phases/02-capture-curate-lint-skills/02-LEARNINGS.md`). Do NOT assume `.planning/phases/<N>/`. Read the file. Parse its four `##` categories — `Decisions`, `Lessons`, `Patterns`, `Surprises` — and, for each item, its `Source:` attribution line(s).

   **If `LEARNINGS.md` is absent, stop — capture has no input — and report the precise remediation rather than a bare error.** Do NOT generate the LEARNINGS file yourself, and do NOT auto-run another command: producing LEARNINGS is a separate, GSD-specific step (this keeps `/kb-capture` harness-independent — the KB must stay usable when GSD is absent). Report which path was checked and, when the matched phase directory exists (so this is a GSD project), tell the user to generate the input first:

   ```
   No LEARNINGS.md found at <resolved-path>.
   /kb-capture has no input to promote.

   If this is a GSD project, produce it first, then re-run capture:
     /gsd-extract-learnings <padded-N>   →  writes <phase-dir>/<padded-N>-LEARNINGS.md
     /kb-capture <N>

   Otherwise, pass a direct path to an existing LEARNINGS.md:
     /kb-capture <path-to-LEARNINGS.md>
   ```

   Substitute the actual resolved path, padded phase number, and phase directory into the message. `/gsd-extract-learnings` is named as a convenience for GSD users only — it is NOT a dependency of this skill.

2. **Apply the promotion filter** (`/schemas/ingestion-rules.md` §1). Keep only **PROMOTABLE** items: durable, reusable, generalizable knowledge with evidence citations that will recur across projects. Drop **EXCLUDED** items: one-off session detail, unverified speculation, raw chat/history transcripts, and ephemeral project detail. Durable knowledge specific to this project/product is promotable, but must be tagged (next step). **Bias toward FEWER, clearly-durable items.** A noisy inbox defeats the human-review gate — and curate can always be re-run, so under-capturing is cheaper than over-capturing. If nothing clears the filter, write no candidate and say so.

3. **Map each promoted item's category to a `memory_type`** using the D-04 default mapping:
   - `decision → decision`
   - `lesson → lesson`
   - `pattern → pattern`
   - `surprise → lesson` (a surprise is a lesson about the unexpected)
   - `concept` / `preference` are **inferred from the item's content**: use `concept` when the item is a definition of a linkable idea; use `preference` when the item is a standing choice / stated preference. There is no `concept` or `preference` LEARNINGS category — these come only from content inference.

   **Project tag.** For each item, decide whether the claim is general or specific to this repo/project. If specific, set `projects: [<repo-name-kebab-case>]` (reuse tags already present in `wiki/`, check with `rg "^projects:" -A3 wiki/`); if general, omit the field. When unsure, tag.

4. **Write EXACTLY ONE candidate file** to the path pattern:
   ```
   inbox/session-extracts/YYYY-MM-DD-<project>-<phase>.md
   ```
   where `YYYY-MM-DD` is today's ISO-8601 date, `<project>` is the repo/project name, and `<phase>` is the source phase. Copy the **file-level frontmatter** shape from `/templates/candidate-template.md` (`type: candidate`, `memory_type`, `title`, `description`, `generated`, `sources`), then write **one repeatable body item-section per promoted item**, each restating its own `claim`, `memory_type`, `evidence` (the item's `Source:` paths), `confidence` (`verified | probable | speculative`), `projects` (optional), `reuse_rationale`, and `possible_conflicting_pages`. **Do NOT re-specify the candidate shape here — `/templates/candidate-template.md` is the authoritative contract; copy its prescribed frontmatter and its `## Item N — <claim>` per-item block verbatim in structure.** All cross-links in the candidate are absolute bundle-relative (`/`-prefixed), never `../`, never `[[wikilinks]]`.

5. **Confine ALL writes to `inbox/session-extracts/`.** Restate the gate before writing: **`/kb-capture` NEVER writes to `wiki/` (CAPTURE-04); it never creates or edits any canonical page.** Curation of the candidate into a canonical `wiki/` page is a separate, human-gated step (`/kb-curate`, Plan 03) — capture stops at inbox. Verify the resolved write path begins with `inbox/session-extracts/` before calling `Write`; if it does not, stop.
</process>

<one_file_multiple_items>
**CAPTURE-03 reconciliation — one FILE, many item-sections.** A single capture run produces **exactly ONE candidate file**, even when several items clear the promotion filter. Multiple promoted **items** become multiple `## Item N — <short claim>` **body sections within that one file**, under a single file-level frontmatter block — **never multiple files, and never dropping all-but-one item to satisfy the "one file" constraint.** The file-level frontmatter describes the capture run as a whole; each item-section carries the per-item contract (`claim` / `memory_type` / `evidence` / `confidence` / `reuse_rationale` / `possible_conflicting_pages`) exactly as `/templates/candidate-template.md` prescribes. `/kb-curate` later processes each item-section independently behind its human-reviewed diff.
</one_file_multiple_items>

<gates>
- **Promotion filter applied** — only durable/reusable/generalizable items with evidence; biased toward fewer, clearly-durable items (`/schemas/ingestion-rules.md` §1).
- **Write confinement** — every write lands under `inbox/session-extracts/`; **`/kb-capture` NEVER writes to `wiki/`** (CAPTURE-04).
- **One candidate file per run** — multiple items → multiple item-sections in ONE file (CAPTURE-03).
- **Contract fidelity** — frontmatter and per-item block copied from `/templates/candidate-template.md`, not re-invented.
</gates>
