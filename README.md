# Namagiri

**Namagiri is the orchestration and inspection layer for Shiva.**

Namagiri is not a fork, replacement, or reimplementation of Shiva. Shiva remains the low-level engine. Namagiri examines ELF targets, discovers Shiva capabilities, validates proposed operations, builds reproducible execution plans, coordinates execution, collects results, and explains what happened.

> **Namagiri orchestrates Shiva; it does not impersonate Shiva.**

## Project status

This repository is intentionally a **skeleton**. It is a foundation for building the project in the correct order, not a claim that the architecture is already implemented.

The first implementation milestone is read-only inspection and source-grounded capability discovery. Execution comes later.

## Authority

The supplied Shiva source snapshot in `reference/shiva-source/` is preserved **unchanged** and is the primary technical reference for this project.

The project follows this authority order:

1. Ryan O'Neill's Shiva source repository.
2. The pinned Shiva source snapshot stored under `reference/shiva-source/`.
3. Direct clarification from Ryan O'Neill when the source is ambiguous or incomplete.
4. Controlled observations made against a specific Shiva build.
5. Everything else is orientation only and cannot establish Shiva behavior.

See [`docs/CANON.md`](docs/CANON.md) and [`docs/SOURCE_OF_TRUTH.md`](docs/SOURCE_OF_TRUTH.md).

## Repository layout

```text
.
├── README.md
├── ROADMAP.md
├── CANON.md
├── CONTRIBUTING.md
├── TODO.md
├── docs/
│   ├── ARCHITECTURE.md
│   ├── DEV_NOTES.md
│   ├── SECURITY.md
│   ├── SOURCE_OF_TRUTH.md
│   └── TESTING.md
├── src/
│   └── namagiri/
├── tests/
│   ├── fixtures/
│   └── integration/
├── tools/
├── config/
├── experiments/
├── results/
└── reference/
    └── shiva-source/
```

## Development order

Do not skip ahead because a later feature is exciting.

```text
Shiva reconnaissance
        ↓
ELF inspection
        ↓
Shiva discovery
        ↓
module discovery
        ↓
capability model
        ↓
validation
        ↓
execution planning
        ↓
controlled execution
        ↓
experiment records
        ↓
reproduction / regression
```

The project lead will decide when live testing is appropriate. Until then, keep changes read-only and source-grounded.

## Testing host

The user's personal development machine is called **Bucky** by the user. No live Shiva/Namagiri execution should be performed on Bucky merely because a feature has been written. A test is run only after the project reaches the appropriate milestone and a specific test procedure has been prepared.

## License

Namagiri is licensed under the MIT License.

Copyright (c) 2026 James Earl Stambaugh III

The Shiva snapshot retains the files and licensing information supplied with that source. Do not imply that Shiva and Namagiri have the same license.
