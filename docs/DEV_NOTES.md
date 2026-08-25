# Developer Notes

## Read this before coding

Namagiri is being built as a professional companion project to Shiva. The quality of the surrounding tooling can reflect on Ryan O'Neill's work. Clean boundaries, accurate claims, provenance, and reproducibility therefore matter more than speed.

The developer is intentionally green on parts of ELF/runtime instrumentation. The project lead is responsible for choosing the implementation order and for deciding when live tests should be run on Bucky.

## Rules of engagement

- Do not rush past an architectural gate.
- Do not patch Shiva to make Namagiri easier.
- Do not infer undocumented behavior and present it as fact.
- Do not use the paper as a substitute for source inspection.
- Do not turn a TODO in Shiva into a Namagiri workaround without reviewing the boundary.
- Keep generated artifacts out of source directories.
- Keep experiments under `experiments/` or `results/`.
- Keep source snapshots under `reference/`.
- Keep documentation under `docs/` unless it is a project-level contract such as `CANON.md` or `CONTRIBUTING.md`.

## Interruptions and new ideas

New questions are welcome. They do not automatically change the build order.

A useful question may be recorded in `TODO.md`, `docs/DEV_NOTES.md`, or an appropriate issue/record and then deferred until the roadmap reaches it.

The default response to a tempting shortcut is:

```text
record it → classify it → defer it if necessary → continue the canonical build order
```

## Bucky

Bucky is the live development/test host. Do not run unapproved experimental Shiva operations against arbitrary binaries there.

When a milestone is ready for live testing, the project lead should provide:

1. exact command;
2. expected output;
3. what would constitute failure;
4. what files are expected to change;
5. cleanup procedure;
6. rollback procedure, if applicable.

## Source snapshot

`reference/shiva-source/` is a frozen reference copy. Never edit it to make a build pass. If a source change is required, that belongs to Shiva and must be handled as a separate, explicitly authorized activity.
