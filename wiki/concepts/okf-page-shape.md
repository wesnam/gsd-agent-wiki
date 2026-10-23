---
type: concept
title: OKF Page Shape
description: What a conformant OKF page looks like in this bundle — required type, recommended fields, reserved-vs-non-reserved distinction, and absolute cross-links.
generated: { by: claude/sonnet-5.5, at: 2026-10-22T00:00:00Z }
status: stable
confidence: verified
related:
  - /templates/page-template.md
  - /wiki/decisions/store-knowledge-as-markdown-in-git.md
---

# OKF Page Shape

A conformant page in this OKF v0.2 bundle is a plain Markdown file whose durable structure follows a small set of conventions. This page is both the definition of that shape and a sample content page that follows the rules it describes.

## Required and recommended frontmatter

Every **non-reserved** page opens with a YAML frontmatter block (first line `---`) carrying a non-empty `type`. `type` is the only field a consumer may rely on being present. For a page under `wiki/<category>/`, `type` is the singular category name. Recommended fields are `title` and `description`. OKF v0.2 adds optional provenance, trust, and lifecycle fields, which this bundle uses on every page: `generated` (who produced the content and when), `verified` (a list of `{ by, at }` events whose actors derive the trust tier), and `status` (`draft`, `stable`, or `deprecated`). `sources` (where a claim came from) and `stale_after` are available when a page needs them. This bundle also keeps two extension keys that OKF permits: `confidence` (separating verified fact from personal judgment) and `related` (a list of absolute bundle-relative links). See [the page template](/templates/page-template.md) for the full field-by-field reference.

## Reserved vs non-reserved

**Reserved** files — every `index.md` (root and each `wiki/` subdir) and `log.md` — carry **no** frontmatter; their first line is never `---`. They are progressive-disclosure listings and the audit trail, not knowledge pages. **Non-reserved** pages (real wiki pages, the template, the ingestion schema) always carry frontmatter with a non-empty `type`. The two root docs `CLAUDE.md` and `README.md` are in-bundle but frontmatter-exempt.

## Cross-links

Cross-links are **absolute bundle-relative Markdown paths** with a leading `/` (e.g. `/wiki/concepts/okf-page-shape.md`) — never relative `../` paths and never `[[wikilinks]]`. A path with no trailing `.md` (e.g. `/inbox/`) is a directory pointer, not a page cross-link.
