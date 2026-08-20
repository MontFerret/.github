# AGENTS.md

This file defines the engineering quality standards for coding agents working in this repository.

Repository-specific architecture, package ownership, commands, toolchains, generated-code rules, compatibility surfaces, and domain invariants should be documented separately when needed.

The rules here define the default quality bar for implementation work.

A task is not complete merely because the implementation compiles or its tests pass. Code must also be correct, appropriately designed, comprehensible, maintainable, well-tested, scoped to the requested work, and deliberately reviewed before completion.

# Engineering principles

* Preserve correctness first.
* Preserve existing observable behavior unless the task explicitly requires changing it.
* Identify behavioral ownership before changing implementation.
* Preserve architectural boundaries and lifecycle invariants.
* Prefer the smallest coherent change that fully solves the task.
* Prefer straightforward, idiomatic Go over clever implementations.
* Keep behavior, state ownership, dependencies, cancellation, cleanup, and resource lifetimes obvious.
* Avoid abstractions, indirection, and generalization without a concrete need.
* Do not optimize by intuition alone. Measure performance-sensitive work.
* Reuse existing patterns only after verifying that they are appropriate for the same semantics, ownership, and lifecycle.
* Existing technical debt is not precedent.
* Leave already-correct code alone.
* Do not treat the first working implementation as final.
* A task is complete only after implementation, validation, self-review, necessary corrections, final validation, and complete diff inspection.

# Ownership and design

Before making a non-trivial change, identify:

1. the subsystem, package, or type that owns the requested behavior;
2. the observable contract being preserved or changed;
3. the invariants involved;
4. the lifecycle and resource ownership involved, where applicable;
5. the compatibility surface involved, where applicable;
6. whether the change is concurrency-sensitive;
7. whether the change is performance-significant.

Begin with the code that owns the behavior.

Do not move behavior into a caller, adapter, command, transport, or presentation layer merely because that call site is convenient.

Keep domain behavior separate from transport, serialization, presentation, and protocol translation.

Adapters should validate and translate boundary values, delegate behavior to the owning implementation, and translate results back. They should not become alternate owners of domain semantics.

Avoid duplicated semantics. When one subsystem owns a rule, consumers should use that rule rather than independently reproducing it.

Keep reusable behavior at the narrowest ownership level that preserves clear responsibility and testability.

Do not expose implementation details across boundaries merely to avoid making a proper change in the owning layer.

# Abstraction discipline

Prefer concrete types and direct implementations until there is a real need for abstraction.

Introduce an interface when there is:

* an actual substitution boundary;
* more than one meaningful implementation;
* a focused consumer-side contract;
* a concrete test seam that materially improves the design.

Interfaces are usually most useful at the point of consumption.

Do not introduce interfaces, wrappers, managers, factories, helpers, generic types, or abstraction layers merely:

* for aesthetic symmetry;
* to reduce a few repeated lines;
* because another codebase uses the pattern;
* because the pattern is fashionable;
* to prepare for hypothetical future requirements;
* solely to make mocking convenient;
* to make files shorter.

Similar-looking code is not sufficient reason to share an implementation.

Extract shared behavior when the duplicated code represents the same concept with the same semantics, ownership, and lifecycle.

Do not force domain-specific concepts into a generic abstraction merely because their implementations have structural similarities.

Prefer deletion and simplification over another abstraction layer.

An abstraction must make this repository easier to reason about, not merely make the code look more architecturally sophisticated.

# Semantic types

Introduce a named type when it can own meaningful:

* semantics;
* invariants;
* behavior;
* validation;
* conversion;
* lifecycle;
* API safety.

Do not introduce a wrapper merely to give a primitive another name.

Once an established semantic type exists, APIs should normally use that type rather than repeatedly bypassing it with its underlying primitive.

Do not leave a meaningful domain type disconnected from its intrinsic behavior while free functions continue operating on primitive representations.

Unless zero has a natural and safe domain meaning, reserve zero as the unspecified or invalid value for enum-like types and begin meaningful values after it.

Keep sibling enum-like APIs consistent. Document intentional meaningful-zero exceptions.

# Function and method ownership

These rules are mandatory unless the task explicitly requires otherwise.

Prefer a method when behavior:

* belongs intrinsically to a semantic type;
* depends on that type's invariants;
* is a natural query or transformation of the value;
* manages resources owned by the value;
* operates primarily on state owned by the receiver.

Prefer a package-level function when behavior:

* constructs a value;
* combines unrelated values;
* performs genuinely package-wide work;
* performs a conversion with no natural receiver;
* has no meaningful owning type.

Do not turn every helper into a method merely for stylistic uniformity.

Conversely, do not introduce a meaningful domain type and leave its intrinsic behavior in package-level functions that accept primitive representations.

A file centered on a method-bearing type should contain that type, its methods, and its constructors.

Constructors are the normally allowed package-level functions in a type-centered file.

Do not mix unrelated package-level helpers into a type-centered file.

If logic conceptually belongs to the primary type, implement it as a method.

If logic genuinely does not belong to the type and must remain package-level, place it with the concern it actually serves rather than mixing it into a type-centered file.

Keep conversion helpers near the boundary or concern they serve.

# Dependencies and construction

Required dependencies must be explicit.

Construct required services or dependencies once at a clear composition root and pass them into consumers.

A constructor must not interpret a nil required dependency as a request to construct a hidden default.

Tests should construct required dependency graphs explicitly rather than relying on production-only hidden initialization.

Optional callbacks, options, or dependencies may have defaults only when their optional nature and default behavior are intentional and clear.

Avoid service locators, hidden globals, implicit initialization, and invisible dependency construction when explicit construction is practical.

Keep option validation, trimming, normalization, and defaulting close to the option-owning type or constructor.

Do not repeat normalization rules across unrelated layers.

# Nil semantics

Do not silently assign convenient semantics to nil.

Required dependencies should reject or make impossible nil values rather than converting nil into hidden defaults.

Require non-nil `context.Context` values at operation boundaries. Do not silently replace nil with `context.Background()`.

Do not normally make nil receivers valid domain objects or map nil receivers to lifecycle states such as closed.

Use nil semantics only when nil is genuinely part of the intentional contract.

# Resource ownership and lifecycle

Make resource ownership visible in APIs.

A type that owns a closable resource should normally expose the lifecycle operation itself rather than exposing a DTO-like field that callers must discover and close manually.

When the distinction matters, make clear whether resources are:

* owned;
* borrowed;
* leased;
* transferred.

Release partially acquired resources on every failure path.

Cleanup must remain correct on:

* successful completion;
* errors;
* cancellation;
* early returns;
* partial initialization;
* repeated shutdown or close operations when idempotency is part of the contract.

Do not eagerly materialize, retain, copy, or promote expensive resources without a concrete need.

When values can escape their current execution or ownership scope, make the resulting ownership transition explicit.

Lifecycle transitions should have one authoritative representation.

Derived flags, events, atomics, and channels must not become competing sources of truth.

Do not represent the same lifecycle independently through several synchronization mechanisms without a concrete reason.

# Context and cancellation

Accept `context.Context` at operation boundaries that can:

* block;
* perform I/O;
* be canceled;
* perform potentially long-running work;
* participate in a caller-owned lifecycle.

Check or propagate cancellation early enough to avoid committing state after cancellation.

Propagate caller contexts rather than replacing them with `context.Background()` without a concrete protocol or lifecycle reason.

Do not store contexts in long-lived structs.

Store explicit lifecycle state and cancellation functions when ownership requires them.

Long-running work must not outlive its owning context without an explicit lifecycle reason.

# Concurrency

Every goroutine must have:

* a clear owner;
* a termination condition;
* a cleanup path.

Reason about goroutine termination under:

* normal completion;
* errors;
* cancellation;
* partial startup;
* repeated shutdown.

Avoid goroutine leaks.

Identify which mutex protects each field or cohesive state group.

Keep lock scope narrow and make protected state obvious from the type layout or a focused invariant comment.

Prefer one cohesive lock-owned representation when fields participate in the same lifecycle transition.

Do not mix mutexes, atomics, channels, once-guards, and duplicate flags for the same state without a concrete ordering or performance reason.

Do not call unknown, external, blocking, or potentially re-entrant code while holding a lock unless the ordering requirement is explicit and tested.

Do not hold service locks across blocking I/O or potentially long-running operations unless a documented invariant requires it.

Return copies or immutable views when callers must not mutate synchronized internal state.

Preserve required ordering between state changes and externally visible events.

Scrutinize repeated hand-written lifecycle or synchronization state machines. Extract a shared mechanism only when the semantics and ownership genuinely match.

Never trade obvious domain ownership for a clever concurrency abstraction.

Concurrency comments should explain ownership, invariants, and non-obvious ordering rather than narrating individual statements.

For changes that add or materially alter shared mutable state or goroutine coordination:

* add deterministic lifecycle/concurrency tests;
* run the race detector on affected packages.

# Error handling

Use standard Go error mechanisms.

Preserve error identity with `%w`, `errors.Is`, and `errors.As` when callers need classification.

Add context at subsystem and process boundaries without repeating the entire call chain.

Keep sentinel errors for stable conditions callers need to classify.

Use typed errors when they express a meaningful structured contract.

Do not compare error strings in production code when `errors.Is`, `errors.As`, a sentinel, or a typed error can express the contract.

Distinguish failures such as:

* cancellation;
* invalid input;
* missing state;
* stale state;
* dependency failure;
* external or transport failure;
* runtime/domain failure;
* internal invariant violation;

when callers need different behavior.

Do not collapse expected user/domain failures and internal invariant violations into the same conceptual error class.

Do not log and return the same error at every layer. The owning process, transport, or presentation boundary should decide how to report it.

Error messages should normally be concise lowercase sentence fragments unless proper names or externally defined text require otherwise.

# Go type and file structure

These rules are mandatory unless the task explicitly requires otherwise.

Do not define multiple substantial method-bearing structs in the same `.go` file.

Prefer declaring a method-bearing struct as a standalone declaration:

```go
type Service struct {
	// ...
}
```

A method-bearing struct should usually live in its own file named after the primary type or responsibility whenever practical, for example:

* `service.go` for `Service`;
* `manager.go` for `Manager`;
* `result.go` for `Result`;
* `session.go` for `Session`.

Grouped `type (...)` declarations are allowed for:

* interfaces;
* passive data-only structs;
* enums and related value types;
* small related helper types belonging to one narrow concern.

A grouped declaration may contain exactly one method-bearing struct only when:

* it is the file's sole behavioral type; and
* the other grouped declarations are passive helpers from the same narrow concern.

Do not use grouped declarations to hide multiple substantial behavioral types.

If a helper gains methods and would create another substantial method-bearing type in the same file, extract it into its own file.

Methods should live with their struct unless a strong concern-based split makes the result clearer.

Do not place a new method-bearing struct into an existing file merely because the code compiles.

A file centered on a behavioral type should remain centered on that type.

# Package, file, and abstraction organization

Do not use `helpers.go`, `utils.go`, `common.go`, or similarly generic files as long-term containers for unrelated functionality.

A helper-focused file is acceptable while its contents represent one cohesive concern.

As a concern grows, organize files around responsibilities a reader can predict, such as:

* lifecycle;
* conversion;
* snapshots;
* parameters;
* identifiers;
* protocol state;
* validation.

Keep package boundaries domain-oriented.

Do not create a package solely to:

* shorten files;
* remove a few repeated lines;
* manufacture an abstraction layer;
* avoid keeping related private implementation together.

Prefer cohesive private implementation types and files over unnecessary package fragmentation.

Keep symbols unexported until another package has a real need for them.

Avoid both extremes:

* files, functions, types, or packages doing too much;
* tightly related behavior fragmented across excessive helpers, files, interfaces, or packages.

Behavioral ownership should be predictable from code organization.

# Local types

Local types declared inside functions are allowed when they are:

* small;
* passive;
* method-free;
* used only by that function;
* purely part of the local algorithm;
* easier to understand locally than at package scope.

Prefer a package-level unexported type when the type:

* represents a meaningful domain, lifecycle, protocol, state, or algorithmic concept;
* is used across a substantial function or by nearby helpers;
* would make control flow easier to scan when declared separately;
* may reasonably gain methods;
* is likely to be reused;
* clarifies ownership or responsibility at package scope.

Do not promote tiny throwaway structs merely for consistency.

Do not hide meaningful concepts inside long functions merely to avoid another package-level declaration.

Choose based on readability, conceptual ownership, and expected evolution.

# Comments

Do not add comments to every function or method by default.

Exported declarations should have useful doc comments when they define package-facing contracts.

Comment unexported code only when it carries non-obvious:

* semantics;
* invariants;
* side effects;
* ownership;
* synchronization;
* lifecycle behavior;
* cleanup requirements;
* recovery behavior;
* compatibility constraints.

Comments should explain:

* why the code exists;
* what must remain true;
* what the contract guarantees;
* how ownership works;
* why ordering matters.

Do not merely restate names or signatures.

Prefer:

```go
// Close releases resources owned by the result.
// It is safe to call multiple times. Once closed, the result must not be reused.
func (r *Result) Close() error
```

Avoid:

```go
// Close closes the result.
func (r *Result) Close() error
```

Prefer semantic and invariant comments over implementation narration.

Keep future plans out of code comments unless the comment describes a deliberate current boundary.

Update or remove comments when implementation changes make them obsolete.

Avoid comment wallpaper. Dense, meaningful comments are preferable to mechanically documenting obvious code.

# Go control-flow spacing

These rules apply to handwritten Go code.

Blank lines should separate logical units and make control-transfer boundaries easy to scan.

## Immediate producer and check

A declaration, assignment, lookup, call, parse, or type assertion may remain directly adjacent to the `if` that immediately checks or consumes its result.

Preferred:

```go
value, err := load()
if err != nil {
	return err
}
```

Preferred:

```go
value, ok := values[name]
if !ok {
	return ErrNotFound
}
```

The producer and immediate check form one logical unit.

Do not separate them with an artificial blank line.

## Separation from preceding work

If a producer-and-check unit follows another logical unit, separate it from the preceding work.

Preferred:

```go
prepareState()

value, err := load()
if err != nil {
	return err
}
```

## Consecutive control flow

Separate independent control-flow blocks.

Preferred:

```go
if foo != nil {
	useFoo(foo)
}

if bar != nil {
	useBar(bar)
}
```

Add a blank line after completed control flow before continuing with an independent statement.

## Return and break separation

When another statement precedes `return` or `break` in the same block, begin the control transfer as a new logical group.

Preferred:

```go
result := buildResult()

return result
```

Preferred:

```go
if ready {
	state = stateRunning

	break
}
```

No blank line is required when `return` is already the first statement in its block:

```go
if err != nil {
	return err
}
```

Do not surround every `return` or `break` mechanically.

The purpose of these rules is to expose logical structure, not maximize whitespace.

# API and compatibility discipline

Treat observable behavior as intentional until the task establishes otherwise.

Do not change public, external, language-visible, wire-visible, CLI-visible, persistence-visible, or integration-visible behavior as collateral cleanup.

Do not export new symbols merely to share implementation internally.

Prefer unexported helpers inside the owning package before expanding an API surface.

When a new exported symbol is genuinely necessary, document the external contract clearly.

For compatibility-sensitive changes:

* make the behavior change explicit;
* preserve previous behavior unless incompatibility is required;
* add focused tests at the observable boundary;
* document intentional incompatibility;
* avoid unrelated contract changes.

Do not infer desired behavior from historical discussions, abandoned designs, stale comments, or future-looking architecture when current implementation and tests establish a different contract.

# Tests

Add or update tests for every behavior change.

Put tests beside the package or layer that owns the behavior whenever practical.

Test observable contracts rather than mirroring implementation details.

Prefer focused table-driven tests when several inputs exercise the same contract.

Use `t.Helper()` in reusable test helpers.

Use `t.Cleanup()` for restoring globals, closing resources, canceling contexts, or stopping goroutines.

Avoid sleeps as synchronization.

Use channels, contexts, deadlines, barriers, or observable state.

Keep timeouts bounded and generous enough for CI while still detecting leaks and deadlocks.

Verify both success and failure paths.

Test relevant:

* positive cases;
* negative cases;
* boundary conditions;
* invalid inputs;
* cancellation;
* cleanup;
* repeated operations;
* idempotency;
* stale state;
* error identity;
* concurrency behavior.

For bug fixes, add a regression test that fails without the fix whenever practical.

When behavior crosses meaningful package or integration boundaries, include integration-level coverage rather than relying exclusively on direct method tests.

Do not add redundant tests that increase maintenance cost without protecting meaningful behavior.

Avoid brittle tests unnecessarily coupled to implementation details.

Assertions must verify meaningful behavior strongly enough that plausible regressions fail.

A passing test suite is evidence of correctness. It is not evidence that the design is good.

# Performance

Do not optimize by intuition.

A change is performance-significant when it could reasonably affect:

* execution throughput;
* latency on common or hot paths;
* allocation patterns;
* memory usage or retention;
* repeated parsing, compilation, conversion, or serialization;
* caching or pooling;
* synchronization or lock contention;
* resource cleanup;
* materialization cost;
* startup or shutdown performance;
* long-running process memory behavior.

Documentation-only, test-only, pure rename, formatting-only, and narrow non-hot-path refactoring changes are normally not performance-significant.

When uncertain whether a change affects a hot path, treat it as performance-significant and measure it.

For performance-significant changes:

1. Identify an existing focused benchmark or add one.
2. Run it before implementation and retain the baseline.
3. Implement the change.
4. Run the same benchmark afterward under comparable conditions.
5. Compare relevant metrics such as `ns/op`, `B/op`, and `allocs/op`.
6. Investigate meaningful regressions before considering the task complete.

Inspect performance-sensitive implementations for:

* accidental allocations;
* unnecessary copying;
* repeated conversions;
* repeated computation;
* unnecessary materialization;
* avoidable synchronization;
* increased lock contention;
* blocking work added to hot paths;
* unnecessary work added to disabled or optional paths;
* resources retained longer than necessary.

Do not trade clear correctness or maintainability for speculative micro-optimization.

If no relevant benchmark exists for an affected hot path, add one when practical.

If benchmark tooling or the environment is unavailable, state that explicitly rather than claiming performance validation.

# Change discipline

Keep the diff focused on the requested task.

Do not perform opportunistic:

* refactoring;
* dependency upgrades;
* formatting churn;
* API redesign;
* package reshuffling;
* abstraction creation;
* generated-file changes;
* documentation rewrites;
* implementation of future features;

unless they are required by the requested change.

Do not modify unrelated code merely to make it conform stylistically.

Do not use an implementation task as an excuse to clean up the surrounding repository.

A cleanup discovered while working may be included when it is:

* small;
* local;
* low-risk;
* clearly understood;
* directly related to the affected area;
* beneficial to correctness, lifecycle safety, ownership, or maintainability of the requested change.

If a discovered issue requires broader architectural work, preserve current behavior and report it for a separate task.

Preserve unrelated dirty, modified, and untracked files.

Do not overwrite, revert, or reformat unrelated user changes.

Do not update dependencies unless the task requires a dependency change.

Do not change compatibility-sensitive contracts as collateral cleanup.

Generated files must be changed through their source inputs or generator when the repository provides such a workflow. Do not manually edit generated output.

Inspect generated diffs when generation is required.

# Required workflow for non-trivial changes

For every non-trivial coding task:

1. **Identify ownership.**
   Determine the subsystem, package, type, or layer that owns the requested behavior.

2. **Identify the contract.**
   Determine the observable behavior, invariants, compatibility requirements, lifecycle, resource ownership, and error semantics being preserved or changed.

3. **Understand the current implementation.**
   Read current source and tests before relying on architecture prose, historical discussion, old branches, or assumptions.

4. **Choose the smallest coherent design.**
   Prefer a local, comprehensible implementation that fits existing ownership boundaries.

5. **Evaluate risk.**
   Determine whether the change is concurrency-sensitive, lifecycle-sensitive, compatibility-sensitive, or performance-significant.

6. **Establish a performance baseline when necessary.**
   Run relevant benchmarks before changing performance-sensitive code.

7. **Add or update correctness tests.**
   Define the observable behavior the implementation must satisfy.

8. **Implement the change.**
   Keep the implementation focused on the requested behavior.

9. **Run focused validation.**
   Run the narrowest tests and checks that directly exercise the changed behavior first.

10. **Broaden validation according to risk.**
    Run package, integration, race, lint, build, generation, or repository-level validation as appropriate.

11. **Perform the mandatory final self-review.**
    Review the implementation itself rather than merely confirming that automated checks pass.

12. **Correct review findings.**
    Fix problems introduced by the task and appropriate directly adjacent issues according to the scope rules below.

13. **Re-run affected validation.**
    Any validation invalidated by review-driven changes must be repeated.

14. **Re-run affected benchmarks.**
    If review-driven corrections affect benchmarked code, repeat the relevant benchmark comparison.

15. **Inspect the complete final diff.**
    Review the change as one coherent unit.

16. **Report accurately.**
    State what changed, what was tested, what was measured, what was reviewed, and what remains unresolved.

Do not consider a task complete merely because the implementation compiles and its tests pass.

# Mandatory final self-review

Every non-trivial coding task must end with a deliberate design and implementation review before it is considered finished.

Review the final implementation as though reviewing another engineer's pull request.

The review must evaluate the code itself, not merely confirm that compilation, tests, lint, static analysis, or benchmarks succeeded.

Review all changed code and directly adjacent code necessary to understand the change.

For non-trivial work, inspect the complete diff as a coherent change.

The purpose of self-review is to catch correctness, design, quality, organization, performance, and maintainability problems introduced or exposed by the task.

It must not become justification for unrelated refactoring or redesign.

## Correctness

Verify:

* every requested behavior is implemented;
* explicit non-goals remain untouched;
* existing behavior is preserved unless intentionally changed;
* assumptions made by the implementation are valid;
* boundary conditions are handled;
* failure paths are handled;
* partial operations do not leave invalid state;
* errors preserve required identity and context;
* cancellation works where applicable;
* resources are cleaned up on every relevant path;
* cleanup and shutdown are idempotent where required;
* lifecycle transitions remain valid;
* stale state is handled correctly where relevant;
* ordering requirements remain correct;
* concurrent behavior remains correct;
* goroutines terminate;
* locks are not held across inappropriate work;
* public or externally observable semantics match the intended contract.

Look actively for missing cases and regressions rather than reviewing only the successful path.

For bug fixes, verify that a regression test fails without the fix whenever practical.

Ensure tests would detect plausible regressions rather than merely repeat implementation structure.

## Code clarity and cleanliness

Review the implementation for:

* unnecessary complexity;
* duplicated behavior;
* duplicated semantics;
* excessive nesting;
* awkward control flow;
* misleading naming;
* overly large functions;
* hidden state transitions;
* hidden ownership;
* unnecessary mutation;
* unnecessary indirection;
* difficult-to-follow execution paths;
* dead branches;
* temporary implementation artifacts;
* debugging output;
* obsolete helpers;
* abandoned approaches left in comments or code.

The primary execution path should remain easy to follow.

Prefer straightforward code whose behavior can be understood locally.

Simplify code when the simpler implementation is clearly equivalent and easier to reason about.

Do not perform stylistic rewrites merely because another form is also valid.

## Go design and API quality

Check:

* API consistency;
* naming;
* semantic-type grounding;
* method-versus-function ownership;
* constructor behavior;
* dependency construction;
* nil semantics;
* option ownership and normalization;
* enum zero values;
* resource ownership;
* lifecycle visibility;
* context propagation;
* error wrapping;
* synchronization;
* lock scope;
* goroutine ownership;
* cleanup behavior.

Look specifically for:

* meaningful types bypassed by primitive APIs;
* free functions containing behavior naturally owned by a type;
* methods whose behavior does not naturally belong to their receiver;
* required dependencies hidden behind nil defaults;
* ambiguous resource ownership;
* repeated option normalization;
* competing lifecycle representations;
* generic helpers containing unrelated behavior.

Do not introduce a pattern merely because it is common or fashionable elsewhere. It must improve this codebase specifically.

## Abstraction quality

Review every new abstraction critically.

Ask:

* Is this abstraction required by the current problem?
* Does it represent a real concept?
* Are its implementations genuinely substitutable?
* Does it clarify ownership?
* Does it reduce meaningful duplication?
* Does it simplify reasoning?
* Would direct concrete code be clearer?
* Is this abstraction preparing for hypothetical future requirements rather than solving a current need?

Remove abstractions that do not earn their complexity.

Do not generalize two concepts merely because their implementations look structurally similar when their semantics, ownership, or lifecycle differ.

## Architecture

Verify:

* behavior remains in the correct subsystem, package, type, and layer;
* dependency direction remains clear;
* domain behavior has not leaked into transport, adapter, command, or presentation layers;
* adapters translate and delegate rather than becoming alternate implementations;
* implementation details have not leaked unnecessarily across package boundaries;
* semantics are defined once rather than duplicated by consumers;
* new exported APIs are genuinely necessary;
* package boundaries remain meaningful;
* abstractions exist at the correct level;
* current implementation has not accidentally incorporated speculative future architecture.

Consider whether the design will remain understandable as the feature evolves, without attempting to design hypothetical future features now.

## Code organization and split

Verify that files, types, methods, functions, and packages have coherent responsibilities.

Check compliance with the type/file structure rules.

Check compliance with function/method ownership rules.

Look for:

* files doing too much;
* types owning unrelated behavior;
* functions performing several unrelated operations;
* package-level helpers mixed into type-centered files;
* multiple substantial behavioral types hidden in one file;
* generic utility dumping grounds;
* meaningful concepts hidden as local implementation details;
* unrelated responsibilities grouped together.

Also look for the opposite problem:

* excessive helper extraction;
* unnecessary file splitting;
* tiny interfaces without meaningful boundaries;
* package fragmentation;
* layers that merely forward calls;
* abstractions whose only effect is additional navigation.

Keep tightly related behavior cohesive.

Helpers should exist at the narrowest appropriate ownership level.

A reader should be able to predict where behavior lives from its responsibility.

## Comments and documentation

Re-read comments directly affected by the change.

Verify that comments:

* describe current behavior;
* describe current contracts;
* describe invariants accurately;
* explain ownership correctly;
* do not describe abandoned implementation approaches;
* do not speculate about future architecture unnecessarily;
* remain useful after the implementation change.

Remove comments made obsolete by clearer code.

Do not add comments merely to compensate for unnecessarily confusing implementation.

When a change alters user-visible, integration-facing, or public API behavior, evaluate whether documentation must be updated.

Documentation synchronization is part of a behavior change when existing documentation would otherwise become incorrect.

Do not use documentation impact as justification for unrelated documentation cleanup.

## Tests

Review the tests themselves, not only their result.

Look for missing:

* positive cases;
* negative cases;
* boundary cases;
* invalid inputs;
* cancellation paths;
* cleanup paths;
* repeated-operation cases;
* idempotency cases;
* stale-state cases;
* error-classification cases;
* concurrency cases;
* integration coverage where behavior crosses boundaries.

Check for:

* weak assertions;
* tests that merely mirror implementation details;
* brittle dependence on internal structure;
* redundant tests with little behavioral value;
* flaky timing;
* sleeps used for synchronization;
* leaked goroutines;
* leaked resources;
* mutable global state;
* unnecessarily narrow happy-path coverage.

Verify that tests assert meaningful observable behavior.

For errors whose identity is part of the contract, test classification rather than only message strings.

For concurrency behavior, prefer deterministic lifecycle tests.

For user-visible behavior spanning multiple layers, ensure package-local tests are supplemented by appropriate boundary or integration coverage.

## Performance

For performance-significant changes, review the final implementation for:

* accidental allocations;
* repeated work;
* unnecessary copying;
* unnecessary conversions;
* unnecessary materialization;
* unnecessary synchronization;
* lock contention;
* blocking work;
* memory retention;
* hot-path overhead;
* expensive work added to optional or disabled paths.

Compare final benchmark results against the pre-change baseline.

Verify that benchmark setup remained comparable.

Investigate meaningful regressions.

Do not rationalize a regression merely because correctness tests pass.

Do not trade clear correctness or maintainability for speculative micro-optimization.

# Self-review findings and remediation

When self-review finds a problem, classify it before changing code.

## 1. Problems introduced by the task

Fix every meaningful deviation introduced by the task.

This includes:

* correctness problems;
* regressions;
* lifecycle problems;
* resource leaks;
* concurrency problems;
* ownership violations;
* architecture violations;
* API problems;
* significant maintainability problems;
* significant test-coverage gaps;
* meaningful performance regressions caused by the change.

Do not leave such problems unresolved merely because the initial implementation already works or tests pass.

## 2. Directly adjacent pre-existing problems

A pre-existing issue may be fixed when the correction is:

* small;
* local;
* low-risk;
* clearly understood;
* directly within the affected area;
* relevant to correctness, ownership, lifecycle, architecture, or maintainability of the requested change.

Do not copy a poor existing pattern merely because it already exists.

Existing technical debt is not precedent.

## 3. Broader pre-existing problems

If a discovered problem requires:

* broad refactoring;
* package restructuring;
* API redesign;
* unrelated cleanup;
* dependency changes;
* speculative architecture;
* substantial additional behavior;

leave it unchanged and report it as separate follow-up work.

Do not allow self-review to expand the task without a concrete reason.

# What self-review must not become

Do not use self-review as justification for:

* speculative refactoring;
* unrelated cleanup;
* rewriting correct code for stylistic consistency;
* unrelated API redesign;
* broad package reshuffling;
* dependency upgrades;
* introducing abstractions without a concrete need;
* implementing future features;
* changing unrelated behavior;
* changing semantics outside the requested task.

Distinguish actual problems from optional preferences.

Existing code that is clear, correct, idiomatic, appropriately designed, and appropriately organized should be left alone.

# Final diff inspection

Immediately before finishing every non-trivial task, inspect the complete final diff as a whole.

Do not review only individual edited files in isolation.

Verify that:

* every changed line belongs to the requested task or a necessary supporting change;
* unrelated user changes remain intact;
* no temporary code remains;
* no debugging output remains;
* no dead or abandoned implementation remains;
* no accidental behavior changes slipped in;
* no accidental public API changes slipped in;
* no accidental compatibility changes slipped in;
* no accidental dependency changes slipped in;
* no unrelated refactors slipped in;
* generated files changed only when their source inputs required regeneration;
* tests describe intended behavior rather than implementation details;
* comments describe current contracts and invariants;
* package boundaries remain coherent;
* file responsibilities remain coherent;
* type responsibilities remain coherent;
* method and function ownership remains coherent;
* cancellation remains correct;
* concurrency remains correct;
* cleanup remains correct;
* resource lifetimes remain correct;
* the resulting implementation is the smallest coherent change that fully solves the task.

Inspect the diff for formatting churn and unrelated whitespace changes.

If final inspection causes another code change, repeat every validation or benchmark whose result may have been invalidated.

The final diff, not an earlier intermediate implementation, is what must satisfy this guide.

# Validation discipline

Run the narrowest validation that proves the changed behavior first.

Then broaden according to scope and risk.

Typical progression:

1. focused tests for the affected package or behavior;
2. directly affected integration tests;
3. race detection for concurrency-sensitive changes;
4. static analysis or lint where relevant;
5. broader repository tests;
6. build or compilation validation;
7. generation checks when generated artifacts are involved.

Do not run unrelated expensive validation merely to create validation theater.

Conversely, do not stop at a narrow unit test when the change affects behavior across packages or external boundaries.

After review-driven changes, re-run every command whose result may have been invalidated.

Never claim a validation command succeeded unless it was actually run successfully.

If validation cannot be completed because of tooling, environment, permissions, external dependencies, or time constraints, report the limitation explicitly.

# Validation evidence

When finishing a non-trivial change, report:

* the owning subsystem;
* files changed;
* behavior changed;
* important behavior and invariants preserved;
* tests added or updated;
* validation commands actually run;
* race-detector validation when applicable;
* benchmarks added or updated when applicable;
* benchmark commands and before/after comparison when applicable;
* final self-review completion;
* meaningful issues found and corrected during self-review;
* noteworthy pre-existing issues intentionally left outside scope;
* remaining concerns or limitations;
* environmental or tooling failures that prevented validation.

Do not claim:

* tests passed unless they were run;
* lint passed unless it was run;
* builds passed unless they were run;
* race detection passed unless it was run;
* benchmarks were completed unless they were run;
* generation was verified unless it was verified;
* self-review was completed unless the final implementation and diff were actually inspected.

Accuracy of the completion report is part of engineering quality.

# Decision bias when uncertain

When uncertain:

* inspect current source and tests before relying on assumptions;
* preserve existing observable behavior;
* identify ownership before adding behavior;
* prefer the smaller local change;
* prefer concrete code over speculative abstraction;
* keep dependencies explicit;
* make ownership and lifecycle explicit;
* preserve error identity;
* propagate cancellation;
* add a focused test;
* treat concurrency changes cautiously;
* measure when performance might be affected;
* fix concrete review findings before speculative cleanup;
* avoid expanding the task unnecessarily;
* leave already-correct code alone.

When choosing between a clever implementation and an obvious implementation that satisfies the same requirements, prefer the obvious implementation.

When choosing between an abstraction that might become useful and concrete code that clearly solves the current problem, prefer the concrete code.

When choosing between broad cleanup and a focused change, prefer the focused change.

When choosing between assuming behavior and verifying it in source or tests, verify it.

# Definition of done

A non-trivial coding task is complete only when:

* ownership and contracts were understood;
* the requested behavior is implemented;
* relevant existing behavior is preserved;
* tests cover the meaningful behavior;
* relevant validation has passed;
* concurrency validation has been performed when applicable;
* performance has been measured when applicable;
* the implementation has undergone mandatory self-review;
* review findings introduced by the task have been corrected;
* affected validation has been repeated after corrections;
* the complete final diff has been inspected;
* the final change is focused and coherent;
* completion results and limitations are reported accurately.

Compiling is not completion.

Passing tests is not completion.

The standard is a correct, clean, appropriately designed, well-tested, deliberately reviewed change.