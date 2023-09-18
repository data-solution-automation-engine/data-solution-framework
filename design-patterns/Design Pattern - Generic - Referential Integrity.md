# Design Pattern - Generic - Referential Integrity

## Purpose



## Motivation



## Applicability



## Structure

 Referential Integrity and constraints

The fundamental approach of the Data Warehouse modelling is to enforce Referential Integrity (RI) on database level. This is a Data Warehouse best practice that allows the database to efficiently manage the consistency of the solution. Exceptions can be made where RI is temporarily disabled when certain ETLs can be run in different order and/or parallel (especially in the case of a 3NF Integration Layer) but RI must be enabled after the processing to ensure integrity. Only for very large datasets (>250TB), or for very light hardware RI is disabled altogether. For this purpose all ETL designs must take RI into account using placeholders and key distribution.  

The Data Vault 2.0 approach still requires enforcing of RI, however due to options of parallel loading the integrity cannot be implemented in ETL. For this purpose a Batch level verification process needs to be executed, to make sure the RI is in order after ETL processing. 

The Data Warehouse design implements various levels of predefined constraints and placeholder mechanisms to support this principle. The Operational Meta Data repository, as a component managed by the Data Warehouse team, is exempt from these conventions and is allowed a greater freedom in implementation options (data types, keys, constraints). 

Every Data Warehouse table contains a predefined set of metadata attributes, which are – with the exception of the Update process attributes – always set to NOT NULL. 

| **Layer    / area**           | **Constraint    / concept**                                  |
| ----------------------------- | ------------------------------------------------------------ |
| Staging Area (STG)            | All source attributes are nullable (NULL).                   |
| Persistent Staging Area (PSA) | All source attributes are nullable. HSTG tables have a meaningless key   as Primary Key and a unique constraint on the combination of the source key   and the event date/time. This means only one value can be valid at a point in   time. The source to staging interface design ensures that no duplicates can   ever occur by the (correct) assignment of this event date/time. |
| Integration Layer (INT)       | Data Warehouse key tables will always have a -1 placeholder value to   server as the ‘unknown’ record.       Data Warehouse history tables will always have a complete time   interval. This means there is never a ‘gap’ or ‘island’ in the time intervals   and inner joins can always be used. This is implemented by insert a starting   record every time a new DWH key is created.        All record sets that are loaded to the Integration Layer support their   own ‘keying’ processes to reduce dependencies, but also to ensure the   referential integrity requirements are always met. This also means that the   system will always provide a correct view of the data when it was processed,   and how it improves over time. |
| Presentation Layer (PRES)     | Every Dimension will contain a -1 dummy record to link orphan records   to. This means all Fact records will have an inner join link to the   Dimensions.       Additionally, if transactions refer to business entities (Data   Warehouse keys) that have no match to other reference data when joining the   various entities into Dimensions (through the intersection / ‘link’ entities   in the Integration Layer) the upper levels are set to ‘Unknown’. No loss of   data is ensured in this process because the standard use of outer joins when   implementing business logic in Dimensions. No NULL values are allowed in the   Dimensions. |

## Implementation Guidelines



## Considerations and Consequences

TBD

## Related Patterns

N/A.