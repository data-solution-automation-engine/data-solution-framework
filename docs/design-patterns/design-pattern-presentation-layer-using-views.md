---
uid: design-pattern-presentation-layer-using-views
---

# Design Pattern - Presentation Layer - Using Views

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

Explain how and why views are used in the Presentation Layer to decouple consumers from table changes, support history delivery, and enforce security.

## Motivation

Views provide a stable contract to BI tools and users while underlying tables evolve. They enable history slicing without changing physical storage, and they simplify security by exposing only the required columns and rows.

## Applicability

Applicable to all Presentation Layer exposures (dimensional models, wide tables, extracts) where consumer-facing stability, security, or history control is required.

## Structure

### Using views for decoupling

The Data Warehouse design incorporates views on top of the Presentation Layer (Information Mart). This is applied for the following reasons:

- Views allow a flexible implementation of data access security (in addition to the security applied in the BI Layer).
- Views act as a decoupling mechanism between the physical table structure and the semantic layer (business model).
- Views allow for flexible delivery of history without changing the underlying storage.

These views are meant to be 1-to-1, meaning that they represent the physical table structure of the Information Mart. However, during development and upgrades these views can be altered to temporarily reduce the impact of changes in the table structure from the perspective of the BI platform. This way changes in the Information Mart can be made without the necessity to immediately change the semantic layer and/or reports. In this approach normal reporting can continue and the switch to the new structure can be done at a convenient moment.  

This is always meant as a temporary solution to mitigate the impact of these changes and the end state after the change should always include the return to the 1-to-1 relationship with the physical table.

A very specific use which includes the only allowed type of functionality to be implemented in the views is the way they deliver the historical information. Initially these views will be restricted to Type 1 information by adding the restriction of showing only the most recent state of the information (where the Expiry timestamp = `9999-12-31'). Over time however it will be possible to change these views to provide historical information if required. On a full Type 2 Information Mart, views can be used to deliver any type of history without changing the underlying data or applying business logic.

### Using views for virtualization

Another use case for views is virtualizing the Presentation Layer. As all granular and historic information is stored in the Integration Layer it is possible, if the infrastructure allows it, to use views to present information in any specific format. This removes the need for data logistics - physically moving data - from the solution design. Applicability of virtualization depends largely on the way the information is accessed and the infrastructure that is in place. Possible applications include when the BI platform uses the information to create cubes, when information is infrequently accessed, or with a smaller user base.

## Implementation guidelines

* Maintain a clear naming convention for logic views vs. consumer-facing views; keep schemas separated (for example, `ben` vs. `pres`).
* Enforce history filters in views (for example, `expiry_date = '9999-12-31'` for current) and provide alternate history views where needed (for example, `_history`).
* Use views to enforce row/column security where supported; avoid embedding business logic that belongs in integration or semantic layers.
* Treat views as part of change management: deploy new views before underlying table changes; deprecate temporary shims promptly.
* Document view contracts (columns, filters, history semantics) so BI layers can align.

## Considerations and consequences

* Heavy logic in views can degrade performance and complicate cost management; push complex transformations to integration where possible.
* Overreliance on temporary shims can cause semantic drift; timebox their use.
* Virtualization depends on infrastructure capacity; validate concurrency, caching, and latency before committing to a virtual-only approach.

## Related patterns

* [Solution Pattern - Data Modeling - Presentation Layer](xref:solution-pattern-data-modeling-presentation-layer).
* [Design Pattern - Generic - Types of History](xref:design-pattern-generic-types-of-history).
* [Design Pattern - Generic - Managing temporality by using Load, Event and Change dates](xref:design-pattern-generic-managing-multi-temporality).

