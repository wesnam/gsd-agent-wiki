---
type: pattern
title: Supersession Over Deletion
description: When new evidence replaces an old claim in a durable store, mark the old claim superseded with a date and a pointer instead of deleting or silently overwriting it.
generated: { by: claude/sonnet-5.5, at: 2026-10-22T00:00:00Z }
status: stable
confidence: probable
related:
  - /schemas/ingestion-rules.md
  - /wiki/patterns/idempotency-keys-for-retried-writes.md
---

# Supersession Over Deletion

When a claim in a long-lived store is replaced by better evidence, keep the old claim and mark it `superseded` with the **date** and a **pointer** to its replacement.

## Why

- Deleting hides the fact that the claim was ever believed. The next person who finds the same stale idea in a blog post or an old branch has no record that it was already refuted.
- Silent overwrites make history unauditable: a reader cannot tell a refinement from a reversal.
- A superseded trail turns past mistakes into searchable lessons.

## How

1. Leave the old text in place.
2. Add a marker near it: `Superseded 2026-03-02 by [new page](/wiki/...)`.
3. Put the new claim on the replacing page, stating what changed and what evidence changed it.
4. Set the old page's `status: deprecated` if the whole page is replaced.

## When not to

Do not keep secrets, personal data, or content that is legally required to be removed. Deletion is correct there; record only that a removal happened.

## Related

- [Ingestion rules](/schemas/ingestion-rules.md): the rule this bundle applies when curating.
- [Idempotency Keys for Retried Writes](/wiki/patterns/idempotency-keys-for-retried-writes.md): another pattern about making history safe to replay.
