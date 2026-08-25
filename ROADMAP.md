# Namagiri — ROADMAP

## Status

**Document status:** Initial architecture and implementation roadmap  
**Authority:** Ryan O'Neill's `elfmaster/shiva` repository and the supplied `shiva-main.zip` source snapshot  
**Namagiri role:** Orchestration, inspection, validation, planning, execution management, result collection, reproducibility, and explanation  
**Shiva role:** ELF runtime/linker/loader, module loading, relocation/linking, runtime instrumentation, tracing, hooks, and runtime mutation  
**Android:** Explicitly out of scope until Shiva's behavior and support are established  
**Policy:** Namagiri does not fork, modify, replace, or reimplement Shiva merely to make Namagiri easier.

---

# 1. Executive Summary

Namagiri is the surgical team around Shiva.

The target ELF is the patient.

Ryan O'Neill is the surgeon and the authoritative designer of the instrument.

Shiva is the scalpel: a specialized low-level programmable runtime linker/interpreter and instrumentation engine for x86-64 Linux PIE ELF targets.

Namagiri must therefore **not become another Shiva**.

Its job is to make Shiva understandable, inspectable, repeatable, and safely usable without requiring the operator to understand every internal detail of ELF relocation, runtime linking, breakpoint mechanics, process maps, auxiliary vectors, trampolines, PLT/GOT behavior, or Shiva's internal module ABI before beginning an experiment.

The core workflow is:

```text
TARGET
  |
  v
DISCOVER
  |
  v
INSPECT
  |
  v
CAPABILITIES
  |
  v
MODULE DISCOVERY
  |
  v
COMPATIBILITY
  |
  v
EXECUTION PLAN
  |
  v
HUMAN REVIEW / EXPLICIT APPROVAL
  |
  v
SHIVA
  |
  v
OBSERVE
  |
  v
COLLECT
  |
  v
REPRODUCE
  |
  v
EXPLAIN
```

Namagiri should make the boundary between **what Shiva knows**, **what Namagiri knows**, and **what neither system knows** explicit.

The most important architectural principle is:

> **Namagiri coordinates Shiva; it does not impersonate Shiva.**

---

# 2. Source-of-Truth Policy

## 2.1 Primary authority

The only authoritative technical sources for Shiva are:

1. Ryan O'Neill's `elfmaster/shiva` repository.
2. The exact Shiva source snapshot supplied for this project.
3. Direct clarification from Ryan O'Neill when the source is ambiguous, incomplete, unstable, or intentionally undocumented.

The Shiva paper, presentations, third-party writeups, blog posts, derivative repositories, and external explanations may be useful as orientation, but they **must not establish Namagiri's understanding of Shiva behavior**.

If an external description conflicts with the repository, the repository wins.

If the repository and observed behavior conflict, record the discrepancy.

If the repository is ambiguous, ask Ryan.

If Ryan says a behavior differs from what we inferred, Ryan wins.

## 2.2 Knowledge provenance

Namagiri should eventually distinguish at least these states:

```text
SOURCE
    Verified directly from Shiva source.

CONFIRMED
    Explicitly confirmed by Ryan.

OBSERVED
    Demonstrated experimentally against a particular Shiva build.

INFERRED
    Reasoned from source/observations but not explicitly confirmed.

UNKNOWN
    Not established.
```

These are not interchangeable.

For example:

```text
capability:
    name: PLTGOT hook

status:
    SOURCE

evidence:
    shiva_trace.h
    shiva_trace.c
    modules/examples/pltgot_hook.c

confidence:
    high
```

Whereas:

```text
capability:
    name: "works with arbitrary PIE binaries"

status:
    UNKNOWN

reason:
    Shiva source explicitly says its supported target class is
    x86-64 Linux ELF PIE, but this does not prove compatibility
    with every possible PIE configuration.
```

Namagiri must never convert an inference into a fact merely because doing so makes the UI cleaner.

---

# 3. What Shiva Actually Is to Namagiri

The supplied Shiva source identifies Shiva as:

- a programmable runtime linker / program interpreter;
- an x86-64 ELF system;
- currently intended for Linux;
- currently supporting x86-64 ELF PIE targets;
- capable of loading ELF microprograms/modules into a process;
- capable of linking those modules into the process address space;
- equipped with the ShivaTrace API;
- capable of in-process debugging and instrumentation;
- capable of function tracing and hooking through several breakpoint mechanisms;
- capable of userland execution/loading;
- capable of examining target mappings, auxiliary vectors, callsites, and branch information.

The repository contains the following major implementation areas:

```text
shiva.c
shiva_ulexec.c
shiva_module.c
shiva_trace.c
shiva_trace_thread.c
shiva_analyze.c
shiva_callsite.c
shiva_maps.c
shiva_auxv.c
shiva_target.c
shiva_signal.c
shiva_error.c
shiva_util.c

modules/
    shakti_runtime.c
    examples/
```

Namagiri should treat those as **implementation surfaces**, not as components to duplicate.

---

# 4. Namagiri's Boundary

## 4.1 Namagiri owns

### Target discovery

- resolve target path;
- canonicalize path;
- verify existence;
- identify ELF;
- identify architecture;
- identify ELF type;
- identify PIE status;
- identify interpreter;
- identify relevant program headers;
- identify sections;
- identify dynamic metadata;
- identify symbols where available;
- identify relocation information;
- identify security-relevant ELF properties;
- calculate and record target hash.

### Shiva discovery

- locate Shiva executable;
- locate the Shiva source/build identity;
- locate module directories;
- locate `shakti_runtime.o`;
- identify standalone versus interpreter build where possible;
- identify Shiva configuration;
- identify dependencies;
- identify expected target class.

### Module discovery

- enumerate available Shiva modules;
- parse module ELF metadata;
- identify module entry point convention;
- identify exported/imported symbols;
- identify relocation requirements;
- identify expected compiler model;
- identify required Shiva APIs;
- identify module purpose;
- identify whether the module is an example, runtime module, or user module;
- identify module compatibility constraints.

### Compatibility checking

Namagiri determines whether the proposed operation appears compatible with the discovered target, Shiva build, and module.

It does **not** silently repair incompatibility.

### Planning

Namagiri creates a deterministic execution plan.

### Execution management

Namagiri invokes Shiva according to the approved plan.

It should not reach into Shiva's memory or reproduce Shiva's internal algorithms simply because doing so would be convenient.

### Experiment management

Namagiri creates experiment identities, directories, manifests, logs, and result records.

### Reproducibility

Every run should be reconstructable from its recorded inputs.

### Explanation

Namagiri should translate low-level facts into operator-readable explanations.

---

# 5. Shiva Owns

Namagiri must defer to Shiva for:

- runtime behavior;
- ELF target loading;
- userland execution;
- custom interpreter behavior;
- module loading;
- module memory image creation;
- relocation application;
- module GOT creation;
- module PLT stubs;
- module symbol resolution;
- module execution;
- target runtime mutation;
- target memory writes;
- breakpoint installation;
- signal-based breakpoint handling;
- ptrace operations;
- register manipulation;
- tracing;
- call/jump/trampoline instrumentation;
- PLTGOT hooks;
- INT3/SIGILL/SIGSEGV mechanisms;
- runtime control transfer;
- Shiva's internal address calculations;
- Shiva's runtime data structures.

If Shiva provides an operation, Namagiri should expose it rather than reproduce it.

---

# 6. Decision Rule

Every proposed Namagiri feature must pass this test:

```text
Does Shiva already provide this?

    YES
      |
      +--> Namagiri exposes/orchestrates it.
      |
      v

    NO
      |
      +--> Is the missing capability orchestration,
          inspection, validation, planning, recording,
          reproducibility, or explanation?

              YES --> Namagiri may own it.

              NO --> Do not implement it until Ryan
                     confirms the boundary.
```

This rule prevents scope creep.

---

# 7. Initial Repository Findings

The supplied repository is not a polished high-level application. It is a low-level research/engineering codebase with explicit unfinished areas.

Important repository characteristics include:

- x86-64-specific register structures;
- x86-64-specific relocation handling;
- x86-64-specific breakpoint behavior;
- Linux-specific process interfaces;
- dependence on `libelfmaster`;
- bundled `udis86`;
- separate standalone and interpreter builds;
- musl-based interpreter construction;
- runtime module loading;
- examples that demonstrate several instrumentation mechanisms;
- hard-coded paths and assumptions that Namagiri should discover rather than assume.

The repository itself contains a TODO list identifying unfinished or partially finished work. Namagiri must treat those items as **known Shiva limitations**, not as invitations to fix Shiva.

Known areas called out by the repository include:

1. multiple module loading/dependencies;
2. ET_EXEC target support;
3. large-code-model `SHIVA_TRACE_BP_CALL` support;
4. complete INT3 breakpoint reinstallation;
5. analogous SIGILL/SIGSEGV breakpoint completion;
6. intermittent interpreter-mode stack-related segfault;
7. incomplete register get/set operations;
8. file mappings for the target and `ld-linux` as future work.

The repository also marks fcf-protection PLT support as completed.

Namagiri should surface these facts during compatibility assessment.

---

# 8. Phase 0 — Shiva Reconnaissance

**Goal:** Build a machine-readable model of Shiva before writing significant orchestration logic.

This phase is deliberately read-only.

## 8.1 Inventory Shiva source

Create a Shiva adapter manifest containing:

```text
shiva_path
shiva_hash
shiva_version_identity
source_repository
source_revision_if_available
build_mode
target_architecture
target_os
supported_elf_types
runtime_module_path
dependency_paths
```

Do not assume `/opt/shiva/modules/shakti_runtime.o` is universal.

The repository currently contains that path as a runtime assumption. Namagiri should discover the actual path on the local system and report when it differs.

## 8.2 Inventory Shiva APIs

Build a registry from `shiva.h` and `shiva_trace.h`.

At minimum, record:

### Context

```text
shiva_ctx
```

### Module interface

```text
shiva_module_loader
```

### Tracing

```text
shiva_trace
shiva_trace_register_handler
shiva_trace_set_breakpoint
shiva_trace_write
```

### Address/map inspection

```text
shiva_maps_build_list
shiva_maps_get_base
shiva_maps_validate_addr
shiva_maps_prot_by_addr
```

### Callsite analysis

```text
shiva_analyze_run
shiva_analyze_find_calls
shiva_callsite_iterator_init
shiva_callsite_iterator_next
```

### Auxiliary vector

```text
shiva_auxv_iterator_init
shiva_auxv_iterator_next
shiva_auxv_set_value
```

### Target manipulation

```text
shiva_target_dynamic_set
```

### Error handling

```text
shiva_error_set
shiva_error_msg
```

### Register/tracing structures

Record the exact x86-64 register representation and breakpoint types from the source.

Do not abstract away details prematurely.

---

# 9. Phase 0.5 — Establish a Shiva Capability Registry

The capability registry is the heart of Namagiri.

Each Shiva capability should have:

```yaml
id:
name:
category:
source_symbols:
source_files:
status:
architectures:
target_requirements:
module_requirements:
runtime_requirements:
known_limitations:
verification:
notes:
```

Initial categories:

```text
runtime
module-loading
relocation
target-analysis
callsite-analysis
memory-maps
auxv
tracing
breakpoints
hooks
registers
signals
PLTGOT
trampolines
userland-exec
interpreter-mode
standalone-mode
```

Initial breakpoint capability IDs:

```text
breakpoint.call
breakpoint.jmp
breakpoint.int3
breakpoint.segv
breakpoint.sigill
breakpoint.trampoline
breakpoint.pltgot
```

Do not label all of these simply "supported."

The source explicitly shows that some mechanisms exist but are unfinished or have known limitations.

Use states such as:

```text
implemented
implemented-with-limitations
unfinished
experimental
unknown
```

---

# 10. Phase 1 — Target Inspection

**Goal:** Make Namagiri useful before it executes anything.

This is the first real user-facing milestone.

Command concept:

```text
namagiri inspect ./target
```

Output should contain:

```text
TARGET
  path:
  canonical path:
  sha256:
  architecture:
  ELF class:
  ELF type:
  PIE:
  entry:
  interpreter:

PROGRAM HEADERS
  PT_LOAD:
  PT_DYNAMIC:
  PT_INTERP:
  PT_PHDR:
  PT_TLS:
  ...

DYNAMIC
  NEEDED:
  JMPREL:
  FLAGS:
  FLAGS_1:
  BIND_NOW:
  RELRO:

SECTIONS
  .text
  .plt
  .plt.sec
  .got
  .got.plt
  .rela.plt
  .rela.text
  .dynamic
  ...

SYMBOLS
  exported:
  local:
  stripped:

RELOCATIONS
  total:
  types:
  text relocations:
  PLT relocations:

SHIVA COMPATIBILITY
  preliminary:
  reasons:
```

Inspection must be read-only.

---

# 11. Target Identity

Every target gets an immutable identity based on at least:

```text
canonical path
file size
SHA-256
architecture
ELF type
```

The hash is the important identity.

A path is not an identity.

If `./foo` is replaced between inspection and execution, Namagiri must detect it.

Execution should either:

1. refuse because the target changed; or
2. explicitly create a new experiment against the new target.

Never silently execute a changed file under an old plan.

---

# 12. Phase 2 — ELF Capability Analysis

Namagiri should develop a normalized target model.

Example:

```yaml
elf:
  class: ELF64
  machine: EM_X86_64
  type: ET_DYN
  pie: true

interpreter:
  path: /lib64/ld-linux-x86-64.so.2

dynamic:
  bind_now: false
  relro: partial
  jmprel: present

sections:
  text:
    present: true
  rela_text:
    present: false
  plt:
    present: true
  plt_sec:
    present: false
  got_plt:
    present: true
```

This data model becomes the input to compatibility checks.

---

# 13. Phase 3 — Module Discovery

Namagiri should inspect the module tree without executing modules.

The supplied repository demonstrates several module categories.

Examples include:

```text
func_tracer
trampoline
pltgot_hook
int3_bp
sigill_bp
plt_cfi
sandbox
command_inject
crackme_bypass
ldso_fuzz
ssh-backdoor
```

These examples are valuable because they reveal the actual Shiva module ABI and API usage.

They must not automatically become "supported Namagiri modules."

Instead, Namagiri should classify them:

```text
example
runtime
experimental
user-supplied
```

---

# 14. Module ABI Model

The source shows that module entry behavior is selected by Shiva flags.

For runtime modules, Shiva looks for:

```text
shakti_main
```

For initialization modules, Shiva looks for:

```text
shakti_module_init
```

Namagiri should therefore detect and report module entrypoints.

Example:

```text
MODULE
  path: modules/examples/func_tracer.o
  format: ELF64 ET_REL
  entrypoint: shakti_main
  status: VALID
```

If neither expected entrypoint is found:

```text
MODULE
  status: INCOMPATIBLE

REASON
  No supported Shiva module entrypoint found.
```

---

# 15. Module Compatibility

Before execution, Namagiri should inspect:

- ELF class;
- machine;
- relocations;
- symbol table;
- undefined symbols;
- module entrypoint;
- section flags;
- writable sections;
- executable sections;
- compiler model where detectable;
- references to Shiva symbols;
- use of Shiva tracing APIs;
- use of architecture-specific registers;
- module-specific assumptions.

Do not claim that these checks prove runtime compatibility.

The result should be:

```text
STATIC COMPATIBILITY
    PASS

RUNTIME COMPATIBILITY
    NOT PROVEN

SHIVA COMPATIBILITY
    READY FOR CONTROLLED TEST
```

---

# 16. Phase 4 — Capability Matching

A module should declare or Namagiri should infer requirements.

Example:

```yaml
module: func_tracer

requires:
  target:
    architecture: x86_64
    pie: true

  shiva:
    tracing: true
    callsite_analysis: true
    breakpoint.call: true

  module:
    code_model: large
```

Namagiri compares this against the target and Shiva registry.

Result:

```text
MODULE: func_tracer

TARGET
  x86-64: PASS
  PIE: PASS

SHIVA
  callsite analysis: PASS
  CALL breakpoint: PRESENT
  large code model limitation: WARNING

DECISION
  CONDITIONAL
```

The word **CONDITIONAL** matters.

Do not turn warnings into passes.

---

# 17. Compatibility Result Model

Use a small fixed vocabulary:

```text
PASS
FAIL
WARNING
UNKNOWN
NOT_APPLICABLE
```

Overall decision:

```text
READY
CONDITIONAL
BLOCKED
UNKNOWN
```

Example:

```text
RESULT
  CONDITIONAL

WHY
  Target is x86-64 PIE.
  Shiva supports this target class.
  Selected module uses CALL instrumentation.
  Repository documents a large-code-model limitation for
  CALL breakpoints.

ACTION
  Controlled experiment permitted only if the limitation
  is explicitly accepted.
```

---

# 18. Phase 5 — Execution Planning

Namagiri should never jump directly from:

```text
namagiri run target module
```

to:

```text
exec shiva ...
```

There should be an intermediate plan.

Example:

```text
PLAN 000001

TARGET
  ./foo
  sha256: ...

SHIVA
  ./shiva
  identity: ...

MODULES
  01. func_tracer.o

MODE
  standalone

VALIDATION
  target architecture: PASS
  target ELF type: PASS
  PIE: PASS
  module ELF class: PASS
  module entrypoint: PASS
  callsite analysis requirement: PASS
  CALL breakpoint limitation: WARNING

POLICY
  original preserved: YES

OUTPUT
  ./results/foo-000001/

DECISION
  CONDITIONAL
```

The plan should be serializable.

---

# 19. Execution Plan as a First-Class Artifact

Store the plan as machine-readable data.

Suggested:

```text
plan.json
```

and a human-readable:

```text
PLAN.md
```

The JSON is authoritative for replay.

The Markdown is explanatory.

---

# 20. Phase 6 — Safe Execution

Namagiri should default to:

```text
original target preserved
original module preserved
results isolated
execution recorded
```

Never overwrite the original target merely because Shiva is capable of modifying process state.

The default execution directory:

```text
results/
  <target-name>/
    <experiment-id>/
```

Suggested contents:

```text
manifest.json
plan.json
target.json
shiva.json
modules.json
stdout.log
stderr.log
exit.json
warnings.json
artifacts/
trace/
```

---

# 21. Execution Boundary

Namagiri should treat Shiva as an execution boundary.

Conceptually:

```text
Namagiri
    |
    | validated invocation
    v
  SHIVA
    |
    | runtime operation
    v
 TARGET
```

Namagiri should not secretly inject additional runtime machinery.

If an operation requires a Shiva module, the plan should explicitly name it.

---

# 22. Phase 7 — Result Collection

After execution, Namagiri collects:

- exit status;
- termination signal;
- stdout;
- stderr;
- execution duration;
- target hash;
- Shiva identity;
- module identities;
- plan;
- warnings;
- errors;
- generated artifacts;
- traces where produced;
- core dump information where applicable;
- relevant runtime observations.

The result record should never imply success merely because Shiva returned control.

Example:

```text
EXECUTION
  started: ...
  ended: ...
  exit status: 0

SHIVA
  returned: success

MODULE
  loaded: success

EXPERIMENT
  completed: success
```

These are separate facts.

---

# 23. Phase 8 — Explanation Engine

Namagiri's explanation layer is where the surgical-team analogy becomes concrete.

Instead of:

```text
shiva_trace_set_breakpoint failed
```

produce:

```text
ACTION
  Install CALL breakpoint.

FAILURE
  Shiva rejected the breakpoint.

CONTEXT
  Target: ./foo
  Address: 0x...
  Module: func_tracer.o

KNOWN SHIVA CONSTRAINT
  CALL breakpoint implementation currently has
  a documented large-code-model limitation.

INTERPRETATION
  The operation was not attempted again automatically.

NEXT ACTION
  Consult the Shiva capability record or Ryan before
  changing the module or Shiva.
```

The explanation layer must never invent a fix.

---

# 24. Unknowns Are First-Class Results

If Namagiri cannot determine whether a target is compatible:

```text
COMPATIBILITY
  UNKNOWN
```

not:

```text
probably compatible
```

If Shiva behavior is unclear:

```text
SHIVA CAPABILITY
  UNKNOWN

SOURCE
  insufficient evidence

RECOMMENDED AUTHORITY
  Ryan O'Neill
```

This is a core feature, not an error condition.

---

# 25. Phase 9 — Reproducibility

Every experiment should be reproducible from:

```text
target hash
Shiva hash/identity
module hashes
configuration
execution mode
arguments
environment policy
plan
```

Record environment information that materially affects the result.

At minimum:

```text
kernel identity
architecture
libc identity
Shiva build identity
module build identity
```

Where possible also record:

```text
compiler identity
compiler flags
linker identity
relevant environment variables
```

Do not indiscriminately dump secrets.

Sensitive environment values must be redacted or excluded.

---

# 26. Experiment IDs

Use monotonically assigned or UUID-style experiment IDs.

Recommended human-friendly format:

```text
YYYYMMDD-HHMMSS-NNN
```

Example:

```text
20260825-121500-001
```

The ID is not the identity of the target.

The target remains identified by its content hash.

---

# 27. Shiva Version Drift

Shiva is unfinished and expected to change.

Namagiri must therefore never assume that a module capability remains identical forever.

At startup:

```text
SHIVA DISCOVERY
  binary: ...
  source: ...
  build identity: ...
```

At experiment time:

```text
PLAN SHIVA IDENTITY
  ...
```

At execution time:

```text
ACTUAL SHIVA IDENTITY
  ...
```

If they differ:

```text
PLAN INVALIDATED
```

unless the operator explicitly approves a revalidation.

---

# 28. Module Version Drift

The same policy applies to modules.

A module's source changing without its filename changing must invalidate the previous module identity.

Hash:

```text
module.o
```

and ideally record source/build metadata when available.

---

# 29. Phase 10 — Regression Harness

Namagiri should eventually include a controlled Shiva test laboratory.

Do not start with arbitrary real-world binaries.

Use small targets specifically designed to exercise one property at a time.

Initial target families:

```text
hello-pie
hello-pie-with-plt
hello-pie-now
hello-pie-full-relro
hello-pie-fcf
hello-pie-stripped
hello-pie-symbol-rich
hello-pie-with-text-relocs
multithreaded-pie
signal-heavy-pie
```

Each target should have a declared purpose.

---

# 30. Capability Test Matrix

Build a matrix:

| Capability | Target | Module | Expected | Observed | Status |
|---|---|---|---|---|---|
| module loading | basic PIE | runtime | load | ? | ? |
| callsite analysis | basic PIE | tracer | enumerate | ? | ? |
| CALL breakpoint | PIE | func tracer | trigger | ? | ? |
| trampoline | PIE | trampoline | hook | ? | ? |
| PLTGOT | PIE | pltgot | hook | ? | ? |
| INT3 | PIE | int3 | trigger/rearm | ? | ? |
| SIGILL | PIE | sigill | trigger/rearm | ? | ? |
| SIGSEGV | PIE | future test | trigger/rearm | ? | ? |
| AUXV | PIE | ldso fuzz | inspect/mutate | ? | ? |
| map discovery | PIE | runtime | enumerate | ? | ? |

The test harness must distinguish:

```text
Shiva feature exists
```

from:

```text
Shiva feature works under this exact condition
```

---

# 31. Phase 11 — Failure Taxonomy

Namagiri should classify failures.

Suggested classes:

```text
TARGET_NOT_FOUND
TARGET_NOT_ELF
TARGET_UNSUPPORTED
TARGET_CHANGED
SHIVA_NOT_FOUND
SHIVA_BUILD_UNKNOWN
SHIVA_DEPENDENCY_MISSING
MODULE_NOT_FOUND
MODULE_INVALID
MODULE_INCOMPATIBLE
MODULE_SYMBOL_FAILURE
VALIDATION_FAILED
PLAN_INVALID
EXECUTION_FAILED
SHIVA_RUNTIME_FAILURE
TARGET_RUNTIME_FAILURE
SIGNAL_TERMINATION
TIMEOUT
UNKNOWN_FAILURE
```

This lets Namagiri explain failures without pretending to know their root cause.

---

# 32. Phase 12 — Human Approval Gate

Certain operations should require explicit approval.

Especially:

```text
runtime mutation
memory writes
hooks
breakpoints
instrumentation
potentially destructive modules
```

The first implementation can simply require:

```text
DECISION: READY

Proceed? [y/N]
```

Later this becomes policy configuration.

Example:

```yaml
policy:
  preserve_original: true
  require_confirmation:
    - mutation
    - memory_write
    - hook
```

---

# 33. Module Trust Model

A module is executable code.

Namagiri must therefore distinguish:

```text
known module
unknown module
modified module
untrusted module
```

The module hash belongs in the experiment record.

A module should not become trusted merely because it exists inside a `modules/` directory.

---

# 34. Dangerous Examples

The repository includes examples whose names or behavior are clearly experimental/security-oriented.

Examples include:

```text
command_inject
crackme_bypass
ssh-backdoor
ldso_fuzz
sandbox
```

Namagiri should not silently present these as ordinary production modules.

Classify them visibly as:

```text
EXPERIMENTAL
```

or:

```text
SECURITY RESEARCH
```

according to what the source actually demonstrates.

The purpose here is provenance and operator awareness, not censorship of the Shiva repository.

---

# 35. Phase 13 — CLI Design

Initial CLI should be small.

Suggested commands:

```text
namagiri inspect TARGET
namagiri modules
namagiri module inspect MODULE
namagiri shiva
namagiri validate TARGET MODULE...
namagiri plan TARGET MODULE...
namagiri run PLAN
namagiri show EXPERIMENT
namagiri reproduce EXPERIMENT
namagiri diff EXPERIMENT_A EXPERIMENT_B
```

Do not build a giant CLI before the underlying data model stabilizes.

---

# 36. `inspect`

Example:

```text
namagiri inspect ./foo
```

Must never execute the target.

Primary output:

```text
TARGET
  compatible class: x86-64 PIE
  hash: ...
  interpreter: ...

SHIVA
  detected: yes
  compatible target class: yes

PRELIMINARY RESULT
  COMPATIBLE
```

---

# 37. `modules`

Example:

```text
namagiri modules
```

Output:

```text
MODULE                  TYPE          STATUS
shakti_runtime.o        runtime       supported
func_tracer.o           example       available
trampoline.o            example       available
pltgot_hook.o           example       available
int3_bp.o               example       experimental
...
```

---

# 38. `module inspect`

Example:

```text
namagiri module inspect func_tracer.o
```

Output:

```text
MODULE
  format: ELF64 ET_REL
  architecture: x86-64
  entrypoint: shakti_main
  undefined symbols: ...
  relocations: ...
  Shiva APIs referenced:
    shiva_trace
    shiva_trace_register_handler
    shiva_trace_set_breakpoint
    shiva_callsite_iterator_init
    shiva_callsite_iterator_next

REQUIREMENTS
  callsite analysis
  CALL breakpoint

WARNINGS
  CALL breakpoint has known large-code-model limitation.
```

---

# 39. `validate`

Example:

```text
namagiri validate ./foo func_tracer.o
```

Output should resemble:

```text
TARGET
  architecture: PASS
  ELF type: PASS
  PIE: PASS

MODULE
  architecture: PASS
  entrypoint: PASS

SHIVA
  callsite analysis: PASS
  CALL breakpoint: WARNING

RESULT
  CONDITIONAL
```

No execution occurs.

---

# 40. `plan`

The plan command should generate a durable artifact.

```text
namagiri plan ./foo func_tracer.o
```

Produces:

```text
results/foo/<experiment-id>/plan.json
results/foo/<experiment-id>/PLAN.md
```

The operator can inspect the plan before execution.

---

# 41. `run`

`run` accepts a plan, not an ambiguous collection of options.

```text
namagiri run plan.json
```

This gives us an important property:

> **Planning and execution are separate operations.**

That makes experiments auditable.

---

# 42. `reproduce`

Example:

```text
namagiri reproduce 20260825-121500-001
```

Behavior:

1. load experiment;
2. verify target hash;
3. verify module hashes;
4. verify Shiva identity;
5. compare environment;
6. report differences;
7. refuse to silently substitute changed components.

---

# 43. `diff`

Example:

```text
namagiri diff 001 002
```

Compare:

```text
target
Shiva
modules
configuration
environment
plan
exit state
artifacts
```

This becomes extremely useful when Shiva changes.

---

# 44. Architecture of Namagiri

Recommended logical layers:

```text
namagiri/
|
+-- cli/
|
+-- core/
|   +-- models/
|   +-- identity/
|   +-- policy/
|   +-- errors/
|
+-- elf/
|   +-- inspector/
|   +-- dynamic/
|   +-- symbols/
|   +-- relocations/
|   +-- security/
|
+-- shiva/
|   +-- discovery/
|   +-- source-model/
|   +-- capability/
|   +-- invocation/
|   +-- modules/
|
+-- validation/
|
+-- planning/
|
+-- execution/
|
+-- experiments/
|
+-- reporting/
|
+-- tests/
|
+-- fixtures/
```

The `shiva/` layer is an adapter.

It should not become a copy of Shiva.

---

# 45. Shiva Adapter

The Shiva adapter should have a narrow responsibility:

```text
discover
validate
invoke
collect
```

Not:

```text
reimplement
```

Possible interface:

```text
discover_shiva()
inspect_shiva()
discover_modules()
validate_module()
build_invocation()
execute()
collect_result()
```

---

# 46. ELF Adapter

Namagiri needs an ELF inspection layer.

Its purpose is not to become another linker.

It should answer questions such as:

```text
What is this ELF?
What architecture?
What type?
Is it PIE?
What interpreter?
What program headers?
What dynamic tags?
What relocations?
What symbols?
What sections?
What relevant properties?
```

It should not perform Shiva's runtime relocation logic.

---

# 47. Data Model

The central model should look approximately like:

```text
Target
  identity
  elf
  dynamic
  symbols
  relocations
  capabilities

Shiva
  identity
  build
  capabilities
  environment

Module
  identity
  elf
  entrypoint
  requirements
  capabilities

Plan
  target
  shiva
  modules
  configuration
  policy
  validation

Experiment
  plan
  actual execution
  results
  artifacts
  observations
```

---

# 48. Capability Model

Avoid boolean capability flags where possible.

Bad:

```text
call_breakpoint = true
```

Better:

```yaml
call_breakpoint:
  state: implemented_with_limitations
  architecture:
    - x86_64
  target:
    - PIE
  limitations:
    - large_code_model
  source:
    - shiva_trace.c
    - shiva.h
    - TODO
```

This prevents false certainty.

---

# 49. Configuration Model

Namagiri configuration should describe **Namagiri behavior**, not silently rewrite Shiva.

Example:

```yaml
shiva:
  path: /path/to/shiva
  runtime_module: /path/to/shakti_runtime.o

policy:
  preserve_original: true
  require_confirmation: true

results:
  directory: ./results
```

Do not create a giant Shiva configuration abstraction until Shiva itself exposes stable configuration semantics.

---

# 50. Hard-Coded Shiva Assumptions Namagiri Must Detect

The current source contains assumptions such as:

```text
/lib64/ld-linux-x86-64.so.2
/opt/shiva/modules/shakti_runtime.o
/proc/self/exe
```

and build-time paths including:

```text
/home/elfmaster/git/shiva/ldso/shiva
/opt/elfmaster/include/libelfmaster.h
/opt/elfmaster/lib/libelfmaster.a
```

Namagiri must not inherit these assumptions blindly.

Instead:

```text
DISCOVER
  actual path

COMPARE
  expected/source assumption

REPORT
  mismatch
```

---

# 51. Build Environment Discovery

Namagiri should detect whether Shiva's expected build environment exists.

Relevant dependencies visible in the repository include:

```text
gcc
musl-gcc
libelfmaster
udis86
musl development environment
```

Do not assume that Ubuntu package names or paths are universal.

The discovery layer should report:

```text
DEPENDENCY
  libelfmaster

STATUS
  FOUND

PATH
  ...

SOURCE
  Shiva build configuration
```

or:

```text
STATUS
  MISSING
```

---

# 52. Phase 14 — Build Verification

Before Namagiri claims that a Shiva installation is usable:

```text
discover source
discover compiler
discover dependencies
build Shiva if requested
verify output
inspect output ELF
record hash
run harmless test target
```

The first successful build should produce a Shiva installation manifest.

---

# 53. Build Identity

A Shiva identity should include:

```text
binary SHA-256
build mode
compiler identity
source identity
linked library identities
```

If source revision cannot be determined from the supplied snapshot, say so.

Never fabricate a version number.

---

# 54. Phase 15 — Interpreter Mode vs Standalone Mode

The source contains distinct execution paths.

Namagiri should treat these as separate modes.

```text
STANDALONE
  shiva invoked directly

INTERPRETER
  target names shiva as program interpreter
```

They are not equivalent.

The source specifically contains interpreter-mode logic involving:

- target mapping already performed by the kernel;
- map discovery;
- target base calculation;
- loading the runtime module;
- loading the real dynamic linker;
- creation of a new stack;
- copying the top of the old stack;
- transferring control to the dynamic linker.

Namagiri should report which mode is being used.

---

# 55. Interpreter-Mode Risk

The repository explicitly documents an intermittent interpreter-mode stack-copy bug.

Namagiri should therefore display:

```text
MODE
  interpreter

KNOWN SHIVA WARNING
  Repository documents an intermittent interpreter-mode
  stack-copy segfault.

POLICY
  do not classify interpreter mode as universally stable.
```

This is exactly the sort of information Namagiri exists to surface.

---

# 56. ET_EXEC

The repository's TODO explicitly lists ET_EXEC support as unfinished.

Therefore:

```text
ET_DYN / PIE
  supported target class

ET_EXEC
  BLOCKED unless Ryan confirms a newer implementation
```

Namagiri should not attempt to work around this by calculating addresses itself.

That would violate the boundary.

---

# 57. Large Code Model

The source explicitly documents a limitation around:

```text
SHIVA_TRACE_BP_CALL
```

and large-code-model addressing.

Namagiri should make this a capability condition.

Example:

```text
WARNING
  Selected module requires CALL breakpoint instrumentation.

SHIVA LIMITATION
  Repository documents a 2GB immediate-call range issue
  for the current implementation.

DECISION
  CONDITIONAL
```

Namagiri must not implement its own trampoline to "fix" Shiva unless Ryan explicitly establishes that as Namagiri-owned orchestration.

---

# 58. Breakpoint Model

Namagiri should expose breakpoint types from Shiva rather than inventing its own semantics:

```text
JMP
CALL
INT3
SEGV
SIGILL
TRAMPOLINE
PLTGOT
```

For each, record:

```text
source support
module examples
known TODO status
observed tests
limitations
```

---

# 59. INT3 / SIGILL / SIGSEGV

The repository explicitly describes incomplete reinstallation behavior for these mechanisms.

Namagiri must therefore distinguish:

```text
breakpoint mechanism exists
```

from:

```text
breakpoint lifecycle is complete
```

A validation result should be capable of saying:

```text
IMPLEMENTED
BUT INCOMPLETE
```

rather than PASS.

---

# 60. PLTGOT Hooks

The repository contains explicit PLTGOT support and an example module.

Namagiri should inspect the target for relevant dynamic-linking conditions before recommending this operation.

Important source-level facts include:

- the target's GOT/PLT state matters;
- strict linking / BIND_NOW can overwrite hooks;
- Shiva's source contains a strategy involving an alternate relocation table for hooked symbols.

Namagiri should report these as Shiva behavior and constraints, not reproduce them.

---

# 61. `.rela.text`

The Shiva executable warns when `.rela.text` exists because it may alter the effects of breakpoints/instrumentation.

Namagiri should therefore inspect for:

```text
.rela.text
```

and surface:

```text
WARNING
  Target contains .rela.text.

WHY
  Shiva source warns this may alter breakpoint/instrumentation effects.
```

This is an excellent example of what Namagiri should do.

---

# 62. FCF Protection / PLT Layout

The Shiva TODO states that fcf-protection PLT support is completed, with the handling ultimately depending on `libelfmaster`.

Namagiri should detect relevant PLT structures such as:

```text
.plt
.plt.sec
.plt.got
.got.plt
```

but should not declare support solely because those sections exist.

The capability result should depend on the Shiva version/source being used.

---

# 63. Auxiliary Vector

Shiva exposes an auxiliary-vector iterator and setter.

Namagiri should expose this as a capability:

```text
auxv.read
auxv.modify
```

but modifying auxv should be classified as a mutation operation requiring explicit approval.

---

# 64. Process Maps

Shiva maintains a mapping model.

Namagiri can use the capability registry to explain runtime results such as:

```text
Shiva mapping
target mapping
heap
stack
VDSO
misc
```

However, Namagiri should not duplicate Shiva's runtime map tracking unless it is independently useful for reporting.

---

# 65. Callsite Analysis

Shiva analyzes callsites and exposes an iterator.

This becomes a natural Namagiri validation requirement.

For a function-tracing module:

```text
module requirement:
  callsite enumeration

Shiva capability:
  available

target analysis:
  completed

decision:
  ready
```

Namagiri does not need to independently rediscover every callsite merely to prove that Shiva can.

---

# 66. Phase 16 — First End-to-End Vertical Slice

Do not implement the entire roadmap before testing the architecture.

The first complete slice should be:

```text
inspect target
    |
discover Shiva
    |
discover runtime module
    |
validate target
    |
generate plan
    |
show plan
    |
execute a harmless known-good experiment
    |
record result
```

No fancy UI.

No plugin framework.

No daemon.

No Android.

No automatic module generation.

---

# 67. First Demonstration Target

Use the repository's simplest test target.

The repository already contains test programs and Makefile targets.

Namagiri should discover the actual built artifacts rather than assuming their existence.

The first success criterion is not "do something impressive."

It is:

> **Namagiri can correctly explain what it is about to ask Shiva to do, execute it, and record exactly what happened.**

---

# 68. First Demonstration Module

Prefer a benign instrumentation module such as the repository's function-tracing example over mutation-oriented examples.

The purpose is to validate:

```text
module discovery
module ABI recognition
callsite analysis
breakpoint capability
execution
trace/result collection
```

Do not start with:

```text
command injection
crackme bypass
ssh backdoor
```

Those can become later controlled experiments if specifically desired.

---

# 69. Phase 17 — Experiment Database

Initially, filesystem-based JSON is enough.

Do not introduce a database until the model proves insufficient.

Directory model:

```text
results/
  target-name/
    experiment-id/
      manifest.json
      plan.json
      result.json
      logs/
      traces/
      artifacts/
```

Later, an index can be added:

```text
results/index.json
```

---

# 70. Manifest

Suggested manifest:

```yaml
experiment_id:
created_at:
target:
  path:
  hash:
shiva:
  path:
  hash:
  mode:
modules:
  - path:
    hash:
plan:
  path:
policy:
  preserve_original:
  confirmation_required:
result:
  status:
  exit_code:
  signal:
artifacts:
warnings:
errors:
```

---

# 71. Artifact Integrity

Every significant generated artifact should have a hash.

Example:

```text
artifact:
  path: trace/events.json
  sha256: ...
```

This makes experiment records tamper-evident and reproducible.

---

# 72. Logging

Namagiri should separate:

```text
operator log
execution stdout
execution stderr
Shiva stderr
Namagiri diagnostics
structured events
```

Do not merge everything into one log.

---

# 73. Structured Events

A structured event stream will make later automation much easier.

Example:

```json
{
  "event": "validation",
  "check": "pie",
  "result": "PASS"
}
```

Then:

```json
{
  "event": "execution",
  "phase": "shiva",
  "result": "STARTED"
}
```

This allows future UI layers without redesigning the engine.

---

# 74. Phase 18 — Explainability

Every validation check should contain:

```text
WHAT
WHY
EVIDENCE
DECISION
```

Example:

```text
CHECK
  target.pie

WHAT
  Target is ET_DYN.

WHY
  Current Shiva target support is documented for x86-64 ELF PIE.

EVIDENCE
  ELF header: ET_DYN
  dynamic flags: PIE

DECISION
  PASS
```

---

# 75. No Magical Fixes

If a check fails:

```text
FAIL
```

not:

```text
AUTO-FIX
```

unless the fix is explicitly Namagiri-owned and non-invasive.

Examples of acceptable orchestration:

```text
select correct module
select correct Shiva mode
select correct target
select output directory
```

Examples that require boundary review:

```text
rewrite ELF
inject custom trampoline
modify target relocations
patch Shiva
change Shiva source
```

---

# 76. Phase 19 — Shiva Question Queue

Namagiri should maintain a list of questions for Ryan when source analysis reaches ambiguity.

Each question should contain:

```text
question
source evidence
our interpretation
why it matters
experiment performed
exact ambiguity
```

Example:

```text
QUESTION
  Is module ordering intended to be externally controllable,
  or should future multi-module dependency handling be entirely
  driven by Shiva's DT_SHIVA_NEEDED mechanism?

SOURCE
  TODO item 1.

WHY IT MATTERS
  Determines whether Namagiri should expose explicit module ordering.
```

This is much better than asking vague questions.

---

# 77. Direct Ryan Verification Protocol

When a source question needs Ryan:

1. quote the relevant symbol/file/behavior internally;
2. state what the source appears to do;
3. state what is uncertain;
4. ask one precise question;
5. record Ryan's answer;
6. mark the capability `CONFIRMED`;
7. do not reinterpret the answer beyond its scope.

---

# 78. NDA Boundary

Namagiri must have a conceptual information boundary:

```text
PUBLIC SOURCE
      |
      v
Namagiri knowledge base

RYAN CONFIRMED PUBLIC/SHARED INFO
      |
      v
Namagiri knowledge base

NDA/RESTRICTED INFO
      |
      X
Not automatically recorded or redistributed.
```

If Ryan says something cannot be documented or shared, it remains outside the public project knowledge base.

---

# 79. Security Boundary

Namagiri itself should be conservative.

It is orchestrating a runtime instrumentation system capable of modifying live process state.

Therefore:

```text
inspect = safe/read-only
validate = safe/read-only
plan = safe/read-only
run = potentially mutating
```

The command boundary should make that distinction obvious.

---

# 80. Dry Run

Before `run`, Namagiri should support:

```text
namagiri plan ...
```

and ideally:

```text
namagiri run --dry-run plan.json
```

Dry-run should report exactly what would be invoked without executing Shiva.

---

# 81. Environment Capture

Capture only environment information needed for reproducibility.

Recommended classes:

```text
toolchain
loader
kernel
architecture
Shiva configuration
module configuration
```

Redact:

```text
passwords
tokens
private keys
credential-like variables
```

---

# 82. Timeouts

Every execution plan should have an execution timeout policy.

Default:

```text
unset until deliberately chosen
```

or a conservative project-defined default.

A timeout must be recorded as:

```text
TIMEOUT
```

not:

```text
FAIL
```

The underlying cause may still be unknown.

---

# 83. Crash Handling

If target terminates by signal:

```text
SIGNAL_TERMINATION
```

Record:

```text
signal
core dump presence
last structured event
Shiva stderr
target stderr
```

Do not automatically claim "Shiva crashed" simply because the target died.

---

# 84. Phase 20 — Differential Experiments

Namagiri becomes much more useful when experiments can be compared.

Example:

```text
same target
same module
same plan
different Shiva builds
```

Then:

```text
DIFFERENCE
  Shiva identity changed.

OBSERVATION
  experiment A: CALL hook succeeded
  experiment B: CALL hook failed

CONCLUSION
  behavior changed across Shiva builds.

ROOT CAUSE
  UNKNOWN
```

This is the correct scientific posture.

---

# 85. Shiva Change Detection

When the Shiva repository changes, Namagiri should eventually be able to regenerate its capability model.

The source itself remains the authority.

Potential workflow:

```text
new Shiva source
    |
source inventory
    |
API diff
    |
capability diff
    |
test affected capabilities
    |
update Namagiri compatibility database
```

---

# 86. API Drift Detection

Detect changes to:

```text
headers
enums
structs
function signatures
macros
breakpoint types
module flags
```

A change in `shiva.h` or `shiva_trace.h` should trigger a compatibility review.

---

# 87. Module ABI Drift

Likewise, changes affecting:

```text
shakti_main
shakti_module_init
shiva_ctx_t
shiva_trace_handler
shiva_trace_bp
```

should trigger module validation.

Namagiri should not assume binary compatibility across Shiva changes.

---

# 88. Phase 21 — Documentation Generated from the Source Model

Once the capability registry works, Namagiri can generate:

```text
SHIVA-CAPABILITIES.md
MODULES.md
TARGET-COMPATIBILITY.md
KNOWN-LIMITATIONS.md
```

These should be generated from structured records where possible.

The human-maintained source-of-truth policy remains separate.

---

# 89. Recommended Project Documentation

Namagiri should eventually contain:

```text
README.md
ROADMAP.md
ARCHITECTURE.md
SOURCE-OF-TRUTH.md
SHIVA-CAPABILITIES.md
MODULES.md
EXPERIMENTS.md
TESTING.md
SECURITY.md
```

But do not write all of them now.

ROADMAP comes first.

---

# 90. Phase 22 — Testing Strategy

Testing should be layered.

## Unit tests

For:

```text
ELF parsing
identity
hashing
data models
capability matching
plan serialization
manifest serialization
```

## Fixture tests

For known ELF files.

## Shiva integration tests

Against the exact Shiva build.

## Experiment tests

Run actual modules.

## Regression tests

Compare results across Shiva versions.

---

# 91. Golden Records

For stable experiments, store expected structured results.

Example:

```text
fixtures/
  hello-pie/
    inspect.json
    expected-validation.json
```

The purpose is to detect changes in Namagiri interpretation.

---

# 92. Never Golden-File Unstable Runtime Addresses

Do not expect:

```text
0x7f123456...
```

to remain constant.

Normalize address-sensitive output.

Record both:

```text
absolute address
relative/base-adjusted address
```

where meaningful, but don't make absolute ASLR addresses part of generic golden tests.

---

# 93. Address Model

Namagiri should represent addresses explicitly:

```text
file_offset
elf_vaddr
load_bias
runtime_vaddr
module_relative_vaddr
```

Never use one generic integer called "address" internally if it can mean multiple things.

This is especially important because Shiva's source distinguishes target base, Shiva base, module virtual addresses, runtime addresses, and userland-exec state.

---

# 94. Relocation Model

Namagiri should report relocations, not apply Shiva relocations.

Represent:

```text
type
symbol
section
offset
addend
```

For module validation, report whether Shiva's source recognizes the relocation type.

Do not reproduce `apply_relocation()` in Namagiri.

---

# 95. PLT/GOT Model

Inspection should distinguish:

```text
.plt
.plt.sec
.plt.got
.got
.got.plt
```

and dynamic relocation sources.

This allows Namagiri to explain why a PLTGOT operation may or may not be applicable without modifying the target itself.

---

# 96. Phase 23 — Module Metadata Convention

Eventually, Namagiri can support optional metadata alongside modules.

Example:

```text
func_tracer.o
func_tracer.namagiri.yaml
```

Possible metadata:

```yaml
name: func_tracer
kind: example

requires:
  shiva:
    - callsite_analysis
    - breakpoint.call

target:
  architecture:
    - x86_64
  elf_types:
    - ET_DYN
```

Important:

> Metadata describes Namagiri's expectations. It does not override Shiva.

---

# 97. Automatic Module Introspection

Where metadata does not exist, Namagiri can infer preliminary requirements from:

- undefined symbols;
- referenced Shiva APIs;
- entrypoint;
- relocation types;
- source comments if source is available;
- compiled ELF metadata.

Inference must remain `INFERRED`.

---

# 98. Phase 24 — Source-Aware Module Inspection

If module source is available, Namagiri can inspect it.

This is optional and later.

It can detect calls such as:

```text
shiva_trace
shiva_trace_register_handler
shiva_trace_set_breakpoint
shiva_trace_write
shiva_auxv_set_value
shiva_target_dynamic_set
```

This can produce an informative requirement report.

It must not be treated as a substitute for executing the module.

---

# 99. No Curriculum

Namagiri's purpose is not to teach ELF from scratch.

Its explanations should answer:

```text
What happened?
Why did Namagiri make this decision?
What does Shiva require?
What failed?
What evidence supports this?
```

Not:

```text
Here is a 40-page ELF tutorial.
```

Advanced details can be available on demand.

---

# 100. Phase 25 — Advanced UI

Only after the CLI/data model works should Namagiri consider:

```text
TUI
web UI
graph visualization
interactive experiment browser
module dependency graph
ELF visualizer
runtime map visualization
trace timeline
```

The UI must consume the same structured model.

---

# 101. Runtime Visualization

A future Namagiri view could show:

```text
TARGET
  |
  +-- ELF
  +-- dynamic linker
  +-- mappings
  +-- modules
       |
       +-- text
       +-- data
       +-- GOT
       +-- PLT
```

This would be genuinely useful, but it is not Phase 1.

---

# 102. Module Dependency Graph

When Shiva's multi-module support is established, Namagiri can visualize:

```text
runtime
   |
   +-- module A
   |      |
   |      +-- dependency X
   |
   +-- module B
```

Do not implement dependency semantics before Shiva's actual behavior is confirmed.

---

# 103. Phase 26 — Future Multi-Module Support

The repository TODO identifies DT_SHIVA_NEEDED as a future direction.

Namagiri should reserve a model for:

```text
module dependencies
module ordering
module search paths
module identity
dependency resolution
```

But keep it disabled until Shiva's source and/or Ryan confirms the semantics.

---

# 104. Android Boundary

Android remains out of scope.

Do not add:

```text
APK inspection
ART support
Bionic assumptions
Android linker support
ARM support
AArch64 support
```

to the first architecture.

When Android is eventually considered, it should begin with:

```text
Does Ryan's Shiva support this?
What exact Shiva source says so?
What exact target class is supported?
What is different about the loader/runtime?
```

Only then should Namagiri acquire Android-specific orchestration.

---

# 105. What Namagiri Must Never Become

Namagiri must not become:

```text
a Shiva fork
a replacement loader
a second relocation engine
a second debugger
a generic ELF patcher
a stealth injector
a collection of unrelated binary tools
a module compiler framework before the ABI is understood
an Android compatibility layer by assumption
```

The project fails architecturally if it becomes any of these.

---

# 106. Phase 27 — Minimal Viable Namagiri

The first useful release should do exactly this:

```text
1. Find Shiva.
2. Inspect Shiva.
3. Find modules.
4. Inspect target.
5. Validate target/module compatibility.
6. Produce a plan.
7. Ask for confirmation.
8. Execute Shiva.
9. Capture results.
10. Write reproducible experiment record.
11. Explain success/failure.
```

That is enough.

---

# 107. Definition of MVP Success

Namagiri MVP is successful if an operator can run:

```text
namagiri inspect ./target
```

and understand the target.

Then:

```text
namagiri validate ./target ./module.o
```

and understand whether the proposed operation is appropriate.

Then:

```text
namagiri plan ./target ./module.o
```

and understand exactly what will happen.

Then:

```text
namagiri run plan.json
```

and receive a durable experiment record.

---

# 108. Build Order

## Phase 0 — Reconnaissance

- [ ] Freeze source-of-truth policy.
- [ ] Inventory supplied Shiva snapshot.
- [ ] Inventory Shiva source files.
- [ ] Inventory exported APIs.
- [ ] Inventory module flags.
- [ ] Inventory breakpoint types.
- [ ] Inventory known TODOs.
- [ ] Identify hard-coded paths.
- [ ] Identify build dependencies.
- [ ] Create initial capability registry.

## Phase 1 — Inspection

- [ ] Target identity.
- [ ] ELF header inspection.
- [ ] Program-header inspection.
- [ ] Section inspection.
- [ ] Dynamic metadata inspection.
- [ ] Symbol inspection.
- [ ] Relocation inspection.
- [ ] PIE detection.
- [ ] Interpreter detection.
- [ ] `.rela.text` warning.
- [ ] PLT/GOT inventory.

## Phase 2 — Module Model

- [ ] Module discovery.
- [ ] Module hashing.
- [ ] Module ELF inspection.
- [ ] Entrypoint detection.
- [ ] Shiva API reference detection.
- [ ] Relocation inventory.
- [ ] Module requirement model.

## Phase 3 — Validation

- [ ] Target capability model.
- [ ] Shiva capability model.
- [ ] Module capability model.
- [ ] Matching engine.
- [ ] PASS/FAIL/WARNING/UNKNOWN.
- [ ] READY/CONDITIONAL/BLOCKED/UNKNOWN.

## Phase 4 — Planning

- [ ] Plan schema.
- [ ] Plan serialization.
- [ ] Human-readable PLAN.md.
- [ ] Policy model.
- [ ] Approval gate.
- [ ] Target immutability check.

## Phase 5 — Execution

- [ ] Shiva invocation adapter.
- [ ] stdout/stderr capture.
- [ ] exit-state capture.
- [ ] timeout handling.
- [ ] signal handling.
- [ ] artifact collection.

## Phase 6 — Experiments

- [ ] Experiment IDs.
- [ ] Manifest.
- [ ] Artifact hashes.
- [ ] Environment capture.
- [ ] Reproduction.
- [ ] Experiment diff.

## Phase 7 — Regression

- [ ] Fixture targets.
- [ ] Capability tests.
- [ ] Known-good modules.
- [ ] Known-failure cases.
- [ ] Shiva version comparison.

## Phase 8 — Advanced

- [ ] Multi-module model.
- [ ] Runtime observation.
- [ ] TUI/web UI.
- [ ] Trace visualization.
- [ ] Dependency graph.

## Phase 9 — Android Review

- [ ] Only after explicit Shiva capability confirmation.
- [ ] Reassess architecture.
- [ ] Reassess ELF/loader assumptions.
- [ ] Reassess Namagiri boundary.

---

# 109. Immediate Work — The Next Pass

Do **not** start coding the full system.

The next implementation pass should produce only these artifacts:

```text
namagiri/
  README.md
  ROADMAP.md
  SOURCE-OF-TRUTH.md

  src/
    models/
    elf/
    shiva/
    validation/

  tests/
```

The first executable capability should be:

```text
namagiri inspect TARGET
```

Then:

```text
namagiri shiva
```

Then:

```text
namagiri modules
```

Only after those work:

```text
namagiri validate TARGET MODULE
```

---

# 110. Immediate Research Questions

Before implementation expands, resolve these from Shiva source and, where necessary, Ryan:

1. What is the intended stable module ABI?
2. Is `shakti_main` the long-term module entry convention?
3. What is the intended status of `SHIVA_MODULE_F_INIT`?
4. Is module ordering currently meaningful or incidental?
5. What is the intended future DT_SHIVA_NEEDED model?
6. Which Shiva build mode should Namagiri treat as canonical?
7. What is the supported way to locate `shakti_runtime.o`?
8. Which parts of interpreter mode are considered stable?
9. What is the intended status of the incomplete register API?
10. Which breakpoint types are production-ready versus research examples?
11. What target constraints beyond x86-64 PIE are important?
12. Which relocation types are intentionally supported for modules?
13. Which target dynamic tags are safe for Namagiri to inspect versus modify?
14. What should constitute a valid module?
15. Is module source metadata expected to exist?
16. What is the intended relationship between standalone Shiva and interpreter Shiva going forward?

Questions should be asked only when the source cannot answer them.

---

# 111. Source Evidence Ledger

Maintain a ledger such as:

```text
Evidence ID:
SHIVA-SRC-001

File:
README.md

Claim:
Current target class is x86-64 ELF PIE.

Status:
SOURCE
```

Another:

```text
Evidence ID:
SHIVA-TODO-001

File:
TODO

Claim:
ET_EXEC handling is unfinished.

Status:
SOURCE
```

Another:

```text
Evidence ID:
SHIVA-TRACE-001

Files:
shiva_trace.h
shiva_trace.c

Claim:
Shiva exposes multiple breakpoint mechanisms.

Status:
SOURCE
```

This ledger prevents architectural drift.

---

# 112. Source Snapshot Discipline

The supplied `shiva-main.zip` should be treated as a pinned research snapshot.

Record:

```text
archive filename
archive SHA-256
file count
inspection date
repository URL
branch if known
```

If the live repository changes, do not silently mix live-source facts into the snapshot model.

Create a new source identity.

---

# 113. Two-Layer Truth Model

Namagiri should maintain two related but distinct concepts:

```text
SHIVA SOURCE MODEL
    What the source says.

SHIVA OBSERVATION MODEL
    What experiments demonstrate.
```

Then:

```text
SOURCE says X
OBSERVATION says Y
```

is not automatically resolved.

It becomes a discrepancy requiring investigation.

---

# 114. Why This Matters

This is especially important because Shiva is unfinished.

If Namagiri assumes:

```text
source == stable behavior
```

it will eventually lie.

If Namagiri assumes:

```text
observed once == guaranteed behavior
```

it will also lie.

The correct model is:

```text
SOURCE
  +
OBSERVATION
  +
CONFIRMATION
  =
KNOWLEDGE WITH PROVENANCE
```

---

# 115. Architectural North Star

The ultimate Namagiri workflow should feel like this:

```text
"Here is my ELF."

        ↓

"Let me examine it."

        ↓

"Here is what Shiva can see/do with it."

        ↓

"Here is the module I want to use."

        ↓

"Here is what that module requires."

        ↓

"Here is where Shiva's known limitations matter."

        ↓

"Here is the exact plan."

        ↓

"Here is what will happen if you approve it."

        ↓

"Shiva performed the operation."

        ↓

"Here is exactly what happened."

        ↓

"Here is what we know."

        ↓

"Here is what we do not know."

        ↓

"Here is how to reproduce it."
```

That is Namagiri.

---

# 116. Final Non-Negotiables

1. **Ryan's Shiva source is authoritative.**
2. **Ryan is the human authority when source interpretation is ambiguous.**
3. **Namagiri does not fork Shiva.**
4. **Namagiri does not silently modify Shiva.**
5. **Namagiri does not reimplement Shiva's runtime mechanisms.**
6. **Namagiri exposes Shiva capabilities rather than duplicating them.**
7. **Namagiri owns inspection and orchestration.**
8. **Namagiri validates before execution.**
9. **Namagiri preserves original targets by default.**
10. **Every experiment receives an immutable identity.**
11. **Every target and module is hashed.**
12. **Every Shiva build used by an experiment is identified.**
13. **Unknown means UNKNOWN.**
14. **Inference is never silently promoted to fact.**
15. **Source and observation are kept separate.**
16. **Failures are explained without invented causes.**
17. **Known Shiva limitations are surfaced before execution.**
18. **Namagiri adapts when Shiva changes.**
19. **NDA/restricted information is never assumed or exposed.**
20. **Android remains out of scope until Shiva support is established.**
21. **Namagiri must remain useful even when Shiva is incomplete.**
22. **The operator should always be able to understand the planned procedure before Shiva performs it.**

---

# 117. One-Line Definition

> **Namagiri is the orchestration and inspection layer that examines ELF targets, discovers Shiva's actual capabilities, validates module compatibility, builds reproducible execution plans, coordinates Shiva, records experiments, and explains the results—without becoming Shiva.**
