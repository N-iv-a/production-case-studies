# Attributing onboard validations to the driver at the wheel

## Problem

Every contactless or ticket validation on a bus is an event with a card, a device and a timestamp. It does not say who was driving. Three things need that link: driver incentive schemes that reward onboard sales and validations, service quality monitoring, and, further down the line, payroll.

The driver is known from a different source: the onboard AVM system records which driver logged on which vehicle, on which trip, between which times. The two sources share no key, use different trip identifiers, and disagree in the edges: the same trip code can be run by two vehicles in parallel, and two drivers can swap at the wheel in the middle of a trip. On top of that, the AVM feed for a given day lands later than the validations for the same day.

The goal was one governed table, one row per validation, with the driver attached wherever the data allows and an honest null where it does not.

## Approach

### 1. A four condition key

A validation is attributed to the AVM service record that satisfies all four of:

* same trip code and same date;
* same vehicle;
* validation timestamp inside the real driving window recorded onboard, from departure to arrival.

Each condition earns its place. Trip and date alone are ambiguous when a trip is run by more than one vehicle; the vehicle resolves that. The vehicle alone is ambiguous when two drivers alternate on it; the real driving window resolves that. Scheduled times were rejected in favour of the times actually recorded onboard, because a late departure would otherwise push valid validations out of the window.

The AVM source itself needed work before it could serve as a key: the same service appears several times with different timestamps, so it is first collapsed to one record per trip, date and real departure time.

### 2. Verifying on the hardest case

The attribution was verified on the worst possible day: trips with a driver change at the wheel, restricted to trips certified as actually run. If the logic holds where two drivers share a vehicle within one trip, it holds everywhere else. On that perimeter the driver was attributed correctly in 98% of trips and 98.6% of individual validations; the only miss was a trip code run by two vehicles, which is exactly the case the vehicle condition closes.

### 3. Diagnosing the ones that do not match

Over the evaluation period, 95% of accepted check in validations got a driver. The remaining 5% were not left as a number: each null was classified by cause.

* validation outside the driving window, trip and vehicle correct (about a third; recoverable with a tolerance);
* trip not present in the AVM feed at all (about a third; only partly recoverable);
* vehicle mismatch, trip present but on a different vehicle (a quarter; recoverable through vehicle and window);
* trip code missing on the validation (marginal).

About two thirds of the residual is recoverable with a looser fallback. The rest is a genuine upstream gap: neither trip nor vehicle exists in the AVM feed, and no matching logic can invent it.

### 4. The latency problem

On some days almost every validation came out without a driver. That was not the matching logic. The final table was being written before the AVM feed for that day had landed, so every validation of that day was processed against an empty service list, written with a null driver, and never revisited.

Two consequences followed. First, the table is fully recomputed on every run rather than built incrementally: the cost is acceptable at this volume, and it guarantees that a late AVM feed corrects yesterday's nulls the next morning. Second, reporting excludes the current day, where the AVM feed is structurally late. A secondary driver field, populated by a looser temporal match, is used as a fallback in reporting so that recoverable records are not lost behind the strict key.

### 5. What reaches the final layer

Every validation is kept, including the ones without a driver. Dropping them would make the table look cleaner and would silently understate onboard activity on exactly the trips where the AVM feed is weakest. A null driver is information: it tells the business where the upstream data is missing.

Proposed next step: turn the strict key and the looser matches into a single cascade (trip, vehicle and window first; vehicle and window second; vehicle and nearest trip within a tolerance third), with a flag recording which level matched each row. Expected coverage close to 99%, with the residual being the true upstream gap.

## Trade offs

**Precision over coverage.** The strict four condition key covers 95% and does not produce wrong drivers. A looser key would cover more and would start attributing validations to the wrong person, which in an incentive scheme is worse than a null. The cascade with a method flag is the way to get both, and it was proposed rather than built because the business needed a defensible number first.

**Full recompute over incremental.** Incremental would be cheaper and is the reflex on a lakehouse. Here a late upstream feed makes yesterday's result wrong until it is recomputed; full recompute is the simplest way to be correct.

**Keep nulls over clean tables.** See above.

## Outcome

One table, one row per validation, with the driver attached in 95% of cases and the reason for the missing 5% understood. The same table is not a report input only: it feeds the driver performance data product (onboard sales and validations per driver) and is the planned source for payroll integration. Built once, consumed by three products.

## What I'd do differently

* Build the cascade with a method flag from the start instead of a strict column and a parallel fallback column.
* Make the dependency on the AVM feed explicit in the job: do not write the day until the upstream feed for that day has arrived, instead of relying on tomorrow's recompute.

## Attribution

The matching key, the verification perimeter and the field and key conventions were designed together with a colleague on the same team; the pipeline from raw to gold, the null diagnosis, the latency finding and its countermeasures, the recompute and null handling choices, the cascade proposal and the business documentation are mine. The decision to verify on trips with a driver change was taken by others on the team.

## Scope note

No absolute volumes, table names, line codes, dates or model names are published here. They are discussed in interviews.
