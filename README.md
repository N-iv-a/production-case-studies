# Production Case Studies

Write-ups on data engineering problems I've worked on in the public transport / ticketing domain: production systems, the reasoning behind them, and the trade-offs.

**Scope note:** these are notes about problems and approaches, not case studies of a specific employer's product. 
No proprietary code, no real data (not even anonymized), no internal table or system names, no client names, no specific model names.
If it's not something I'd say out loud in an interview, it's not written here.

## Case studies

1. [Fare-evasion risk monitoring](case-studies/01-fare-evasion-monitoring.md) — Bayesian estimation under sparse, partial data
2. [Fuoriclasse: a driver performance data product](case-studies/02-fuoriclasse-driver-data-product.md) — one governed dataset for HR and Operations, with an LLM used for triage only
3. [Attributing onboard validations to the driver](case-studies/03-driver-attribution-validations.md) — joining two sources with no shared key, and keeping the nulls honest

Internal volumes and metrics are discussed in interviews, not published here.

---
