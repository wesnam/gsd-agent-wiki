---
type: preference
title: Prefer Boring Technology
description: Default to well-understood, widely used tools unless a new one removes a concrete, named problem. This is a personal judgment, not a measured fact.
generated: { by: claude/sonnet-5.5, at: 2026-10-22T00:00:00Z }
status: stable
confidence: speculative
related:
  - /wiki/decisions/store-knowledge-as-markdown-in-git.md
---

# Prefer Boring Technology

**Judgment, not fact.** This records a standing preference. It is deliberately marked `speculative` so it is never read as established evidence.

## The preference

When two options solve the problem, choose the one with more users, more documentation, and fewer surprises. Pick the newer tool only when you can name the specific problem it removes.

## Reasoning

- Mature tools have known failure modes and answers already written down.
- Novelty has a cost each team pays once: learning, debugging, and hiring.
- Most projects have a small budget for unfamiliar technology; spend it where it matters.

## Exceptions

- The boring option cannot meet a hard requirement.
- The project exists to learn or evaluate the new tool.

## Related

- [Store Knowledge as Markdown in Git](/wiki/decisions/store-knowledge-as-markdown-in-git.md): an example of applying this preference.
