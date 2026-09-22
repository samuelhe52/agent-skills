---
name: experiment-infra-design
description: Design or review experiment infrastructure that controls execution, scoring, recovery, or evidence retention. Use when infrastructure choices can affect experimental validity or the cost of a failed run.
---

# Experiment Infrastructure Design

Keep results trustworthy and make failures recoverable at a cost appropriate to the experiment. Adapt the design and verification to the consequences of failure, the resources at stake, and how readily the work can be repeated.

## Scope and proportionality

Inspect the component, its consumers, and the existing execution path before proposing a design. Reuse established infrastructure where it fits, and check the effective configuration where work actually runs.

A cheap, supervised probe may need little beyond clear outcomes and useful failure evidence. Expensive or unattended work calls for stronger recovery, observability, and resource controls. Choose the relevant safeguards without imposing a fixed review tier or requiring a full audit for every change.

## Contracts and recovery

Understand the experiment's intended meaning and how the infrastructure can affect it. Failure handling must distinguish missing or invalid results from valid outcomes; recovery that changes what is measured or which observations are included is an experimental decision and should be explicit.

Define how failures reach the consumer and where recovery belongs. A recoverable item failure need not abort a whole run, but continuing is only useful if the remaining result still has a clear interpretation. Check both successful recovery and what happens when recovery is exhausted.

## Evidence and provenance

Retain enough provenance and evidence to explain results, diagnose failures, and support the intended reproduction or recovery. Choose the detail and retention policy for the workload rather than requiring a fixed record format.

Connect results to their inputs, effective configuration, and relevant code or model versions. Preserve original outcomes separately from repairs or reinterpretations. For example, retaining a raw response before parsing can preserve the evidence needed to diagnose a parser failure; a console excerpt may not suffice.

Decide what must survive cleanup based on later analysis and recovery needs. Account for storage cost and sensitive data when selecting what to retain.

## Execution lifecycle

Consider retries, concurrency, and restart behavior across the whole execution path. Local safeguards may not bound aggregate resource use or prevent duplicate work. Make completion, interruption, and recovery behavior clear where they matter.

Retries at several layers can multiply attempts, and per-worker concurrency limits can multiply total load. Bound the resources that matter to the experiment. For resumable work, distinguish completed work from pending or failed work and avoid reusing results under incompatible configurations.

Long runs should expose progress, stalled work, and a usable cancellation path. Match persistence and duplicate-handling guarantees to the consequences of interruption.

## Verification and handoff

Verify consequential assumptions at the boundary where they take effect. Choose checks that address plausible failures and the validity of the result; a successful launch alone does not establish either reliability or experimental success.

Use representative inputs and the real consumer where practical. Before scaling an expensive workload, a small end-to-end run can check integration, while targeted failure tests establish what a successful run cannot. Revisit verification when a change invalidates an earlier assumption; reuse evidence that still applies.

Keep recommendations and implementation within the requested scope. Explain material tradeoffs and unresolved uncertainty, with detail proportional to the task.

For LLM judges and response-processing components, consult [LLM boundaries and failure patterns](references/llm-boundaries-and-failures.md) when their output or failure semantics need closer examination.
