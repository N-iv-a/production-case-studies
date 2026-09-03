# LLM in production: claims liability estimation

## Problem

Estimating fault/liability in incident claims starting from unstructured free-text descriptions — the kind of task that's slow and inconsistent to do manually at volume, but risky to fully automate without guardrails.

## Approach

`ai_query` against Llama 3.3-70B, with a typed output schema enforced on the response (not a free-text answer parsed with regex afterwards) — the model is constrained to return fields that match a defined structure, which then gets parsed and validated before it's used downstream. Records that fail validation are flagged rather than silently accepted.

## Trade-offs

- **Cost and latency** of an LLM call per record, versus a rule-based or simpler-model approach — worth it here because the input is genuinely unstructured text that rule-based extraction handles poorly, but that's a real cost to weigh, not a free upgrade over simpler methods.
- **Non-conforming output.** Even with a typed schema, the model occasionally returns something that doesn't parse cleanly — handled with validation-and-flag rather than assuming every response is trustworthy.
- **When not to use an LLM.** For the parts of the pipeline where the input is already structured (e.g. numeric fields, known categories), a deterministic rule beats an LLM call on cost, latency, and predictability — the LLM is reserved for the genuinely unstructured part of the problem, not applied everywhere for consistency's sake.

## Outcome

[TO FILL IN]

## What I'd do differently

[TO FILL IN]
