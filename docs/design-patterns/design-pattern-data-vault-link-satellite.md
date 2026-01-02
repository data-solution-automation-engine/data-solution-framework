---
uid: design-pattern-data-vault-link-satellite
---

# Design Pattern - Data Vault - Link Satellite

> [!WARNING]
> This design pattern requires a major update to refresh the content.

> [!NOTE]
> Depending on your philosophy on Data Vault implementation, Link Satellites may not be relevant or applicable.
> There are very viable considerations to implement a Data Vault model *without* Link-Satellites.

## Purpose

This design pattern describes how to load process data for a Data Vault methodology Link-Satellite. In Data Vault, Link-Satellite tables manage the change for relationships over time.

## Motivation

To provide a generic approach for loading Link Satellites.

## Applicability

This pattern is only applicable for loading data to Link-Satellite tables from:

* The Landing Area into the Integration Area.
* The Integration Area into the Interpretation Area.
* The only difference to the specified data logistics template is any business logic required in the mappings towards the Interpretation Area tables.

## Structure

Standard Link-Satellites use the Driving Key concept to manage the ending of old relationships. The driving key defines which key in the Link controls history (for example, the transaction id) so that related attributes expire correctly when that driving key changes.

## Implementation guidelines

* Identify the driving key for the Link; use it to manage effective/expiry dates in the Link-Satellite.
* Insert-only SCD2 pattern: close the current record (set expiry), insert a new record when the driving key/value combination changes.
* Carry metadata from the Link (load timestamps, source identifiers) to keep lineage intact.
* Use hash keys/checksums to detect attribute changes when applicable; avoid unnecessary updates.
* Keep Link-Satellites narrow—store only relationship attributes (e.g., status, type, reason) and not Hub-level attributes.

## Considerations and consequences

* Choosing the wrong driving key results in incorrect timelines; validate with business owners.
* Link-Satellites are optional; if relationship attributes are stable or modeled elsewhere, they may be unnecessary.
* Overuse of Link-Satellites can add joins and complexity; apply only when relationship history is required.

## Related patterns

* [Design Pattern - Using Start, Process and End Dates](xref:design-pattern-generic-managing-multi-temporality).
* [Design Pattern - Satellite](xref:design-pattern-data-vault-satellite).
* [Design Pattern - Link](xref:design-pattern-data-vault-link).
