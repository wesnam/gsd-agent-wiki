---
type: lesson
title: A Passing Suite Can Test the Wrong Thing
description: Tests that define their own fixtures and constants can pass while the real configuration is wrong; verify at least once through the real path.
generated: { by: claude/sonnet-5.5, at: 2026-10-22T00:00:00Z }
status: stable
confidence: probable
related:
  - /wiki/patterns/idempotency-keys-for-retried-writes.md
---

# A Passing Suite Can Test the Wrong Thing

A green test run means the code agrees with the test's assumptions, not with reality.

## The failure

A service reads a setting from configuration. The tests build their own value inline and pass it directly to the code under test. The suite is green, yet the shipped configuration holds a typo, and production fails on the first request. No amount of extra unit tests catches this, because none of them touch the real configuration.

## Why it happens

- Fixtures pin their own literals, so the value under test is correct by construction.
- Mocks replace the one component that could have rejected the bad value.
- Coverage numbers count lines executed, not whether the executed line used real inputs.

## What to do

- For every externally supplied value (config key, identifier, endpoint, schema), run **one check through the real loading path**, even if it is a short script.
- Keep that check small and fast so it actually gets run.
- When a bug escapes a green suite, ask which assumption the suite shared with the code, and add the check that would have broken it.

## Related

- [Idempotency Keys for Retried Writes](/wiki/patterns/idempotency-keys-for-retried-writes.md): another case where the unhappy path needs its own test.
