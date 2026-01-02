---
uid: design-pattern-generic-using-checksums
---

# Design Pattern - Generic - Using checksums for row comparison

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This Design Pattern aims to clarify and support the implementation of checksum values for record comparisons.

## Motivation

Checksums can provide a performance and maintenance efficient method for detecting changes. A fundamental design principle of the data logistics Framework is restartability and the general ability for data logistics to be run at any time. A key design decision to support this principle is the implementation of multiple change detection mechanisms to avoid duplicate inserts, for instance when data logistics is executed multiple times for the same source set.

Also known as:

* Delta detection.
* Hash-based change detection (CRC, hash bytes).

## Applicability

This pattern is applicable in any data logistics process that performs record comparisons against historical records. In some cases it may be required to calculate the checksum in the Landing Area as well, for instance as part of a Full Outer Join interface or Disaster Recovery for native CDC.

## Structure

The key aspect regarding the role of record comparisons (with or without using checksums to achieve this) is to be aware of the role the CDC_OPERATION plays.  
The possible scenarios as listed in the table below:
Process
Approach
Reasoning
Source to Staging (Full Outer Join or Disaster Recovery only)
Checksum is created and stored in the Landing Area table.
The checksum does not need to be recalculated from Staging to History as the full outer join process will always correctly identify the CDC operation.
In case of a Logical Delete, all attributes will retain their last known value (copied from the Persistent Staging Area table) thus the checksum will be calculated with these values.
Source to Staging (CDC, push or pull)
No checksums are required.
Other interfaces rely on load windows to select the delta sets.
Staging to History
If the checksum is available in the Landing Area the value can be copied into the Persistent Staging Area. Otherwise the checksum is calculated based on the source attributes.
Checksum values can be identical between inserts/updates and records that have been identified as a logical delete. Therefore record comparison must include the CDC operation.
If the checksums are different or the checksums are the same but the <CDC operation> is different; continue the SCD2 operation.
If the checksums are identical and the <CDC operation> is the same as well; discard (filter).
Staging to Integration
The checksum is always calculated within the Staging to Integration data logistics process based on the necessary attributes in the Landing Area table. This also requires the <CDC operation> to be part of the checksum attributes.
The comparison is executed based on the new checksum and the existing Integration Area checksum. As with the Staging to History process the <CDC operation> is evaluated to identify Logical Deletes, but in this scenario this happens as part of the checksum creation (i.e. source attributes and the CDC operation will be the new checksum).

## Implementation guidelines

* Decide where to compute checksums: at ingestion for change capture (FOJ, CDC recovery) and/or before integration for SCD2 detection.
* Include the CDC operation when comparing checksums to distinguish logical deletes from unchanged values.
* Normalize inputs (trimming, case handling, date formats, NULL placeholders) before hashing to avoid false deltas.
* Pick algorithms based on collision risk and storage: lightweight hashes for staging, stronger hashes (e.g., SHA) for integration if feasible.
* Fall back to attribute-by-attribute comparison if the platform lacks stable hashing or collision risk is unacceptable.
* Store checksums as persisted columns to speed comparison and aid troubleshooting.

## Considerations and consequences

* Hash collisions, while rare with strong algorithms, are possible; critical datasets may require periodic reconciliation beyond checksums.
* Poor input normalization can inflate change rates and load times; standardize before hashing.
* Overuse of wide hashes increases storage and bandwidth; balance precision and cost.

## Related patterns

* [Design Pattern - Data Vault - Loading Satellite tables](xref:design-pattern-data-vault-satellite).
* [Design Pattern - Generic - Handling Logical Deletes](xref:design-pattern-generic-handling-logical-deletes).
* [Design Pattern - Generic - Loading Landing Area tables](xref:design-pattern-staging-layer-landing-area).
* [Design Pattern - Staging Layer - Persistent Staging Area](xref:design-pattern-staging-layer-persistent-staging-area).
* [Design Pattern - Generic - Full Outer Join interfaces](xref:design-pattern-generic-full-outer-join-interfaces).


