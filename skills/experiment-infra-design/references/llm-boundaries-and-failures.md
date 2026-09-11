# LLM boundaries and failure patterns

Read when building or reviewing LLM judges, reward adapters, generation runners, or response parsers. The examples below describe failure mechanisms across experiment infrastructure. Apply those relevant to the component's contract; do not assume a particular provider, framework, dataset, or output format.

## Validate model output as external data

- Validate the response envelope and field types before accessing nested values. Handle empty/null content, missing choices, malformed bodies, ambiguous verdicts, and incomplete responses explicitly.
- Prefer structured output when the selected endpoint supports it and the exact request profile has been verified. JSON mode improves formatting reliability but does not establish schema compliance or semantic correctness.
- Keep final content and reasoning fields separate. Parse the designated final-answer field; do not recover a convenient verdict from reasoning text. Accept only documented, unambiguous format variants.
- Inspect completion status as well as parsed text. Do not accept a partial response merely because it contains a plausible verdict. Make any experiment-specific treatment of truncation explicit.
- Verify reasoning-token accounting and output headroom for the selected provider/model/mode. A tiny visible verdict does not imply a tiny completion budget. Changing reasoning mode invalidates assumptions tied to the previous profile.
- Treat candidate text as data, not judge instructions. Delimit it and test instruction-like candidates. Minimize unnecessary input while retaining the task context and evaluation criteria needed for the intended judgment.
- Separate protocol validity from judgment quality. Calibrate scoring on representative answer types with known labels, including false positives, false negatives, ambiguity, and deterministic-checker fallback where applicable. Test domain-specific tolerances, such as numerical rounding, when relevant. Repeat judgments when consistency matters.

## Retain each attempt

A useful attempt record includes:

- run, sample/group, logical-request and attempt IDs;
- input or immutable input reference and effective non-secret request payload;
- provider/endpoint/model and prompt/parser versions;
- raw returned body, with final and reasoning fields kept distinct if supplied;
- HTTP/completion status, provider request ID when available, latency and usage;
- parsed result or classified error, retry decision, and terminal consumer action.

Persist the response before interpreting it so parser failures do not erase their own evidence. Record all attempts, not just the successful one; failed attempts also consume time and tokens. Use explicit retention limits and access controls appropriate to the data. Console snippets can point to full artifacts without dumping large payloads.

## Failure patterns and design responses

| Failure pattern | Useful correction |
| --- | --- |
| A parser requires a binary verdict in one exact textual format; an unexpected response exhausts retries and aborts the entire run. | Define invalid/unresolved outcomes and test their effect on the actual consumer. Keep result validity strict while making recovery deliberate. |
| A failed judge response is discarded, leaving empty content, format noncompliance, and truncation indistinguishable. | Preserve raw response and completion metadata before parsing. Do not promote a likely cause into a confirmed diagnosis. |
| A recovery patch adds lengths and larger retry budgets but still discards bodies and raises after exhaustion. | Review the full failure path. Better retries do not by themselves provide replayability or failure containment. |
| Endpoint configuration exists in an adapter but does not reach distributed workers. | Verify effective configuration at the execution boundary, including provider-specific payload fields. |
| A small successful judge suite misses an intermittent failure under sustained use. | Test containment of rare failures, not just successful examples. A clean smoke run does not prove production reliability. |
| Result records collapse reasoning and final content, and the scorer accepts truncated responses. | Preserve channels, completion status, extraction provenance, and explicit scoring eligibility. Lost provenance cannot be reconstructed from aggregate scores. |
| Resume/checkpoint behavior allows ambiguous completion and duplicate records. | Separate append-only attempts from canonical results; validate identities, cardinality, compatible provenance, and completed-state semantics. |
| Calls involve retries at multiple layers; an individual successful request timing does not explain the logical call duration. | Correlate logical requests with physical attempts and worker lifecycle evidence before assigning timeout causes. |
| Cleanup removes task artifacts needed for later verifier investigation or regrading. | Identify and collect required outputs prospectively; report unrecoverable evidence gaps honestly. |

## UNKNOWN is a boundary, not a complete policy

A judge returning `UNKNOWN` avoids inventing a binary judgment, but its consumer still needs a defined response. An evaluation runner might retain an unresolved item for repair and report coverage under its declared denominator policy. An RL runner may need to retry the same judgment or pause the affected batch at a safe boundary.

Do not silently map `UNKNOWN` to zero, drop difficult samples, change group composition, regenerate until judgment succeeds, or swap judges. These actions can affect the result or training distribution. Define and record any such policy as part of the experiment rather than treating it as neutral infrastructure recovery.
