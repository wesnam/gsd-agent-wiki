---
okf_version: "0.2"
---
# gsd_wiki

A personal, portable knowledge base — the durable "semantic memory" layer that complements the GSD execution harness. This is an [OKF v0.2](https://github.com/GoogleCloudPlatform/open-knowledge-format) bundle: a directory of Markdown files with YAML frontmatter, readable and writable by any human or agent with no proprietary SDK.

This root `index.md` is the cheap top of the tiered retrieval order — scan it first, then drill into a category index, then into pages. It is a reserved file and carries no frontmatter.

## Categories

- [Concepts](/wiki/concepts/index.md) — durable definitions: what a thing *is*.
- [Patterns](/wiki/patterns/index.md) — reusable approaches: how to *do* a recurring thing.
- [Lessons](/wiki/lessons/index.md) — what went wrong or right, and why.
- [Preferences](/wiki/preferences/index.md) — the developer's standing choices and defaults.
- [Decisions](/wiki/decisions/index.md) — recorded rationale behind a choice.

## Trail

- [Log](/log.md) — chronological ingestion/audit trail (newest first).

Candidate knowledge is staged under `inbox/` before curation — it is not canonical until integrated into `wiki/` through the reviewed curation flow. See the operating rules in `CLAUDE.md` and the ingestion rules in `schemas/ingestion-rules.md`.
