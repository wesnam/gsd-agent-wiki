---
type: template
title: OKF Page Template
description: The copyable shape for a conformant OKF v0.2 page in this bundle — native frontmatter plus the confidence and related extension keys.
status: stable
related:
  - /wiki/concepts/okf-page-shape.md
---

# OKF Page Template

Copy this frontmatter as the starting point for a new wiki page, then fill in the body.

```yaml
---
type: pattern                      # singular category: concept | pattern | lesson | preference | decision
title: <Human-readable page title>
description: <One sentence saying what this page tells you.>
sources:                           # optional; where the knowledge came from
  - id: origin
    resource: <path-or-url the claim came from>
    title: <human label>
generated: { by: kb-capture/1.0, at: 2026-01-01T00:00:00Z }
verified:
  - { by: human:<id>, at: 2026-01-01T00:00:00Z }
status: stable                     # draft | stable | deprecated
stale_after: 2027-01-01T00:00:00Z  # optional
confidence: verified               # extension: verified | probable | speculative
related:                           # extension: absolute bundle-relative links
  - /wiki/concepts/okf-page-shape.md
---
```

This file is itself a non-reserved page, so it carries frontmatter with a non-empty `type`. It is an exemplar, not real knowledge: `/kb-lint` excludes `templates/` from broken-link and orphan checks.

## Frontmatter fields

| Field | Required? | Meaning |
|-------|-----------|---------|
| `type` | **Required, non-empty** | The only field a consumer may rely on. For a page under a category directory it is the singular category name. Other kinds: `template`, `schema`, `candidate`. |
| `title` | Recommended | Human-readable display name. |
| `description` | Recommended | One sentence. This is the middle tier of the retrieval order (index, **descriptions**, search, bodies): an agent decides whether to open a page from this line alone, so make it specific. |
| `sources` | Optional | List of `{ id, resource, title? }`. `resource` is required within an entry. Use `id` as a footnote label for per-claim attribution. |
| `generated` | Recommended | `{ by, at }`: the actor that produced the current content and when it last meaningfully changed. |
| `verified` | Recommended on `wiki/` pages | List of `{ by, at }`. Trust tier derives from it: absent is *unverified*, non-human actors only is *machine-confirmed*, any `human:<id>` actor is *human-reviewed*. `/kb-curate` stamps the human entry after the diff is approved. |
| `status` | Optional | `draft`, `stable`, or `deprecated`. Absent means `stable`. |
| `stale_after` | Optional | Absolute ISO 8601 instant. At or after it a reader treats the page as possibly stale. |
| `confidence` | Extension, required on `wiki/` pages | Separates fact from personal judgment: `verified`, `probable`, or `speculative`. Never present judgment as fact. |
| `related` | Extension, optional | YAML list of absolute bundle-relative links to related pages. |

### Actor forms

`human:<id>` for a person, `<producer>/<version>` for an agent, `process:<id>` for an automated process. Timestamps are ISO 8601 with an explicit UTC offset (`2026-01-01T00:00:00Z`), never a bare date.

Only `type` is relied upon. A permissive consumer must not reject a page for a missing optional field or an unknown key.

## Cross-link rule

All cross-links, in frontmatter `related:` and in the body, MUST be **absolute bundle-relative Markdown paths** with a leading `/`, e.g. `/wiki/concepts/x.md`. Never use relative `../` paths and never use `[[wikilinks]]`.
