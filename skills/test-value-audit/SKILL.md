---
name: test-value-audit
description: Audit tests for distinct defect-detection value relative to their maintenance cost, reading tests alongside production code. Identify redundant or ineffective tests and meaningful coverage gaps. Never edit production code; change tests only when explicitly requested.
---

# Test Value Audit

Assess which plausible regressions a test detects and whether that protection justifies its maintenance cost. Read tests alongside the behavior they exercise and the requirements they are meant to protect.

When proposing or writing a test, identify the observable contract, a credible regression that would make the test fail, and why existing tests would not catch it. Prefer extending an existing test when it can express the distinct risk clearly. Question a production seam introduced only to support a weak test.

## Assess the protection

Identify the contract, the plausible defect, and how the test would expose that defect. Useful protection can cover ordinary business behavior, boundary conditions, failure handling, or interactions between components. Choose the relevant dimensions for the code under review.

Judge a test by whether a consequential defect would cause it to fail. Its expected result should express the intended contract independently enough to expose an incorrect implementation. Assertions that merely reproduce the implementation or confirm a configured mock response offer little evidence of correctness.

Check whether assertions actually run and whether the test can fail for the intended reason. A swallowed assertion or an unawaited asynchronous check can make apparent coverage ineffective.

Inspect tests whose expected values come from the code under test, mocks that supply the asserted behavior, and negative cases that pass because of an unrelated guard. These patterns can hide a regression; determine whether the test has independent protection before changing it.

Assess redundancy by the defects detected, not by similar syntax or shared line coverage. Tests exercising the same code may protect different behavior. Tests with different fixtures may provide interchangeable protection.

Distinguish stable behavioral requirements from replaceable implementation choices. An internal API can still embody an important contract; being internal is not itself a reason to remove its tests.

## Weigh cost and redundancy

Treat implementation coupling, broad snapshots, and many similar cases as prompts for investigation rather than automatic removal criteria. Determine what protection would be lost before simplifying. A compact test with weak assertions can be less valuable than a longer integration test that detects a distinct failure.

Include setup, brittleness, runtime, mocks, and production complexity introduced for testing in the cost assessment. Isolation is useful when it preserves the behavior under examination. Question mocks that remove that behavior or conceal the failure the test is supposed to detect.

Mocking an external dependency can make a test deterministic; mocking the behavior being tested can make it circular. Evaluate the boundary and the asserted outcome rather than counting mocks. Keep production seams that serve architectural purposes as well as testing, and question complexity introduced solely to support weak tests.

## Find gaps and choose changes

Look for missing protection as well as opportunities to simplify. Recommend adding or strengthening a test when it covers a meaningful risk that the existing suite does not detect. Choose additional investigation or test execution according to what would resolve uncertainty in the assessment.

Prefer extending an existing test when it can clearly express the missing contract. Use coverage, mutation testing, or failure history when they would resolve an important uncertainty; avoid making them mandatory for a straightforward review.

For a bug regression test, seek evidence that it fails on the faulty behavior for the intended reason and passes with the fix. If that check is impractical, state what supports the test and what remains unverified.

Merge tests when their protection is interchangeable and can be retained more simply. Rewrite a test when its intended contract matters but its setup or assertions do not establish it. Before recommending removal, identify the failure the test can detect, any remaining test that detects it, and the protection that would be lost. Check relevant history when the test's purpose is unclear. Remove one only when the lost protection is unnecessary or adequately supplied elsewhere.

## Report and scope

Recommend keeping, merging, removing, rewriting, or adding tests as appropriate, with evidence and a clear account of the protection gained or lost. Prioritize consequential findings and choose a report structure suited to the scope.

Cite the test and relevant production behavior, explain non-obvious recommendations, and distinguish confirmed findings from uncertainties. Report meaningful coverage gaps without enumerating every routine keep.

Never edit production code. Change tests only when explicitly requested; report possible production simplifications as recommendations.
