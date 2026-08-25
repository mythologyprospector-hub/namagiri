# Security and Safety

Shiva is a runtime instrumentation system capable of modifying process state. Namagiri therefore treats execution as materially different from inspection.

## Defaults

- inspection is read-only;
- validation is read-only;
- planning is read-only;
- original targets are preserved;
- execution requires an explicit plan;
- mutation-oriented operations should have an approval gate.

## Module trust

A module is executable code. Presence in a directory does not establish trust.

Record module hashes and provenance.

## Experimental examples

The Shiva snapshot contains examples with security-research and mutation-oriented behavior. They should not be surfaced as ordinary safe modules without clear classification.

## Sensitive data

Do not capture credentials, tokens, private keys, or unrelated secrets into experiment records. Environment capture must be selective and redacted.

## Shiva boundary

Do not work around a Shiva limitation by silently patching or replacing Shiva behavior. If a change to Shiva appears necessary, document it and stop for source/Ryan review.
