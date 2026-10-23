# Patterns

Reusable approaches — how to *do* a recurring thing well. A pattern captures a repeatable technique or structure that has paid off across more than one project.

- [Idempotency Keys for Retried Writes](/wiki/patterns/idempotency-keys-for-retried-writes.md) — Attach a client-generated unique key to every non-idempotent write so a retry after a timeout cannot apply the same operation twice.
- [Supersession Over Deletion](/wiki/patterns/supersession-over-deletion.md) — When new evidence replaces an old claim in a durable store, mark the old claim superseded with a date and a pointer instead of deleting or silently overwriting it.
