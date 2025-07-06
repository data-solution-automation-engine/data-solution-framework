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

* The Staging Area into the Integration Area.
* The Integration Area into the Interpretation Area.
* The only difference to the specified ETL template is any business logic required in the mappings towards the Interpretation Area tables.

## Structure

 Standard Link-Satellites use the Driving Key concept to manage the ending of old relationships.

## Implementation guidelines

## CConsiderations and consequences

## Related Patterns

* Design Pattern - Using Start, Process and End Dates
* Design Pattern - Satellite
* Design Pattern - Link
