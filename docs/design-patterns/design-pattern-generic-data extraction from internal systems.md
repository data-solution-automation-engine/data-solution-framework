---
uid: design-pattern-generic-data-extraction-from-internal-systems
---

# Design Pattern - Generic - Data extraction from internal systems

> [!WARNING]
> This design pattern is a placeholder awaiting content

## Purpose

This Design Pattern describes the overarching concepts related to extracting data from internal systems.

## Motivation

Consistent, well-governed extracts reduce risk to operational systems, avoid hidden transformation logic outside the warehouse, and ensure repeatable, auditable data movement. Poorly designed extracts can overload source systems, hide business logic, or undermine reconciliation and lineage.

## Applicability

Applicable to all interfaces that pull data from internal/operational systems into the data platform (Staging, PSA, Integration).

## Structure

Key principles:

* Extract from the system of record, not downstream copies, to preserve lineage and correctness.
* Assess and document source-system impact (windows, locks, latency, bandwidth) in the interface spec.
* Keep extracts free of warehouse transformation/aggregation logic (separation of concerns).
* Include control totals (row counts, hashes, high-water marks) for audit and reconciliation.
* Prefer incremental/CDC-based extracts where supported; fall back to scoped full extracts only when justified.
* Use the standard integration tooling unless the packaged application provides a safe, supported utility.

## Implementation guidelines

* Define extraction windows that respect business SLAs and source maintenance schedules; avoid long locks.
* Use change markers (timestamps, version numbers, log-based CDC) to minimize data movement; keep a high-water mark per feed.
* Secure transport: encrypt in transit, restrict network paths, and avoid staging sensitive data in transient locations.
* Emit and store control totals alongside each extract; validate them on landing before downstream processing.
* Version and document extract queries or APIs; changes to source schemas must be reflected in the extract contract.
* For bulk loads, throttle or batch to protect source performance; coordinate with source owners for peak/blackout periods.

## Considerations and consequences

* Deep coupling to source internals (e.g., undocumented tables) increases maintenance risk; prefer stable interfaces or APIs.
* Incremental extracts reduce load but require robust watermark management and replay/reconciliation procedures.
* Pushing heavy logic into source extracts can obscure lineage and complicate troubleshooting; keep business logic downstream.

## Related patterns

* [Design Pattern - Staging Layer - Landing Area](xref:design-pattern-staging-layer-landing-area).
* [Design Pattern - Staging Layer - Persistent Staging Area](xref:design-pattern-staging-layer-persistent-staging-area).
* [Design Pattern - Generic - Managing temporality by using Load, Event and Change dates](xref:design-pattern-generic-managing-multi-temporality).
* [Design Pattern - Generic - Loading Landing Area Tables Using Record Condensing](xref:design-pattern-generic-loading-landing-area-tables-using-row-compacting).


