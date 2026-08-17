---
name: test-value-audit
description: Audit tests for distinct defect-detection value against their maintenance cost, reading production code and tests together. Use when reviewing a test file, diff, subsystem, or full suite for redundant, implementation-detail, overly defensive, over-mocked, or otherwise low-value tests; when deciding what to keep, merge, remove, rewrite, or add; or when hunting for missing coverage of business rules, boundaries, and failure cases. Never edit production source code. Change tests only when explicitly requested.
---

# Test Value Audit

Evaluate whether each test protects meaningful runtime behavior that another test does not already protect. Prefer the smallest coherent suite that catches plausible regressions in business rules, boundaries, failures, and external contracts.

## Audit workflow

1. Infer the scope and any excluded test families from the request; ask only when genuinely ambiguous. Inspect repository instructions, the current diff, and authoritative test commands. When a test command is available, run the suite and use runtimes, failures, flakes, and skip counts as evidence; do not modify anything to make it pass. Preserve unrelated work.
2. Read each test with the production code, call sites, and relevant requirements. For a large suite, work subsystem by subsystem.
3. Map the behavior before judging the tests: business rules, state transitions, boundaries, failure modes, persistence, concurrency, security, external schemas, and user-visible outcomes.
4. Ask of each test:
   - What invariant or observable behavior does it protect?
   - What plausible defect would make it fail?
   - Is its oracle independent enough to detect a defect, rather than repeat the implementation?
   - Does another test already catch the same defect?
   - Does its value justify its setup, mocks, runtime, brittleness, and production complexity?
5. Recommend `keep`, `merge`, `remove`, `rewrite`, or `add`. Use `merge` when several tests protect one invariant and should collapse into the strongest of them; use `rewrite` when the behavior deserves protection but the current oracle, fixture, or mocking is too weak to catch a defect.
6. Never edit production source code. Change test files only when explicitly requested; otherwise, report findings with evidence and rationale.

## Favor valuable coverage

Keep tests that protect a named contract such as:

- business decisions, calculations, transformations, and state transitions;
- boundary values and materially different input classes;
- errors, retries, cancellation, ordering, concurrency, time, precision, or recovery;
- persistence, migration, durability, locking, or conflict resolution;
- external wire formats, schemas, commands, and user-visible output;
- authorization, security, privacy, and redaction;
- integration wiring that cannot be exercised meaningfully at a lower layer;
- a realistic or previously observed regression.

Treat internal details as eligible when changing them would break compatibility, safety, durability, performance, or another named contract. Do not remove a test merely because it touches an internal API.

## Challenge low-value coverage

Treat these as signals for closer review, not automatic deletion:

- tests that cannot fail: assertion-free bodies, unawaited async assertions, expectations inside callbacks that never run, assertions swallowed by error handling, or skipped and quarantined tests;
- assertions that restate assignments, getters, direct field copying, constants, enum membership, types, schemas already enforced by tooling, or source-visible defaults;
- checks of private helper identity, registration order, call structure, incidental database layout, or other replaceable implementation choices;
- several permutations that traverse the same branch and use the same oracle without protecting distinct boundary behavior;
- duplicate assertions or tests that protect the same invariant at the same effective layer;
- assertions that only confirm a mock returned the value configured by the test;
- broad snapshots, existence checks, or type checks that add nothing beside a stronger behavioral assertion;
- defensive cases with no plausible failure mechanism or business consequence.

Do not equate shared coverage or similar syntax with redundancy. Two tests can execute the same lines while protecting different contracts. Conversely, different fixtures can be redundant when they catch the same defect.

## Audit mocking and testability cost

Mock stable boundaries such as external services, clocks, randomness, slow I/O, and nondeterministic systems when isolation is useful. Prefer exercising real business logic behind those boundaries.

Flag mocking when it:

- replaces the behavior under test or mirrors its control flow;
- verifies self-authored call choreography instead of an outcome;
- makes impossible states look valid;
- hides serialization, persistence, framework, or integration behavior that carries the actual risk;
- requires production indirection, protocols, factories, or visibility changes whose only benefit is testing trivial logic.

Judge the test and any test-driven production seam as one maintenance-cost decision. Keep a seam when it also improves ownership, substitution, isolation, or architecture; question it when it exists only to support a weak test. Report possible production-code simplifications as recommendations only; never implement them. Do not demand an integration test, a negative twin, or a new abstraction for every mocked unit test.

## Find missing value

After pruning candidates, look for meaningful gaps the suite's volume may obscure. Add or propose a test only when it covers a distinct rule, boundary, failure, or integration risk. Prefer extending the canonical test for an invariant over creating another test file or defensive permutation.

Use mutation testing, per-test coverage, or test history only when an important redundancy decision remains ambiguous or the repository already supports them. Do not require heavyweight analysis for straightforward source-restating tests.

## Report with evidence

Open with a short summary of suite health and the highest-impact findings. Order findings by how much they change maintenance cost or regression risk, and do not pad the report with every trivial keep. Group findings by verdict; for each, name the test, the contract it does or does not protect, and the rationale, citing evidence such as the production code exercised, the duplicating test, runtime, or flake history. Highlight non-obvious keeps, removal or merge candidates, mocking trade-offs, and missing high-value cases. Distinguish confirmed findings from uncertain recommendations, and state any validation limits.
