# Orchestration and deployment layer

## Problem

A set of independent deployment bundles (bronze → silver → gold → platinum pipelines) had cascading dependencies between them — bundle B needs bundle A's output to be fresh before it runs — with no central orchestrator to enforce that. Coordination was implicit and manual.

## Approach

A custom framework on top of Databricks Jobs, built around:

- **Dependency tables** describing which pipeline needs which upstream output, instead of dependencies encoded implicitly in run schedules.
- **Explicit state tracking** per pipeline run (pending / running / succeeded / failed / skipped), so "did this actually run successfully today" is a query, not a guess from log-diving.
- **Retries and notifications** driven by that state, not bolted onto each job individually.
- **Conditional tasks** — a downstream pipeline can check upstream state before deciding whether to run at all.

**Deployment:** Databricks Asset Bundles across multiple targets (dev / preprod / prod), externalized configuration variables per environment, and a lock mechanism preventing concurrent conflicting deploys to production.

**Governance:** Unity Catalog with role-based access control per group, and managed storage locations rather than ad-hoc paths per pipeline.

## Why not an off-the-shelf orchestrator (e.g. Airflow)

The honest trade-off: adopting a general-purpose orchestrator would have meant introducing a new piece of infrastructure to operate, secure, and teach to the team, for a dependency-management problem that was solvable within the primitives Databricks Jobs already had. What's lost by not adopting one: a broader ecosystem of operators/integrations, and a more standard mental model for anyone joining who already knows that tool. That's a real cost, not a free win — the custom approach fit the constraints at the time, not a universally better choice.

## Outcome

[TO FILL IN — e.g. number of bundles/pipelines coordinated, reduction in manual dependency-checking, incident reduction]

## What I'd do differently

[TO FILL IN]
