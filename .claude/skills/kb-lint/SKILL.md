---
name: kb-lint
description: "Report-only KB health check: frontmatter, broken links, orphans, stale index entries, contradictions"
argument-hint: ""
allowed-tools:
  - Read
  - Bash
  - Glob
  - Grep
---

<objective>
Scan this OKF v0.2 KB bundle and report drift — non-conformant frontmatter, broken absolute cross-links, orphan pages, missing/stale `index.md` entries, cross-page contradictions, and type/trust mismatches — against the written spec in `CLAUDE.md`. `/kb-lint` is **report-only**: it modifies **no files**. Drift (broken links, orphans, stale indexes, contradictions, non-conformant frontmatter) is the failure mode a human-curated LLM wiki hits first; this skill makes that drift visible for human review without ever mutating the bundle.
</objective>

<read_only_guarantee>
This skill modifies **no files**. Its `allowed-tools` list deliberately contains only `Read`, `Bash`, `Glob`, and `Grep` — it OMITS `Write` and `Edit`. That tool scope is a **structural** guarantee: the skill cannot mutate a single file even if instructed to. The report may *describe* a suggested fix in prose, but it MUST NEVER apply one. Any `Bash` usage is confined to read-only inspection (`grep`, `rg`, `ls`, `find`, `awk`, `head`) — never a command that writes, moves, or deletes.
</read_only_guarantee>

<scope>
The spec these checks validate against is `CLAUDE.md` — specifically its §"KB Operating Rules", §"Bundle Scope, Reserved Files & Lint Exclusions", and §"Permissive Consumer Behavior". Read those sections first; they are the single source of truth for every rule below.

**Bundle scope (what to lint):** `wiki/`, `inbox/`, `templates/`, `schemas/`, plus the reserved root `index.md` and `log.md`, plus the two top-level docs `CLAUDE.md` and `README.md`. Everything else at the repo root is TOOLING, not bundle content, and is OUT of scope: `.planning/`, `.claude/`, `.git/`.

**This skill introduces NO candidate-specific field check — scope is exactly these 6 checks (D-05).** `inbox/` candidate conformance (required candidate fields like `memory_type`, `claim`, `evidence`) is `/kb-capture`'s responsibility at write time and `/kb-curate`'s at read time, not lint's. Lint still applies its universal checks wherever they legitimately apply — e.g. the BUNDLE-03 `type`-frontmatter presence check already covers `inbox/` pages — but no new candidate-required-fields check type is added beyond the locked 6.
</scope>

<process>
Run EXACTLY these 6 checks — no more, no fewer (D-05). Read the relevant `CLAUDE.md` section per check, enumerate the in-scope files, evaluate each, and collect findings. Do not fix anything.

### Check 1 — Non-conformant frontmatter (BUNDLE-03)
Every **non-reserved** `.md` under `wiki/`, `templates/`, `schemas/`, `inbox/` MUST carry parseable YAML frontmatter (first line `---`) with a non-empty `type`.
- EXEMPT (never flag for missing frontmatter): `CLAUDE.md`, `README.md`.
- RESERVED files (`index.md` at any level, and `log.md`) MUST have **NO** frontmatter — their first line is never `---`. The one exception: the root `index.md` MAY carry a frontmatter block containing only `okf_version`.
- **Flag:** any required file missing frontmatter, with an empty/absent `type`, or with unparseable YAML; AND any reserved file whose first line **is** `---` (except the root `index.md` carrying only `okf_version`).

### Check 2 — Broken absolute cross-links
Every `related:` frontmatter link and every body Markdown link MUST be `/`-prefixed (absolute bundle-relative) and resolve to a real file in the bundle.
- A path with **no** trailing `.md` (e.g. `/inbox/`) is a **directory pointer**, NOT a page cross-link — do NOT flag it as broken.
- **Flag:** relative `../` links, `[[wikilinks]]`, and `/`-prefixed `.md` links that point at a file that does not exist.
- **EXCLUDE `templates/`** — template pages are exemplars, so their example `related:` links are expected, not drift (CLAUDE.md lint-exclusions).

### Check 3 — Orphan pages (no inbound links)
A real `wiki/` page with **zero** inbound links from any other page or from any `index.md`.
- **EXCLUDE `templates/`** — exemplar pages are expected to be unlinked, so their absence of inbound links is not an orphan finding (CLAUDE.md lint-exclusions).

### Check 4 — Missing/stale `index.md` entries
Every real `wiki/<category>/` page MUST be listed in its category `index.md` (one-liner format `- [Title](/wiki/<category>/<slug>.md) — description`).
- **Flag:** a page missing from its category index (missing entry); AND a dead listing in an `index.md` pointing at a file that no longer exists (stale entry).

### Check 5 — Cross-page contradictions
Surface pages that appear to contradict each other for **HUMAN review**. Report only — do NOT adjudicate which page is correct.
- Contradictions are resolved by the curate flow via **supersession over deletion** (see `schemas/ingestion-rules.md` §4): the human marks the old claim `superseded` with a date and a pointer, never deleting. Lint's job is only to make the apparent conflict visible.

### Check 6 — Type and trust correctness
For pages under `wiki/<category>/`: (a) `type` MUST equal the singular category (`concept`, `pattern`, `lesson`, `preference`, `decision`); (b) `verified` MUST contain at least one entry whose `by` starts with `human:` (the human-verifier invariant; `/kb-curate` stamps it after approval); (c) every `generated.at` and `verified[].at` MUST be an ISO 8601 datetime with an explicit UTC offset.
- **Flag:** a page whose `type` does not match its category directory; a page with no `human:` verifier; a bare date or offset-less timestamp.
- Do NOT force a category value onto a non-category page (templates are `type: template`, the ingestion schema `type: schema`, inbox candidates `type: candidate` and carry no `verified`) — the category rule applies only to real pages under `wiki/`.
</process>

<exclusions>
- **`templates/` is directory-scoped-excluded** from the broken-link (Check 2) and orphan (Check 3) checks. Because the exclusion is directory-scoped, it already covers `templates/candidate-template.md` (added in Plan 01) with no special-case needed — any file under `templates/` inherits the exclusion automatically.
- A bundle-relative link to a **directory** (a path with no trailing `.md`, e.g. `/inbox/`) is a navigational pointer, not a page cross-link, and is never a broken-link finding.
- OUT of bundle scope entirely (never linted): `.planning/`, `.claude/`, `.git/`.
</exclusions>

<report_format>
Emit a **human-readable** report, **grouped by check type** (one section per check, 1–6). Under each check, list every finding citing `file:line` where a line applies. Each finding MAY include a one-line prose suggestion of how a human could fix it — but the skill applies nothing. End the report with a **summary count line**, e.g. `Summary: 6 checks run — 0 frontmatter, 2 broken links, 1 orphan, 0 stale index, 0 contradictions, 0 type/trust = 3 findings total.` A machine-readable / exit-code mode is a deferred nice-to-have (not required by LINT-01) — the default is this human-facing grouped report. When a check has no findings, still show its section with a "clean — no findings" note so the human sees all 6 were run.
</report_format>
