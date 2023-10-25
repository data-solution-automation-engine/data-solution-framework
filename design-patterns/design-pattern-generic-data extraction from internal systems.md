# Design Pattern - Generic - Data extraction from internal systems

## Purpose
This Design Pattern describes the overarching concepts related to extracting data from internal systems.

## Motivation
- 

## Applicability


## Structure
* Data must be extracted from the sources that created the data (as opposed to using copied data as a source). This is a broader Data Governance principle
* Impacts on operational systems must be assessed and documented as part of the interface specification
* Source system extract processes should not include the Data Warehouse transformation, aggregation and consolidation rules, this is applied later (separation of concerns)
* Source system extracts should include control data to enable audit and reconciliation, e.g. record count, hash totals, etc. This is by default supported by the ETL process control model
* The standard data integration tool must be used to extract data from the source systems, unless another efficient data extract utility is provided as part of the application package (this may include using SQL for ETL)
* Implement incremental extracts where possible, as this is more scalable.

## Implementation Guidelines


## Considerations and Consequences


## Related Patterns
