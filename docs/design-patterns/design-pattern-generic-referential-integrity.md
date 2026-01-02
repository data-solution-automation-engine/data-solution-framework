---
uid: design-pattern-generic-referential-integrity
---

# Design Pattern - Generic - Referential Integrity

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

Describe how referential integrity (RI) is enforced across Data Warehouse layers to maintain consistency and reliable joins.

## Motivation

Consistent RI keeps data trustworthy, prevents orphan records, and allows inner joins for performance. Explicit guidance is needed because parallel data logistics flows and differing layer requirements can erode RI if not managed deliberately.

## Applicability

Applies to all layers of the Data Warehouse, with specific handling per layer (Staging, PSA, Integration, Presentation).

## Structure

 Referential Integrity and constraints

The fundamental approach of the Data Warehouse Modeling is to enforce Referential Integrity (RI) on database level. This is a Data Warehouse best practice that allows the database to efficiently manage the consistency of the solution. Exceptions can be made where RI is temporarily disabled when certain data logisticss can be run in different order and/or parallel (especially in the case of a 3NF Integration Layer) but RI must be enabled after the processing to ensure integrity. Only for very large datasets (>250TB), or for very light hardware RI is disabled altogether. For this purpose all data logistics designs must take RI into account using placeholders and key distribution.  

The Data Vault 2.0 approach still requires enforcing of RI, however due to options of parallel loading the integrity cannot be implemented in data logistics. For this purpose a Batch level verification process needs to be executed, to make sure the RI is in order after data logistics processing. 

The Data Warehouse design implements various levels of predefined constraints and placeholder mechanisms to support this principle. The Operational Meta Data repository, as a component managed by the Data Warehouse team, is exempt from these conventions and is allowed a greater freedom in implementation options (data types, keys, constraints). 

Every Data Warehouse table contains a predefined set of metadata attributes, which are – with the exception of the Update process attributes – always set to NOT NULL. 

| **Layer    / area**           | **Constraint    / concept**                                  |
| ----------------------------- | ------------------------------------------------------------ |
| Landing Area (LND)            | All source attributes are nullable (NULL).                   |
| Persistent Staging Area (PSA) | All source attributes are nullable. PSA tables have a meaningless key   as Primary Key and a unique constraint on the combination of the source key   and the event timestamp. This means only one value can be valid at a point in   time. The source to staging interface design ensures that no duplicates can   ever occur by the (correct) assignment of this event timestamp. |
| Integration Layer (INT)       | Data Warehouse key tables will always have a -1 placeholder value to   server as the ‘unknown’ record.       Data Warehouse history tables will always have a complete time   interval. This means there is never a ‘gap’ or ‘island’ in the time intervals   and inner joins can always be used. This is implemented by insert a starting   record every time a new DWH key is created.        All record sets that are loaded to the Integration Layer support their   own ‘keying’ processes to reduce dependencies, but also to ensure the   referential integrity requirements are always met. This also means that the   system will always provide a correct view of the data when it was processed,   and how it improves over time. |
| Presentation Layer (PRES)     | Every Dimension will contain a -1 dummy record to link orphan records   to. This means all Fact records will have an inner join link to the   Dimensions.       Additionally, if transactions refer to business entities (Data   Warehouse keys) that have no match to other reference data when joining the   various entities into Dimensions (through the intersection / ‘link’ entities   in the Integration Layer) the upper levels are set to ‘Unknown’. No loss of   data is ensured in this process because the standard use of outer joins when   implementing business logic in Dimensions. No NULL values are allowed in the   Dimensions. |

## Implementation guidelines

* Prefer database-enforced RI constraints wherever feasible; if temporarily disabled for parallel loads, add a batch-level verification to reassert integrity.
* Use placeholder keys (-1 or similar) to avoid NULL foreign keys and to keep joins consistent.
* Ensure Integration and Presentation layers initialize dummy records for unknown/placeholder keys so facts always have a valid target.

## Considerations and consequences

* Temporarily disabling RI without post-load checks risks silent data loss or mismatched joins.
* Overusing placeholders can mask upstream data quality issues; balance between availability and accuracy.

## Related patterns

* [Design Pattern - Data Vault - Missing Keys and Placeholders](xref:design-pattern-data-vault-missing-keys-and-placeholders).
* [Design Pattern - Generic - Managing temporality by using Load, Event and Change dates](xref:design-pattern-generic-managing-multi-temporality).


