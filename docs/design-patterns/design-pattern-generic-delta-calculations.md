---
uid: design-pattern-generic-delta-calculations
---

# Design Pattern - Generic - Delta Calculations

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This design pattern describes how to calculate and process deltas efficiently when loading Hub or Surrogate Key style tables.

## Motivation

Loading data into Hub tables is a relatively straightforward process with a fixed location in the process of loading from the Staging Layer to the Integration Layer. It is a vital component of the Data Warehouse architecture, making sure that Data Warehouse keys are distributed properly and at the right point in time. This pattern specifies how this process works and why it is important to follow.

Also known as:

* Hub (Data Vault modeling concept).
* Surrogate Key (SK) distribution.
* Data Warehouse key distribution.

## Applicability

This pattern is only applicable for loading processes from the Staging Layer into the Integration Area (of the Integration Layer) only.

## Structure

The data logistics process can be described as an 'insert only' set of the unique business keys. The process performs a SELECT DISTINCT on the Landing Area table and performs a key lookup (outer join) to verify if that specific business key already exists in the target Hub table. If it exists, the row can be discarded, if not it can be inserted. Avoid repeated scans by isolating true deltas (new business keys) using change markers or high-water marks when available.

## Implementation guidelines

* Use a single data logistics process to load the Hub table to keep key distribution centralized.
* Identify deltas up front (distinct business keys not yet in Hub) to minimize scans; use staging high-water marks if available.
* Run Hub loads before dependent Satellite/Link loads; additional passes can then rely on the newly inserted keys.
* The designated business key (usually the source natural key) is the only non-process attribute in the Hub.
* Carry the staging Load timestamp into the Hub for consistent timing; avoid stamping with data logistics execution time.

## Considerations and consequences

* Multiple passes on source data may be required (Hub first, then Satellites/Links).
* Misidentifying deltas leads to duplicate keys or missed inserts; ensure business key uniqueness and reliable high-water marks.
* Hub loads are foundational; failure to load keys blocks downstream processing.

## Related patterns

* [Design Pattern 006 - Using Start, Process and End Dates](xref:design-pattern-generic-managing-multi-temporality).
* [Design Pattern 009 - Loading Satellite tables](xref:design-pattern-data-vault-satellite).
* [Design Pattern 010 - Loading Link tables](xref:design-pattern-data-vault-link).


