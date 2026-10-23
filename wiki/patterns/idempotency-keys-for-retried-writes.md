---
type: pattern
title: Idempotency Keys for Retried Writes
description: Attach a client-generated unique key to every non-idempotent write so a retry after a timeout cannot apply the same operation twice.
generated: { by: claude/sonnet-5.5, at: 2026-10-22T00:00:00Z }
status: stable
confidence: probable
related:
  - /wiki/patterns/supersession-over-deletion.md
---

# Idempotency Keys for Retried Writes

A timeout does not tell you whether the server did the work. Retrying a non-idempotent request (create an order, send a payment, post a message) can therefore apply it twice.

## Pattern

1. The client generates a unique key (a UUID) **once per logical operation**, before the first attempt.
2. Every retry of that operation sends the same key, usually in a header.
3. The server records the key with the result of the first successful attempt.
4. On a repeat key it returns the stored result instead of doing the work again.

## Details that matter

- **Scope the key** to the caller and endpoint, so two clients cannot collide.
- **Store the result, not just the key.** Returning "already processed" without the original response forces the client to guess.
- **Expire keys** after a window longer than the longest retry policy.
- **Reject a reused key with a different payload.** That is a client bug, and silently returning the old result hides it.
- Generate the key at the start of the operation, not inside the retry loop, or each retry gets a new key and the protection disappears.

## Related

- [Supersession Over Deletion](/wiki/patterns/supersession-over-deletion.md): keeping history safe to replay.
