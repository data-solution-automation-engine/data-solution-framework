---
uid: design-pattern-generic-loading-landing-area-tables-using-row-compacting
---

# Design Pattern - Generic - Loading Landing Area Tables Using Record Condensing

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This Design Pattern specifies how a data source that contains multiple changes for the same business key is processed. For instance when using 'net changes' within a Change Data Capture interval or when the source application supplies redundant records.

## Motivation

This process is optional for the Landing Area; its application depends on the nature of the data source itself. The reason to implement a 'condense' process in the Landing Area data logistics is to prevent implementing this logic in multiple locations when loading data out of the Landing Area (to the History and Integration Areas). During this process no information is lost; only redundant records are removed. These are records that are, in reality, no changes at all in the Data Warehouse context.

Also known as:

* Condensing Records.
* Net changes.

## Applicability

This pattern is only applicable for loading processes from source systems or files to the Landing Area. Only add this process when the data source shows this particular behavior or when a history of changes is loaded in one run (for example, catch-up loads).

## Structure

Depending on the nature of the source data, the following situation may occur. In this example these are the original records as they appear in the source system:

| Key | Value            | Event Date Time   |
|-----|------------------|-------------------|
| CHS | Cheese           | 2011-10-28 15:00  |
| CHS | Cheese - Yellow  | 2011-11-29 11:00  |
| CHS | Cheese - Gold    | 2011-11-29 13:00  |
| CHS | Cheese - Yellow  | 2011-11-29 17:00  |
| CHS | Cheese           | 2011-11-29 23:00  |

In this example a user changes the name of the product with key `CHS` multiple times in a single day and afterwards resets the value to the original value. If the data logistics interval is daily, only these two values are selected from the source (with the load timestamp stamp being the event timestamp):

| Key | Value  | Event Date Time   |
|-----|--------|-------------------|
| CHS | Cheese | 2011-10-28 15:00  |
| CHS | Cheese | 2011-11-29 23:00  |

This is a situation where the condensing process can be implemented so that the record will not be inserted into the Data Warehouse as a new record when no real change occurred.

## Implementation guidelines

* Implement the condensation step in the Landing Area data logistics to remove redundant records before they flow further downstream.
* Where possible, define the process as a reusable or generic data logistics component.
* Use this approach to process a source containing multiple intervals of data in a single run instead of replaying each interval.
* Skip condensation when all changes from a CDC source are already captured (i.e., no 'net change' compression upstream).
* Expect a performance overhead when processing larger deltas or when running an initial load.

## Considerations and consequences

* Condensing adds processing time; ensure load windows account for the extra step, especially on large deltas.
* When only net changes are available (for example, last change per day), condensation prevents false deltas from being propagated.
* Message-based sources treated as daily snapshots can show the same behavior and may benefit from the same approach.

## Related patterns

* [Design Pattern - Generic - Loading Landing Area tables](xref:design-pattern-staging-layer-landing-area).
* [Design Pattern - Generic - Using CDC](xref:design-pattern-generic-loading-landing-from-transactional-cdc).



