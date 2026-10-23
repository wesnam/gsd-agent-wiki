# Harness independence

This bundle is usable with no agent harness installed. [ARCHITECTURE.md](/docs/ARCHITECTURE.md) shows where GSD fits.

| Need | Requires GSD? |
|---|---|
| Read a page, browse `index.md`, search with `rg` | No |
| Curate a candidate (`/kb-curate`) | No |
| Lint the bundle (`/kb-lint`) | No |
| Capture from a GSD phase (`/kb-capture <phase>`) | Only to produce the `LEARNINGS.md` it reads, via GSD's learnings-extraction command |
| Capture without GSD | No: write a candidate by hand in `inbox/manual-notes/` using [`templates/candidate-template.md`](/templates/candidate-template.md) |

`/kb-capture` stops with a remediation message when its input is missing. It never generates the input itself and never requires GSD to be installed.

To check this yourself, search the skills and bundle for runtime references to GSD:

```bash
grep -rniE 'gsd|\.planning|extract-learnings' wiki/ inbox/ templates/ schemas/ index.md log.md CLAUDE.md README.md .claude/skills/
```

Every match should be prose naming GSD as an input source, never a file or command the bundle needs in order to be read or curated.
