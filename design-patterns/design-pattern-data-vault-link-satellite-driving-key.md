---
uid: design-pattern-data-vault-link-satellite-driving-key
---

# Design Pattern - Data Vault - Link Satellite - Driving Key

> [!WARNING]
> This design pattern requires a major update to refresh the content.

> [!NOTE]
> Depending on your philosophy on Data Vault implementation, Link Satellites may not be relevant or applicable.
> This is especially applicable to the driving key mechanism.
> There are very viable considerations to implement a Data Vault model *without* Link-Satellites.

## Purpose

This design pattern describes how to represent cross-key end-dating for a Link-Satellite in Data Vault methodology. In Data Vault, Link-Satellite tables manage the change for relationships over time, but there might not be a good data-driven candidate to evaluate when certain relationships are valid.

## Motivation

To provide an approach for establishing validity for Link Satellites, when no context is available to define validity of the relationship in time.

## Applicability

This pattern is only applicable for loading data to Link-Satellite tables from:

* The Staging Area into the Integration Area.
* The Integration Area into the Interpretation Area.
* The only difference to the specified ETL template is any business logic required in the mappings towards the Interpretation Area tables.

## Structure

 Standard Link-Satellites use the Driving Key concept to manage the ending of �old� relationships.

## Implementation Guidelines

## Considerations and Consequences

## Related Patterns

* Design Pattern - Using Start, Process and End Dates
* Design Pattern - Satellite
* Design Pattern - Link
