# Testing

## Principle

Testing follows the architecture. We do not begin by running invasive Shiva modules against arbitrary binaries.

## Test ladder

### Level 0 — Static project checks

- documentation structure;
- source snapshot integrity;
- Python/C/etc. syntax as applicable;
- serialization tests;
- unit tests.

### Level 1 — ELF inspection

Read-only inspection of known ELF fixtures.

No Shiva execution.

### Level 2 — Shiva discovery

Locate and identify the Shiva build without executing target instrumentation.

### Level 3 — Capability validation

Compare a target and module against the source-derived capability model.

Still no runtime mutation.

### Level 4 — Plan validation

Generate a complete execution plan and inspect it manually.

### Level 5 — Controlled Shiva execution

Only after the preceding levels pass. Use a deliberately chosen benign fixture and a known module.

### Level 6 — Regression

Repeat controlled experiments against pinned Shiva identities and compare structured results.

## Bucky procedure

Live testing on Bucky is gated by the project lead.

Before a live test, the lead must provide the exact command and expected outcome. Do not improvise a substitute command because the planned one fails.

## Test records

An experiment should record:

- target hash;
- Shiva identity;
- module hash(es);
- plan;
- configuration;
- environment information necessary for reproduction;
- stdout/stderr;
- exit status or signal;
- artifacts;
- warnings/errors.

## Golden tests

Do not golden-file ASLR-dependent absolute addresses. Normalize address-sensitive values where necessary.

## Unknown behavior

If a test reveals behavior that contradicts the source model, record the discrepancy rather than rewriting the model immediately.
