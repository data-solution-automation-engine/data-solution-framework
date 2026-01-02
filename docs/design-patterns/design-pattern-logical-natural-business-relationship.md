---
uid: design-pattern-logical-natural-business-relationship
---

# Design Pattern - Natural Business Relationship

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This design pattern defines a Natural Business Relationship (NBR) entity.

## Motivation

A NBR assists in explaining how [Core Business Concepts](xref:design-pattern-logical-core-business-concept) interact together. This, in turn, clarifies how business processes are structured.

## Applicability

This pattern can be used during information/data modeling, including conducting workshops with business subject matter experts (SMEs).

## Structure

The Natural Business Relationship (NBR) describes how the Core Business Concepts (CBCs) are related to each other in a way the business is using them, and talking about them.

If there can be multiple different ways the same concepts can be related to each other it will result in multiple business relationships.

## Implementation guidelines

* A Natural Business Relationship is typically implemented as a 'Link' table using Data Vault methodology.
* When working with multiple relationships between the same concepts, it is not advisable to 'type' these e.g. by adding a relationship type or identifier. Instead, it is preferred to model this as separate relationships.

## Considerations and consequences

N/A

## Related patterns

* [Design Pattern - Data Vault - Link](xref:design-pattern-data-vault-link).
