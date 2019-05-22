# Design Pattern - Dimensional Model - Time Dimension

## Purpose
This Design Pattern describes the positioning, structure and purpose of Data Warehouse reference information.

## Motivation
Data Warehouse solutions typically require a separate dataset to resolve common descriptions or codes (descriptive information). The reference data provides additional context for the information that is sourced from the various operational systems.  Reference data is typically a description of a code where the code itself is provided by the system but the description is not provided directly.
Also known as
User Managed data.
Sourcing text files.

## Applicability
This pattern is applicable for the Integration Layer only.

## Structure
The key design decision is to define whether the reference data is tracked for changes over time. In other words is it a Type 1 or Type 2 table?
Type 1 table
Implementation guidelines
Every separate source system has its own directory in the landing area.
Every source directory has an archive directory.

## Considerations and Consequences
The decision not to copy the data types from the file definitions but to check and explicitly convert these in the ETL process will mean that explicit checks and data type conversions will have to be added later.
Known uses
None.

## Related Patterns
Design Pattern 015 – Generic – Loading Staging Area tables.