# Fuoriclasse: a driver performance data product for HR and Operations

## Problem

HR needed a way to evaluate and rank the whole driver population on measurable criteria, feeding the company incentive scheme. Operations area managers needed something different from the same data: a monitoring view per driver, updated daily, to spot patterns (repeated incidents, absences after an incident, behaviour during the probation period) before they became disciplinary cases.

The raw material was hostile. Attendance came from the HR payroll system with roughly 300 event codes accumulated over years, many overlapping or obsolete. Incidents came as free text notes written by workshop staff and by the driver. Disciplinary records lived in a static file maintained by hand. None of these sources agreed on identifiers, granularity or calendar conventions.

The goal was one governed dataset that both audiences could use without asking the data team for a new extract every time.

## Approach

### 1. Reduce the attendance codes to something a human can reason about

Together with HR, the ~300 payroll codes were collapsed to 65 and classified on two levels: a macro category (attendance, absence) and a category (sickness, injury, protected leave, presence). The mapping lives in a list owned by HR, not in code, so HR can add or reclassify a code without a release.

Correctness was proven with an annual reconciliation per employee: contractual working days must equal attendance plus absences plus rest days, with a delta of zero. The reconciliation was reviewed and signed off by HR before the dataset went live.

### 2. Define the calculation rules the source does not give you

The payroll export records events, not days. Turning events into a daily driver record required explicit rules, all of them documented and configurable:

* hours converted to days using the contractual daily hours;
* synthetic events for days that would otherwise be ambiguous: a presence event for days with only partial events, a rest event for days with nothing;
* half days counted as 0.5;
* partial absences above a threshold counted as a full absence day;
* sickness waiting days (the first days of a sickness episode, paid by the employer) computed per episode, with contiguous episodes merged up to the legal cap, and linked to the medical certificate table.

These rules are where most of the interpretation lives. Keeping them explicit, and separate from the code mapping, means a question like "why does this driver have 2.5 absence days in March" has a traceable answer.

### 3. Move data ownership to the people who own the process

Two sources were turned from static files into lists the business maintains directly, synchronised into the platform:

* the code classification, owned by HR;
* disciplinary records, fed by area managers and administration, with the disciplinary protocol number as a mandatory key so records can be joined and cannot be duplicated.

The trade off is accepted: a list is less controlled than a table with constraints, but it is maintained by the people who know the process, and the platform validates on ingestion rather than blocking users at entry.

### 4. Use an LLM for triage, never for the decision

Incident notes are unstructured text and the volume makes manual reading impractical. An open weights model served on the platform reads the workshop note and the driver's account of the dynamics and returns a typed JSON with three fields:

* estimated probability that the driver is liable, on a scale from 0 to 90. The value 100 is reserved for the formal insurance assessment, on purpose: the model is not allowed to close a case;
* damage magnitude, 0 to 9;
* a short justification.

Guardrails are part of the prompt and of the validation: zero when the driver is clearly not liable, low values when notes are thin, a hard cap when the damage is only inferred from the description rather than observed. Only new incidents are sent to the model (incremental anti join on the incident identifier). The raw response is stored for audit; results are appended, never overwritten.

The output is a flag for a human to look at. Every case that matters is reviewed by a person.

### 5. Deliver as a semantic model, not as tables

Everything lands in one Power BI semantic model that serves both audiences. The work that made it usable was unglamorous: relationships between six views (personal data, attendance, sickness and waiting days, service kilometres and hours, incidents with the model's assessment, disciplinary records), measures defined once, fields renamed in business language and organised in folders so that an HR analyst can find "absence days" without knowing which table it comes from.

The model refreshes automatically when the daily job completes. HR consumes it from a connected Excel workbook, area managers from a report, management from slides built on the same measures.

Rollout was incremental: hands on sessions with HR, a demo per area manager group, a prefilled Excel connected to the model, training on the new disciplinary process, and features released in packages rather than one big launch.

## Trade offs

**One semantic model for two audiences.** HR wants a ranking, Operations wants monitoring. A single model keeps one definition of every measure, at the cost of a model that is wider than either audience needs. The alternative, two models, would have drifted within months.

**Business owned lists as sources.** Faster to change, closer to the process, but not transactional. Validation on ingestion is the compensation, and it is not a complete one.

**An LLM on incidents only.** Absences, kilometres and disciplinary records are structured; a rule beats a model there on cost, latency and explainability. The model is used where the input is text and nowhere else.

**A capped probability.** Limiting the model to 90 loses nothing in practice and makes the division of responsibility between the tool and the insurance process explicit in the data itself.

## What I'd do differently

* Build a labelled evaluation set for the incident assessment before tuning the prompt, so that a change in guardrails can be measured instead of judged case by case.
* Version the calculation rules as tested code from the start. They were documented, but a rule change today is verified by rerunning the reconciliation, not by a unit test.

## Attribution

The data model, the code classification (with HR), the calculation rules, the ingestion and daily job, the LLM integration, the semantic model and the rollout are my work. The anomaly rule engine (configurable rules producing pending anomalies for review) and the notification layer were built on top of this model by an external partner, without changes to the model itself.

## Scope note

No internal volumes, user counts, table or list names, or the specific model used are published here. They are discussed in interviews.
