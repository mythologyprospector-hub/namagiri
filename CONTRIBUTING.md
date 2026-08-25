# Contributing to Namagiri

## Before making a change

Read:

1. `CANON.md`
2. `ROADMAP.md`
3. `docs/ARCHITECTURE.md`
4. `docs/DEV_NOTES.md`
5. `docs/TESTING.md`

## The standard

A change is not complete merely because it works on one machine. It should be:

- understandable;
- source-grounded;
- narrowly scoped;
- testable;
- reproducible;
- documented when it changes behavior or architecture.

## Shiva changes

Do not modify Shiva inside the Namagiri repository. The pinned snapshot under `reference/shiva-source/` is evidence, not a working subtree.

If Namagiri appears to require a Shiva change, stop and document the requirement. The correct next step is source review and, if necessary, a question for Ryan.

## Code organization

- `src/` — implementation.
- `tests/` — tests and controlled fixtures.
- `tools/` — developer utilities.
- `config/` — project configuration/templates.
- `experiments/` — experiment definitions or metadata.
- `results/` — generated experiment results; do not commit large generated output unless explicitly required.
- `docs/` — supporting documentation.
- `reference/` — immutable upstream/source material.

## Commits

Prefer small, coherent commits with one reason for existing.

Good:

```text
Add ELF target identity model
```

Less useful:

```text
Namagiri stuff
```

## Documentation changes

If a change alters a canonical rule, update `CANON.md`.

If it changes implementation direction, update `ROADMAP.md` or `docs/ARCHITECTURE.md`.

If it records a temporary engineering observation, use `docs/DEV_NOTES.md`.

## Tests

Do not add a test that depends on undefined Shiva behavior without marking it as such.

See `docs/TESTING.md` for the test ladder.
