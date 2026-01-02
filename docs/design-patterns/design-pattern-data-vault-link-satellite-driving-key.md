---
uid: design-pattern-data-vault-link-satellite-driving-key
---

# Design Pattern - Data Vault - Link Satellite - Driving Key

> [!WARNING]
> This design pattern requires a major update to refresh the content.

> [!NOTE]
> Depending on your philosophy on Data Vault implementation, Link Satellites may not be relevant or applicable.
> There are very viable considerations to implement a Data Vault model *without* Link-Satellites.
> This is especially applicable to the driving key mechanism.

## Purpose

This design pattern describes how to represent cross-key end-dating for a Link-Satellite in Data Vault methodology.

In Data Vault, Link-Satellite tables are an option that can be used manage the context (and changes for) relationships over time. In many cases, the context includes attributes that can be used to evaluate if a relationship can be considered 'active', or not, at a point in time.

For example, if an attribute 'is valid' or equivalent exists, this data can be used to determine if a relationship is valid and can be used for downstream data delivery. However, depending on the systems and data that are available, there might not always be a good data-driven candidate to evaluate the validity of a relationship.

The 'driving key' mechanism is a special technique to apply validity of relationships when there are no reliable data points to do so.

## Motivation

To provide an approach for establishing validity for Link Satellites, when no context is available to define validity of the relationship in time.

## Applicability

This pattern is applicable for processing data for a Link-Satellite table, or its keyed-instance (relationship describing Hub) equivalent.

## Structure

Standard Link-Satellites use the Driving Key concept to manage the ending of old relationships.

## Implementation guidelines

To avoid data redundancy, it is recommended to manage this process into the target table as opposed to using end-dating.

## Considerations and consequences

## Related patterns

* [Design Pattern - Using Start, Process and End Dates](xref:design-pattern-generic-managing-multi-temporality).
* [Design Pattern - Satellite](xref:design-pattern-data-vault-satellite).
* [Design Pattern - Link](xref:design-pattern-data-vault-link).
