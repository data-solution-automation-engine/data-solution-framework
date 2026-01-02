---
uid: design-pattern-logical-core-business-concept
---

# Design Pattern - Core Business Concept

## Purpose

This design pattern defines a Core Business Concept (CBC) entity.

## Motivation

A CBC can help organizations to agree on terms and definitions. CBCs capture what is important in the business, how this is named, and how it is defined.

## Applicability

This pattern can be used during information/data modeling, including conducting workshops with business subject matter experts (SMEs).

## Structure

A Core Business Concept (CBC) represents everything that is important for the organization. These are the terms (words) used when people in the business are explaining what they do in the business, and what their process and responsibilities are.

These terms used can be identified and be described, so they are not metrics or descriptive information themselves.

Basically the Core Business Concepts are the words (verbs and nouns) used by the business they want to know more about.

Core Business Concepts can be categorized into:

* Events (including transactions).
* Person (people and organizations).
* Place.
* Thing (physical or Virtual).
* Other Concepts (everything that is important, but does not fit the other categories).

## Implementation guidelines

A Core Business Concept is typically implemented as a 'Hub' table using Data Vault methodology.

## Considerations and consequences

N/A

## Related patterns

* [Design Pattern - Data Vault - Hub](xref:design-pattern-data-vault-hub).
