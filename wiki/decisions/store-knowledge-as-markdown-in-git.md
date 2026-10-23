---
type: decision
title: Store Knowledge as Markdown in Git
description: This bundle keeps knowledge as plain Markdown with YAML frontmatter in a git repository, rather than in a database or a hosted service.
generated: { by: claude/sonnet-5.5, at: 2026-10-22T00:00:00Z }
status: stable
confidence: verified
related:
  - /wiki/preferences/prefer-boring-technology.md
  - /wiki/concepts/okf-page-shape.md
---

# Store Knowledge as Markdown in Git

## Decision

Knowledge lives as Markdown files with YAML frontmatter in a git repository, following [OKF v0.2](https://github.com/GoogleCloudPlatform/open-knowledge-format).

## Options considered

| Option | For | Against |
|---|---|---|
| Markdown in git | Readable anywhere, diffable, reviewable, no vendor lock-in | No built-in search or access control |
| Hosted notes app | Good search and editing UI | Lock-in, export friction, hard for agents to write safely |
| Database | Fast structured queries | Needs a server, opaque to humans, awkward review |

## Why

- Review is the core safety mechanism here. A git diff is the review surface.
- Agents and humans can both read and write files with no SDK.
- The bundle survives tool changes: it is a folder.

## Consequences

- Search is `rg` over files plus the tiered `index.md` reading order.
- Indexes and cross-links are maintained by convention, so a lint pass is needed.
- Access control is whatever the git host provides.

## Related

- [Prefer Boring Technology](/wiki/preferences/prefer-boring-technology.md)
- [OKF Page Shape](/wiki/concepts/okf-page-shape.md)
