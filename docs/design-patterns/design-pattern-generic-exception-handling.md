---
uid: design-pattern-generic-exception-handling
---

# Design Pattern - Generic - Exception Handling

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

Define a consistent approach to detect, classify, and handle exceptions in data pipelines so availability, correctness, and auditability are balanced.

## Motivation

Exceptions arise from both environment issues (infrastructure, connectivity) and data issues (quality, business rule breaches). A common handling model avoids bespoke logic, reduces downtime, and keeps consumers informed about data fitness. Separating concerns (load first, transform where recoverable) limits blast radius when exceptions occur.

## Applicability

Applies to all data logistics/ELT processes across Landing, Integration, and Presentation layers. Choose the handling strategy per dataset/process based on business criticality and tolerance for imperfect data:

* **Report and continue**: tag, default, log, and load the record.
* **Report and reject**: divert to a reject store for recycling until quality thresholds are met.
* **Report and abort**: stop processing to prevent corruption or when critical reconciliation fails.

## Structure

### Strategy 1: report and continue

Mark the exception, set a default value, log details, and load the record. Use when completeness is more important than perfect correctness (e.g., missing FK, invalid date, out-of-range value).

### Strategy 2: report and reject

Tag the record, log details, and write it to a reject table for later recycling. Use when correctness is paramount (e.g., GL balancing, operational CRM data). Reject tables mirror target structure; records move to the target once quality is sufficient.

### Strategy 3: report and abort

Abort the load and require intervention. Use for environmental failures, unrecoverable corruption risk, or failed critical reconciliations. Still log the failure with enough detail for follow-up.

### Exception bitmaps

Use an exception bitmap to capture multiple issues on a row with a single value. Store the bitmap alongside the record (or in reject tables) and in the operational metadata layer. Bitmaps enable downstream reporting and automated recycling.

## Implementation guidelines

* Standardize exception categories and bit positions in the bitmap; keep a registry.
* Always log context: process id, source/target, keys, counts, checksum where available.
* For reject flows, provide a recycle process to reprocess fixed records; keep reject and target schemas aligned.
* Ensure restartability: retries must not duplicate or skip records; use control metadata.
* In Landing/Integration layers, avoid heavy business logic; focus on capturing and isolating exceptions quickly.
* Make Presentation-layer handling a business decision (e.g., mask, flag, exclude); document the chosen behavior.

## Considerations and consequences

* Overuse of aborts reduces availability; default to continue/reject unless corruption risk is high.
* Excessive rejection without recycling leaves data inaccessible; enforce SLAs for remediation.
* Bitmaps aid reporting but require governance to stay meaningful as checks evolve.
* Masking or defaulting values can hide data quality issues; ensure transparency via metadata and monitoring.

## Related patterns

* [Design Pattern - Generic - Managing temporality by using Load, Event and Change dates](xref:design-pattern-generic-managing-multi-temporality).
* [Design Pattern - Generic - Referential Integrity](xref:design-pattern-generic-referential-integrity).
* [Design Pattern - Generic - Control Framework](xref:design-pattern-generic-control-framework).

