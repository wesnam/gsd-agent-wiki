## Project

**gsd-agent-wiki**

A personal, portable knowledge base — the durable "semantic memory" layer that complements the GSD execution harness. GSD owns execution/working memory (STATE.md, phase artifacts, per-project planning); gsd-agent-wiki owns long-term semantic memory: concepts, patterns, lessons, preferences, and decisions learned across all projects and over time. The wiki is a [Google Open Knowledge Format (OKF) v0.2](https://github.com/GoogleCloudPlatform/open-knowledge-format) bundle — a directory of Markdown files with YAML frontmatter, readable and writable by any human or agent with no proprietary SDK.

**Core Value:** GSD remembers where the work is; gsd-agent-wiki remembers what you've learned. The KB must influence GSD planning (retrieve before big decisions) and receive promoted knowledge after verified phases — while GSD never depends on the KB to function, and the KB never depends on GSD to be readable.

### Constraints

- **Format**: OKF v0.2 conformance is strict — Markdown + YAML only; the bundle must be readable/writable with no proprietary SDK.
- **Compatibility**: The bundle must remain fully usable without GSD installed (portable as git repo / tarball / filesystem mount).
- **Harness independence**: KB curation logic (skills) ships inside the KB repo, never inside the GSD plugin — must survive `gsd-update` and a harness swap.
- **Safety**: Human-in-the-loop before any `wiki/` commit; `inbox/` staging isolates candidate knowledge from canonical pages.
- **Provenance**: Pages separate fact from personal judgment (the `confidence` extension field) and cite evidence (`sources`).
- **Tech**: Markdown, YAML, git; Python only for optional later helper scripts. macOS. No server.

## KB Operating Rules

These rules govern how any agent reads and writes this OKF v0.2 knowledge bundle. They are directives, not suggestions.

1. **OKF conformance.** The bundle is OKF v0.2 ([spec](https://github.com/GoogleCloudPlatform/open-knowledge-format)). `type` is the only required-present field on non-reserved pages (OKF §4.1, §11). Reserved files (`index.md`, `log.md`) carry no frontmatter (OKF §8), except that the root `index.md` carries `okf_version: "0.2"`. Trust is recorded in `generated` and `verified` (OKF §5): `/kb-curate` stamps a `human:<id>` entry in `verified` only after the human approves the diff, and consumers derive the trust tier from it. Cross-links are absolute bundle-relative Markdown paths with a leading `/` (e.g. `/wiki/concepts/x.md`).
2. **Fact-vs-judgment discipline.** Separate verified fact from personal judgment. Use the `confidence` frontmatter field to record how sure a claim is. Never present judgment as fact.
3. **Prefer current-project evidence on conflict.** When KB knowledge conflicts with evidence in the project currently being worked, prefer the current project's evidence and record the conflict — do not silently trust the older KB claim.
4. **Never rewrite canonical `wiki/` pages during normal execution.** Canonical writes happen only through the `/kb-curate` human-reviewed flow , never as a side effect of a normal coding session.
5. **Project scoping.** The KB spans many projects and products. Page content specific to a repo/project carries a `projects:` tag list; untagged pages are general. When working in project `X`, consider pages tagged `X` and untagged pages first; skip pages tagged only for other projects unless clearly relevant. See [the ingestion rules](/schemas/ingestion-rules.md) §5.
6. **Tiered retrieval order.** To keep query cost flat as the wiki grows, read in this order: (a) `index.md` files, then (b) page frontmatter `description` fields, then (c) `rg` full-text search, then (d) full page bodies only for the finalists. The order, stated as one literal string, is: `index -> frontmatter descriptions -> rg full-text -> page bodies`.

The capture → curate lifecycle and promotion filter are defined in [the ingestion rules](/schemas/ingestion-rules.md).

## Bundle Scope, Reserved Files & Lint Exclusions

This is the single written spec `/kb-lint` checks against.

- **Bundle scope (what "the bundle" IS):** the KB content is `wiki/`, `inbox/`, `templates/`, `schemas/`, plus the reserved root `index.md` and `log.md`, plus the two top-level docs `CLAUDE.md` and `README.md`. Everything else at the repo root is TOOLING, not bundle content, and is out of scope for OKF conformance and lint: `.planning/` (GSD planning artifacts), `.claude/` (skills/tooling), and `.git/`.
- **Reserved files (NO frontmatter, OKF §8):** every `index.md` (root and each `wiki/` subdir) and `log.md`. Their first line is never `---`, except the root `index.md`, which carries only `okf_version`.
- **Frontmatter-EXEMPT bundle docs:** `CLAUDE.md` and `README.md` are in-bundle docs (they are normal bundle docs) but are EXEMPT from the "every non-reserved `.md` carries `type` frontmatter" rule (BUNDLE-03). Reason: `CLAUDE.md` is loaded verbatim as agent instructions, so YAML frontmatter would be noise; `README.md` is a human landing doc. Both `CLAUDE.md` and `README.md` are named exempt so `/kb-lint`'s frontmatter check has a written spec and does not flag them.
- **Frontmatter-REQUIRED files:** all other non-reserved `.md` under `wiki/`, `templates/`, `schemas/`, and `inbox/` (real pages, the template, the ingestion schema, and future candidates) MUST carry parseable frontmatter with a non-empty `type` (BUNDLE-03).
- **`type` and `confidence`:** for a real `wiki/` page `type` MUST equal the singular wiki category (`concept`, `pattern`, `lesson`, `preference`, `decision`). Non-category pages use their own kind (`template`, `schema`, `candidate`). `confidence` (`verified`, `probable`, `speculative`), `projects` (optional list of kebab-case project/repo tags; absent = general cross-project knowledge) and `related` are extension keys this bundle adds on top of OKF v0.2; OKF permits unknown keys.
- **`/kb-lint` exclusions:** `templates/` is EXCLUDED from broken-cross-link and orphan-page checks — template pages are exemplars, not real knowledge pages, so their example `related:` links and their absence from any `index.md` are expected, not drift. A bundle-relative link to a DIRECTORY (a path with no trailing `.md`, e.g. `/inbox/`) is a navigational pointer, not a page cross-link, and is likewise not a broken-link finding.

## Permissive Consumer Behavior

Any reader or consumer of this bundle — human or agent — MUST read it permissively and degrade gracefully. It MUST NOT reject or error on:

- missing optional fields,
- unknown `type` values,
- unknown frontmatter keys,
- broken cross-links, or
- missing `index.md` files.

Only `type` is relied upon to be present on non-reserved pages; reserved files (`index.md`, `log.md`) intentionally have no frontmatter (OKF §8, above). OKF §11's permissive-consumer clause is what requires a *reader* to degrade gracefully rather than error — the two concerns are distinct. See [the ingestion rules](/schemas/ingestion-rules.md) for the capture/curate lifecycle.
