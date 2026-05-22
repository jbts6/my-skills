# grill-rounds

Self-contained multi-round, multi-session grilling protocol for stress-testing a large design corpus (GDD, ADR set, `docs/`) against itself.

## What it does

Runs a disciplined, ledger-backed grill session that combines two layers:

1. **Interview semantics** — challenges your terminology against the existing glossary, sharpens fuzzy language, stress-tests boundaries with concrete scenarios, cross-references code, and updates `CONTEXT.md` inline as decisions crystallise.
2. **Multi-round persistence** — keeps a `docs/GRILL_BACKLOG.md` ledger so sessions can resume across `/clear` and context compactions. Every round commits with a structured prefix and backfills the hash into the ledger.

## When to use

Trigger `/grill-rounds` when:

- The design corpus is too large for a single sit-down (GDDs, multi-context monorepos, mature ADR sets).
- You want disciplined commit hygiene per decision, not a chat transcript.
- Decisions must survive across sessions and context compactions.

If you only need a single sit-down stress-test against existing docs, use the upstream [`grill-with-docs`](https://github.com/anthropics/claude-code) skill instead.

## Protocol summary

Every round follows six steps:

1. **Open** — read `docs/GRILL_BACKLOG.md`, confirm the next candidate.
2. **Ask one question at a time** with a recommendation + tradeoff.
3. **Apply interview semantics** while asking — challenge glossary, sharpen terms, stress-test scenarios, cross-ref code.
4. **Parse the answer** — `接受` / `修正` / `写` / `1:X;2:Y` (sub-question vote).
5. **Accept-then-write** — only after acceptance, batch-edit the affected docs.
6. **Close the round** — update `GRILL_BACKLOG.md` with `Hash: TBD`, commit with `docs: GDD grill round N - <theme>`, then backfill the hash in a second commit.

See [SKILL.md](./SKILL.md) for the full protocol.

## Files

- [SKILL.md](./SKILL.md) — the skill definition (loaded by Claude Code).
- [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md) — `CONTEXT.md` glossary format.
- [ADR-FORMAT.md](./ADR-FORMAT.md) — ADR format + three-condition test for when to write one.

## Installation

```bash
ln -s "$(pwd)/grill-rounds" ~/.claude/skills/grill-rounds
```

Invoke via `/grill-rounds` in Claude Code.
