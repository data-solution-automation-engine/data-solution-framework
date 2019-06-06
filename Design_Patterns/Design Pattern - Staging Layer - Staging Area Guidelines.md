# Design Pattern - Staging Layer - Staging Area Guidelines

## Purpose


## Motivation
- 

## Applicability

## Structure

The following is a list of conventions for the Staging Area:

- When loading data from the source, always load the lowest level (grain) of data available. If a summary is required and even available as a source, load the detail data anyway
- If the ETL platform allows it, prefix the ‘area’, ‘folder’ or ‘namespace’ in the ETL platform with ‘100_’ because this is the first Layer in the architecture. Source definition folders, if applicable, are labelled ‘000_’. This forces most ETL tools to sort the folders in the way the architecture handles the data.
- Source to Staging Area ETL processes use the truncate/insert load strategy. When delta detection is handled by the DWH (i.e. using a Full Outer Join) a Landing Area table can be incorporated.
- Everything is copied as-is, no transformations are done other than formatting data types. The Staging Area processing may never lead to errors! 

## Implementation Guidelines

### Staging Area table structure

The Staging Area is modelled after the structure of the application that supplies the data. This excludes all constraints or referential integrity the source tables might have; the Data Warehouse assumes the source systems handle this, and otherwise this will be handled by the Integration / Presentation Layer processes of the Data Warehouse. 

The structure of the Staging Area therefore is the same as the source table, but always containing the following process attributes:

| Column Name              | **Required / Optional** | **Data Type / constraint**                            | **Reasoning**                                                | **DIRECT equivalent**     |
| ------------------------ | ----------------------- | ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------- |
| Load Date/Time Stamp     | Required                | High precision date/time   – not null                 | The date/time that the record has been presented to the Data Warehouse environment. |                           |
| Event Date/Time          | Required                | High precision date/time   – not null                 | The date/time the change occurred in the source system.      | Event Datetime            |
| Source System ID / Code  | Required                | Varchar(100) – not null                               | The code or ID for the source system that supplied the data. | Record Source             |
| Source Row ID            | Required                | Integer – not null                                    | Audit attribute that captures the row order within the data delta as provided by a unique ETL execution. The combination of the unique execution instance and the row ID will always relate back to a single record. Also used to distinguish order if the effective date/time itself is not unique for a given key (due to fast-changing data). | Source Row ID             |
| CDC Operation            | Required                | Varchar(100) – not null                               | Information derived or received by the ETL process to derive logical deletes. | CDC Operation             |
| Full row hash            | Optional                | Character(32), when using   MD5 – not null            | Using a checksum for record comparison requires storing a checksum value as an attribute. Can be   made optional if column-by-column comparison is implemented instead. | Hash Full Record          |
| ETL Process Execution ID | Required                | Integer – not null                                    | Logging which unique ETL process has inserted the record.    | Insert Module Instance ID |
| Upstream Hash Values     | Optional                | Character(32), when using   MD5 – not null            | Any pre-calculated hash values one may like to add to optimize upstream parallel loading (i.e. pre-hashing business keys), | Hash <>                   |
| <source attributes>      | Required                | According to data   type  conversion table - nullable | The source attributes as available. Note that if a primary hash key is not used the natural key (source primary key) needs to be set to NOT NULL. All other attributes are nullable. | N/A                       |


## Considerations and Consequences


## Related Patterns
