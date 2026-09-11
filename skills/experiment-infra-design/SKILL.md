---
name: experiment-infra-design
description: Design, change, or review custom experiment infrastructure that controls execution, scoring, recovery, or evidence retention. Use for LLM judges, reward adapters, response parsers, batch runners, generation pipelines, and experiment supervisors; when changing retries, concurrency, persistence, or provider integration; or when promoting a prototype to an expensive or unattended run. Scale review depth to failure cost and experimental validity. Routine operation of an established launcher or metric inspection does not require a full design review.
---

# Experiment Infrastructure Design

Keep experimental results trustworthy while containing failures at the smallest recoverable boundary. Choose rigor from the consequences of failure, not the amount of code or the label “one-off.”

## Establish scope and reuse

- Inspect the requested component, its callers, repository instructions, existing changes, and relevant prior failures. Look for an established harness before inventing a replacement; compare contracts, diagnostics, and recovery behavior, not just features.
- Identify what the component can change: samples, rewards, evaluation denominators, model state, shared resources, and retained evidence. Trace effective configuration through launchers, containers, and workers to the actual execution boundary.
- Keep design/review requests read-only. Implement only within the requested scope and existing authorization. The skill does not authorize launches, remote mutations, publication, or changes to experiment semantics.
- For remote operations, follow the environment's documented procedures when available and verify relevant host assumptions. This skill does not depend on a particular platform, runbook, or companion skill.

## Select the rigor level

State the selected level and its concrete reason briefly. Use only the checks relevant to the changed boundary; do not turn every edit into a full infrastructure audit.

| Level | Use when | Evidence needed |
| --- | --- | --- |
| Exploratory probe | Cheap, supervised, disposable, and easy to rerun | Explicit valid/error outcomes, retained inputs and outputs, bounded execution, a representative successful case and a consequential failure case. A clear failure exit may be sufficient; a resume framework is usually unnecessary. |
| Bounded experiment | Meaningful batch, paid API work, or results intended for comparison | Above, plus effective configuration and provenance, representative calibration where scoring is involved, durable progress, defined retry/resume semantics, and a small end-to-end canary. |
| Rigorous review | Expensive or unattended work, large fan-out, shared-resource effects, training rewards, or evidence that is costly or impossible to reproduce | Above, plus tested terminal failure and recovery, restart compatibility, aggregate resource budgets, progress/stall observability, declared pause/stop thresholds, and a canary covering the changed lifecycle boundary. |

A small reward-parser probe can stay exploratory. Using that parser to supply rewards to a distributed training run warrants rigorous review. A single overnight multi-GPU run is still expensive and unattended.

Apply rigorous review before committing expensive resources. Revisit the affected checks after changes to provider, reasoning mode, output contract, reward policy, concurrency, or persistence. Reuse still-applicable evidence and avoid repeating unrelated validation.

## Define contracts and failure semantics

- Define input identity, output schema, validity criteria, completion state, and the consumer's response to each failure class. Separate transport failures, malformed outputs, unresolved judgments, valid negative results, and configuration errors.
- Never silently turn an unavailable judge into reward zero or a failed trial into an ordinary model failure. Preserve unresolved status even if the final report uses a predefined scoring convention.
- Choose recovery at the correct boundary: retry the pending request, quarantine an item, pause a batch, or stop cleanly. Verify what the real caller does, including propagation through asynchronous tasks and distributed collectives.
- For RL, preserve group membership, reward/advantage semantics, and synchronization consistency. Dropping samples, regenerating candidates, changing judges, or adjusting reward rules can change the experiment; do not hide these inside recovery code.
- Make configuration errors fail promptly. Expected external variability should have an explicit bounded path; unexpected invariant violations should retain evidence and terminate or pause deliberately.

## Make evidence replayable

Connect run, sample/group, logical request, physical attempt, response, interpretation, and resulting action with stable identifiers.

- Record effective configuration and code/model/dataset/prompt/parser revisions or immutable references sufficient to reconstruct the operation. Capture worker-side values when configuration crosses process boundaries.
- Persist raw responses before parsing, alongside input or immutable input references, completion metadata, timing, usage, classified errors, and retry decisions. Exclude credentials and sensitive headers; use access-controlled artifacts for payloads.
- Separate concise operational logs from replay artifacts. Lengths and short excerpts alone cannot reproduce a parser failure. If retention limits truncate or omit data, record that fact and its diagnostic consequence explicitly.
- Preserve authoritative attempts and original outcomes. Keep repaired results and interpretation separately identifiable; do not overwrite historical evidence to make a run look homogeneous.
- Decide which outputs recovery or regrading needs before automatic cleanup. Verify that retained artifacts actually contain those outputs.

For LLM-specific validation, logging, and failure examples, read [LLM boundaries and failure patterns](references/llm-boundaries-and-failures.md).

## Bound retries and lifecycle behavior

- Give retry policy an owner. Count SDK, transport, application, and whole-trial retries together; distinguish a retry count from a total-attempt count.
- Classify transient failures and permanent configuration/authentication failures. Bound total attempts, elapsed time, output tokens, and concurrent requests. Honor service retry guidance where applicable without exceeding the experiment's budget.
- Apply budget escalation or request repair only for an anticipated failure mode. Record changed payloads and preserve the original candidate. Never retry a valid negative judgment until a positive appears.
- Check whether concurrency limits are per worker or global; local semaphores can multiply load across processes. Account for queued work, cancellation, and provider outages.
- For resumable work, use stable item identities, explicit completed states, and durable writes appropriate to failure cost. Reject incompatible provenance, prevent duplicate result commitment, and distinguish pending/failed work from successful work.
- For long runs, expose durable progress and stalled work, retain a concrete cancellation path, and define escalation after repeated failures. Do not claim exactly-once external execution merely because local result writes are deduplicated.

## Validate and close the fix cycle

Choose tests for plausible, consequential failures rather than source structure or test-count targets.

- Start with cheap local checks of the boundary; then verify integration wiring and the effective runtime configuration. A successful HTTP response, import, or shell syntax check proves only that layer.
- Exercise representative data and the real consumer. When warranted, inject malformed/empty/truncated responses, retry exhaustion, provider failure, interruption, and resume. Verify both successful recovery and terminal handling.
- Use a small end-to-end canary matching the changed provider/runtime/payload/concurrency boundary before scaling. For stateful distributed changes, include the relevant lifecycle transition, not just startup.
- After a failure, preserve the initiating evidence, distinguish it from shutdown noise, build the smallest useful reproducer, fix narrowly, and validate recovery plus exhaustion. Add a regression case when it protects a distinct behavior.
- Small successful live samples establish compatibility, not rare-failure tolerance. Distinguish infrastructure readiness, scoring validity, and actual learning signal. Do not claim a patch is proven merely because it targets a plausible mechanism.

## Deliverable

Adapt detail to the task. Report the selected rigor level and why, critical contracts, failure/recovery policy, retained evidence, validation performed, and concrete unresolved blockers. Distinguish observed facts from hypotheses and untested paths. For implementation, also identify the changed files and any deliberate semantic change.

Keep a cheap probe lightweight. Reserve rigorous review for uncertainty that could corrupt results, propagate failures, or cause material waste.
