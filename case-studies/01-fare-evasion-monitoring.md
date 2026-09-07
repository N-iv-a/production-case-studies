# Fare-evasion risk monitoring

## Problem

Estimate fare-evasion risk per line and per stop, to prioritize where ticket inspection actually happens — with data that is sparse per individual line/stop cell, arrives with delay, and is only ever a partial sample of the real evasion rate (you only observe evasion when someone gets checked).

## Approach

A Bayesian Beta-Binomial update per line/stop cell, with:

- **Exponential decay** on older observations, so the estimate tracks recent behavior instead of being dominated by history.
- **Multi-level fallback**: a cell with too few observations borrows strength from a broader aggregate (e.g. the line, then the network) instead of returning a wild, low-confidence estimate from three data points.
- **Verified invariants** on the pipeline output (e.g. posterior parameters stay positive, aggregated risk stays within a sane range) — a monitoring system that silently produces nonsense on bad input is worse than one that fails loudly.

## Why Bayesian, not a classifier

Three reasons this wasn't a supervised classification problem:

- **Sparse data per cell.** Many line/stop combinations have very few historical checks — a discriminative classifier needs volume a Beta-Binomial doesn't.
- **Uncertainty has to be a first-class output**, not an afterthought. The people deciding where to send inspectors need to know when the model is guessing versus when it's confident — a point estimate from a classifier doesn't say that on its own.
- **Interpretability for the decision-maker.** A Beta posterior (mean + credible interval) is something an operations manager can reason about directly. A black-box score is harder to trust for a resourcing decision.

## Trade-offs

- **Batch vs. real-time.** Running as a batch job was the right call here: fare-evasion risk shifts over weeks, not minutes, so there was no case for the operational complexity of a streaming pipeline.
- **Granularity.** Line/stop was chosen over finer grain (e.g. line/stop/time-band) because the data gets too sparse below that level for the Bayesian update to say anything useful — a deliberate ceiling on resolution, not a limitation nobody noticed.
- **Cells with zero observations.** Handled by the fallback hierarchy rather than by imputing a network-wide average directly — this keeps the "we don't really know yet" signal visible instead of hiding it behind a plausible-looking number.

## Outcome

In production, used for real revenue-protection decisions (where to allocate inspection effort). Recognized with a Bronze Award at TTG 2026.

Deployment was not just technical. Adopting the tool required changes in how inspection records were filled in and how inspector resources were allocated, with the control team progressively working closer to what the system suggested. This meant more travel time between assignments, but higher effectiveness at the point of inspection.

Over the period following deployment, sanctions per inspection rose from 0.50 to 0.56 while inspection volume dropped by 19% and inspected passengers by 25%. The overall sanction rate moved from 3.06% to 3.75%. Notably, the rate measured by ticket inspectors declined over the same period, consistent with a deterrence effect on the lines where activity was concentrated.

This is an observational before/after comparison without a control group, and the operational changes came bundled with the tool, so the two cannot be separated. What I can say is that the direction of every indicator is consistent with better targeting rather than more enforcement.

## What I'd do differently

The binding constraint is the input data, not the model. Given the data available, the current version is close to what this approach can deliver: some parameters would benefit from further tuning, but the remaining gains sit upstream, not in the modelling.

Passenger counting reliability. Automatic passenger counting is the weakest input. When those counts drift, the model becomes unpredictable, so we constrain it with thresholds and quality checks. That protects the output but also discards a significant share of otherwise usable observations: the system trades coverage for stability. A more reliable counting source would change the design, not just the accuracy.

Onboard validation is structurally incomplete. We built checks on tickets and passes validated onboard, but validation is not mandatory for passengers, so the absence of a validation does not imply the absence of a valid ticket. The signal is directionally useful but weak as evidence, which limits how confidently evasion can be attributed.

Blind spot on app-sold fares. Tickets purchased through the mobile app are not yet ingested into the data platform, so they sit outside the model entirely. This is a data availability gap rather than a modelling choice, and closing it is the single highest-value improvement available.

Measurement design. With hindsight, I would have pushed for a staggered rollout by line, so that lines adopting the system could be compared against lines that had not yet adopted it in the same period. Without that, the effect of the model and the effect of the accompanying process change cannot be separated.

The general lesson: the modelling was the tractable part. What limited the outcome was data coverage, and the fact that some signals are structurally noisy because of how the service operates, not because of how it is measured.