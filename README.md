# Production Case Studies

Write-ups on data engineering problems I've worked on in the public transport / ticketing domain: production systems, the reasoning behind them, and the trade-offs.

**Scope note:** these are notes about problems and approaches, not case studies of a specific employer's product. No proprietary code, no real data (not even anonymized), no internal table or system names, no client names. If it's not something I'd say out loud in an interview, it's not written here.

## Case studies

1. [Fare-evasion risk monitoring](case-studies/01-fare-evasion-monitoring.md) — Bayesian estimation under sparse, partial data
2. [Orchestration and deployment layer](case-studies/02-orchestration-deployment.md) — dependency management across independent pipelines
3. [Incremental ingestion framework](case-studies/03-incremental-ingestion-framework.md) — a reusable library instead of per-source notebooks
4. [LLM in production](case-studies/04-llm-claims-liability.md) — structured extraction from unstructured text, and when not to use an LLM

## Numbers

Only figures I can stand behind if asked to elaborate — a case study with a fabricated number doesn't survive the first follow-up question.

| Metric | Value |
|---|---|
| Records processed / month (validations, sales, EMV) | [TO FILL IN] |
| Lines and stops covered | [TO FILL IN] |
| Pipelines/jobs in production | [TO FILL IN] |
| Tables managed per layer (bronze/silver/gold/platinum) | [TO FILL IN] |
| Pipeline refresh frequency | [TO FILL IN] |
| Dashboard users/recipients | [TO FILL IN] |
| Fare-evasion system: continuous operation period | [TO FILL IN] |

---

*Built with AI assistance (Claude); the technical decisions, trade-offs and opinions described are mine.*
