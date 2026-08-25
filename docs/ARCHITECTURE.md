# Namagiri Architecture

## The surgical-team model

The project uses a deliberate separation:

```text
Ryan O'Neill
    │
    │ authoritative design / clarification
    ▼
 Shiva
    │
    │ low-level runtime, linker, loader,
    │ modules, tracing, instrumentation
    ▼
 Target process / ELF

Namagiri surrounds Shiva:

    inspect → validate → plan → approve → invoke → collect → explain
```

Ryan is not an API that Namagiri calls. He is the human authority when the source leaves an architectural question unresolved.

## Layers

```text
CLI
 │
Core models / policy
 │
ELF inspection ───── Shiva discovery
 │                       │
 │                       └── capability model
 └──────────────┬───────────────┘
                ▼
           validation
                ▼
             planning
                ▼
            execution
                ▼
          experiment record
                ▼
             reporting
```

## Shiva adapter rule

The Shiva adapter may discover, inspect, validate, invoke, and collect. It must not become a duplicate implementation of Shiva internals.

## First implementation

The first vertical slice is intentionally small:

1. inspect an ELF target;
2. discover a Shiva installation/source identity;
3. discover modules;
4. produce a compatibility result;
5. create a plan.

Execution follows only after those pieces are trustworthy.
