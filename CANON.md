# CANON — Namagiri

This document defines what is allowed to become **canonical knowledge** in Namagiri.

## 1. Shiva is not ours to redefine

Namagiri is built around Shiva. It does not define what Shiva *should* do. It records what Shiva actually does according to authoritative evidence.

## 2. Source of truth

For Shiva behavior, the authority order is:

1. Ryan O'Neill's `elfmaster/shiva` source repository.
2. The exact pinned source snapshot under `reference/shiva-source/`.
3. Direct clarification from Ryan when source is ambiguous, incomplete, or intentionally undocumented.
4. Controlled observations against a named Shiva build.
5. Papers, presentations, blog posts, third-party explanations, and derivative projects are non-authoritative.

If an external explanation disagrees with Shiva source, the external explanation loses.

## 3. Provenance states

Every important Shiva claim should eventually carry one of these states:

- **SOURCE** — verified directly in Shiva source.
- **CONFIRMED** — explicitly confirmed by Ryan.
- **OBSERVED** — demonstrated experimentally against a named build.
- **INFERRED** — reasoned from evidence but not explicitly confirmed.
- **UNKNOWN** — not established.

`INFERRED` is never silently promoted to `SOURCE` or `CONFIRMED`.

## 4. Source versus behavior

Source and observation are separate records.

If source says X and an experiment observes Y, Namagiri records the discrepancy. It does not silently choose whichever answer is convenient.

## 5. The Namagiri boundary

Namagiri owns:

- target discovery;
- ELF inspection;
- capability discovery;
- module discovery;
- compatibility checking;
- configuration;
- execution planning;
- experiment management;
- result collection;
- reproducibility;
- explanations.

Shiva owns its own runtime, loader, linker, module execution, runtime mutation, tracing, breakpoints, hooks, and instrumentation mechanisms.

## 6. The decision rule

```text
Does Shiva already provide this?

YES → expose/orchestrate it.

NO → is it inspection, validation, planning, orchestration,
     experiment management, reproducibility, or explanation?

     YES → Namagiri may own it.

     NO → stop and review the boundary before implementing it.
```

## 7. Unknown is a valid answer

When evidence is insufficient, Namagiri reports `UNKNOWN`.

It does not guess in order to make the interface look complete.

## 8. Preservation

Original targets are preserved by default. Plans, targets, modules, Shiva builds, and experiment outputs receive identities and hashes wherever practical.

## 9. Ryan's work

Namagiri must never make Shiva look more mature, stable, or capable than the authoritative evidence supports. Known limitations belong in the user-facing compatibility model.

## 10. Restricted information

NDA-bound or otherwise restricted information is never assumed. If Ryan provides information with sharing restrictions, those restrictions control its treatment.

## 11. Scope discipline

Android, alternative architectures, Shiva modifications, and unrelated binary tooling remain out of scope until explicitly justified by Shiva's authoritative source and the project roadmap.
