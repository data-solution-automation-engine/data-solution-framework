# Design Pattern - Generic - Data integration into a Data Warehouse

## Purpose


## Motivation
- 

## Applicability


## Structure
*  Reconciliation and auditing must be built in all processes. By default this is managed by the ETL process control model.
* Change history to hierarchy, e.g. product hierarchy, must be captured (also known as slowly changing dimension)
* Optimise throughput without compromising data integrity
* Transformation process must include re-start and exception handling with no or minimum manual intervention. Do not rely on the database backup and recovery for process restart and retry. By default this is built in all ETL Framework templates.
* All exceptions and performance statistics must be saved in the database to enable reporting. By default this is handled by the ETL process control model.
* Do not over complicate processes more than necessary, e.g. cover scenarios that are unpractical or impossible, defects in preceding components, all possible future changes, etc. 
* Keep processes modular to gain leverage over failure and reusability.
* Performance benchmarks must be done as part of the development
* Leverage the RDBMS for data integrity enforcement (e.g. referential integrity, check constraints, validation, etc.). Be aware of the performance trade-offs and always ensure that RI is handled and verified by ETL as well
* Avoid using embedded freehand SQLs in ETL tools 
* Do not ‘over optimise’! Sometimes ‘good enough’ performance is acceptable. Be aware of the cost benefit ratio. Balancing performance against simplicity should by default fall in favour of simplicity
* Do not include functions that are not required, even if it ‘does not take long’. Be aware of additional costs in testing and maintenance
* Take an incremental development approach
* Development should allow more data and new types of data to be added without rework
* Ensure components are ‘built once reuse many times’ without creating additional dependencies unnecessarily
* Do not delete data from the data warehouse except by archive processes
* Design transformation and load processes so that processing schedule can change without any rework. E.g. the same processes can run daily, weekly or monthly, etc.
* Transaction data should be incrementally added to the Data Warehouse, even if the source system cannot provide incremental extracts
* Transaction data must not be physically deleted. Instead, flag the transaction as ‘delete’
* Transaction data should not be updated. E.g. should the data change, flag the old transaction as ‘deleted’ and create a new record
* All data in the Data Warehouse must be reconcilable to the sources

## Implementation Guidelines


## Considerations and Consequences


## Related Patterns
