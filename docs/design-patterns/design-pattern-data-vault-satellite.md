---
uid: design-pattern-data-vault-satellite
---

# Design Pattern - Data Vault - Satellite

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This Design Pattern describes how to represent, or load data into, Satellite tables using Data Vault methodology.

## Motivation

Satellite tables contain the descriptive, time-variant context for a Hub or Link. They carry the changing attributes and record the full history needed for auditability and point-in-time reconstruction.

## Applicability

This pattern is applicable for loading data to Satellite tables from:

* The Landing Area into the Integration Area.
* The Integration Area into the Interpretation Area.

Interpretation Area Satellites may add business logic, but follow the same structural approach.

## Structure

The data logistics process can be described as a slowly changing dimension / history update of all attributes except the business key (which is stored in the Hub table). Most attribute values, including some of the data logistics process control values are copied from the Landing Area table. This includes:
Load Date / Time Stamp (used for the target Effective Date / Time and potentially the Update Date / TimeE attributes).
Source Row Id.

## Implementation guidelines

* Multiple passes of the same source table or file are usually required: first to insert new keys in the Hub/Link, then to populate Satellites.
* Satellites are typically loaded with an insert-only SCD2 pattern: close the current record (set expiry), insert the new record with the new hash/checksum.
* Keep Satellite data logistics modular; separate insert and update branches if tool performance requires.
* Consider using checksums to detect attribute changes; avoid field-by-field comparisons where hashing is reliable.
* Maintain a dummy record per driving key to ensure complete timelines where required by downstream queries.

## Considerations and consequences

* Missing or duplicate effective/expiry dates lead to gaps/overlaps and incorrect time slicing; enforce timeline completeness.
* Excessive Satellite proliferation (too many small Satellites) can increase join complexity; balance attribute grouping with change frequency.
* Business logic in Interpretation Area Satellites should not alter raw history; keep transformations transparent and auditable.

## Related patterns

* [Design Pattern - Data Vault - Hub](xref:design-pattern-data-vault-hub).
* [Design Pattern - Data Vault - Link](xref:design-pattern-data-vault-link).
* [Design Pattern - Generic - Using checksums for row comparison](xref:design-pattern-generic-using-checksums).

If you have a Change Data Capture based source, the attribute comparison is not required because the source system supplies the information whether the record in the Landing Area is new, updated or deleted.

Use hash values to detect changes, instead of comparing attributes separately. The hash value is created from all attributes except the business key and data logistics process control values.

## Considerations and consequences

Multiple passes on source data are likely to be required.

## Related patterns

Design Pattern 006 - Generic - Using Start, Process and End Dates
Design Pattern 009 - Data Vault - Loading Satellite tables
Design Pattern 010 - Data Vault - Loading Link tables
