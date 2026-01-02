---
uid: design-pattern-data-vault-missing-keys-and-placeholders
---

# Design Pattern - Data Vault - Missing Keys and Placeholders

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This Design Pattern documents how to handle situations where there are mismatches with the source business keys leading to values not being available in some cases. Due to the strict approach towards key lookups this would lead to errors in data logistics. This is where placeholders are applied. The pattern assumes that source files are always first processed against Hub tables including loading any transactional tables against the Hubs.

## Motivation

This pattern focuses on processing data from dodgy sources that actually contain NULL business keys.  When a business key is NULL this should be resolved to a placeholder (dummy Surrogate Key).
The reasoning behind this is to prevent overcomplicated error handling while loading data into the (raw) Data Vault; supporting the goal to load everything just as the source system provides it while at the same time preventing losing any records.

Also known as:

* -1 placeholder value.
* Early or late arriving data.
* Empty business keys.

## Applicability

This pattern is only applicable for loading data into the Integration Area tables.

## Structure

The Enterprise Data Warehouse architecture specifies that hard business rules are implemented on the way into the Data Warehouse (the process from the Landing Area into the Integration Area) whereas soft business rules are implemented from the Integration Layer to the Interpretation Area and/or the Presentation Layer (on the way out).
Using placeholders is a hard business rule because no-one can interpret the meaning of a NULL value. SQL cannot deal with NULL values very well and because of this allowing NULL values increases the complexity of the queries against the Integration Area (potentially using outer joins). This is the reason why NULL values are remapped on the way into the Integration Area and ultimately why this kind of (hard) business logic is allowed here.

For example, here are some reasons how NULL values can be presented instead of business keys:

* The source declares them as optional Foreign Keys; for instance when X is true, then the business key is populated. Otherwise the business key remains NULL.
* The source declares them as required but the declaration is broken or not enforced (there is an error in the source application that allows NULLs when it shouldn't).

## Implementation guidelines

NULL/unknown/undefined business key values can be mapped to various placeholder surrogate key values (-1 to -7) with descriptions like Not Applicable or Unknown. A commonly used taxonomy (not all values are applicable in all situations):

* Missing (-1): root node and supertype of all missing information.
* Missing value (-2): supertype of all missing values. Can be Unknown or Not Applicable:.
  * Not Applicable (-3).
  * Unknown (-4).
* Missing Attribute/Column (-5): supertype of all missing values due to missing attributes:.
  * Missing Source Attribute (Non recordable Source) (-6) when the source fails to supply an attribute/column.
  * Missing Target Attribute (Non recordable DWH Attribute) (-7) for temporal data that falls before the deployment of the attribute.

Deciding between the various types of unknown is a business question based on how the source database works.

## Considerations and consequences

* Hubs must be pre-populated with the placeholder values (records).
* data logistics processes loading data into the Integration Area must automatically resolve NULL values to (potentially different) placeholders.
* Implementing a full taxonomy of potential unknown values as hard business rules must be weighed against the extra complexity while loading Integration Area tables.
* Ensure downstream consumers understand placeholder semantics; expose descriptions in metadata.

Known uses:

* This type of data logistics process is to be used in all Hub or Surrogate Key tables in the Integration Area. The Interpretation Area Hub tables, if used, have similar characteristics but the data logistics process contains business logic.

## Related patterns

* [Design Pattern 008 - Data Vault - Loading Hub tables](xref:design-pattern-data-vault-hub).
