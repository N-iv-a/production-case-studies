# Incremental ingestion framework

## Problem

Ingesting from a growing number of heterogeneous REST sources, each with its own schema — and schemas that change over time without warning. Writing a bespoke ingestion notebook per source doesn't scale past a handful of sources: every schema change becomes a notebook to hunt down and fix, and the same bugs (pagination, retries, type coercion) get re-solved slightly differently each time.

## Approach

A reusable library instead of per-source notebooks, built around two abstractions:

- **`TableSpec`** — declares what a table looks like: its schema, primary/incremental keys, and how to detect new or changed records for a given source.
- **`LayerAdapter`** — encapsulates how a given medallion layer (bronze/silver/gold) reads its input and writes its output, so the ingestion logic for a REST source doesn't need to know anything about Delta table mechanics, and vice versa.

On top of these: automatic schema evolution against Delta tables (new columns from the source get added rather than breaking the write), anti-join–based incremental loading (only new/changed records get processed, computed via anti-join against what's already landed rather than relying on source-side change-tracking that isn't always available), and type normalization across sources that represent the same concept differently (e.g. dates as strings vs. epoch millis vs. native timestamps).

## Why a library, not per-source notebooks

The recurring bugs were the same across sources — off-by-one pagination, a retry that doesn't back off, a schema check that silently drops a new column instead of adding it. Fixing those once in a shared library, with tests, beats fixing the same bug N times in N notebooks that have already diverged from each other by the time the second bug shows up.

## Outcome

[TO FILL IN — e.g. number of sources onboarded, time to add a new source before/after]

## What I'd do differently

[TO FILL IN]
