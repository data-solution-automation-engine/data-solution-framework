# Design Pattern - Staging Layer - Persistent Staging Guidelines

## Purpose


## Motivation
- 

## Applicability


## Structure
The following is a list of development conventions for the Persistent Staging Area (PSA):

- When loading the PSA from the Staging Area, always start a PSA ETL process as soon as the Staging Area is finished to ensure that there are no ‘gaps’ in the history. Since the Staging Area has the ‘truncate/insert’ load strategy, PSA data has to be processed before the next Staging Area run. During normal loads, the Integration Area has no dependency on the History Area and loading into history and integration can be done in parallel if the Data Warehouse has capacity for concurrent jobs. This is handled by the ‘Batch’ concept which guarantees the unit of work; e.g. making sure all data delta has been processed
- If the ETL platform supports it, prefix the ‘schema’, ‘area’ or ‘folder’ in the RDBMS and ETL software with ‘150_’. The PSA is part of the first layer in the architecture, but is updated after the Staging Area (if adopted). This forces most ETL tools to sort the folders in the way the architecture handles the data
- Everything is copied as-is, no transformations are done.
- There is no requirement for error handling in PSA ETL.
- When using files, changes in file formats (schema) over time can be handled in separate file mask metadata versions.

## Implementation Guidelines

The structure of the PSA is the same as the Staging Area (including the metadata attributes). The following attributes are mandatory for the PSA tables:

| **Column Name**                   | **Required / Optional** | **Data Type / constraint**                            | **Reasoning**                                                | **DIRECT equivalent**      |
| --------------------------------- | ----------------------- | ----------------------------------------------------- | ------------------------------------------------------------ | -------------------------- |
| Primary hash key i.e. <entity>_SK | Optional                | Character(32), when using   MD5 – not null            | The hashed value of the source (natural) key. Part of the primary key which is issued for each record in the history table. Can be used instead of composite primary key. | N/A                        |
| Effective Date/Time               | Required                | High precision date/time   – not null                 | The date/time that the record has been presented to the Data Warehouse environment. If a Staging Area is used these values will be inherited. | Insert Datetime            |
| Event Date/Time                   | Required                | High precision date/time   – not null                 | The date/time the change occurred in the source system. If a Staging Area is used these values will be   inherited. | Event Datetime             |
| Source System ID / Code           | Required                | Varchar(100) – not null                               | The code or ID for the source system that supplied the data. | Record Source              |
| Source Row ID                     | Required                | Integer – not null                                    | Audit attribute that captures the row order within the data delta as provided by a unique ETL execution. The combination of the unique execution instance and the row ID will always relate back to a single record. Also used to distinguish order if the effective date/time itself is not unique for a given key (due to fast-changing data). If a Staging Area is used these values will be inherited. | Source Row ID              |
| CDC Operation                     | Required                | Varchar(100) – not null                               | Information derived or received by the ETL process to derive logical deletes. If a Staging Area is used these values will be inherited. | CDC Operation              |
| Full row hash                     | Optional                | Character(32), when using   MD5 – not null            | Using a checksum for record comparison requires storing a checksum value as an attribute. Can be made optional if column-by-column comparison is implemented instead. If a Staging Area is used these values will be inherited. | Hash Full Record           |
| ETL Process Execution ID          | Required                | Integer – not null                                    | Logging which unique ETL process has inserted the record.    | Insert Module Instance ID  |
| Current Row Indicator             | Optional                | Varchar(100) – not null                               | A flag or Boolean to indicate if the record is the most current one (in relation of the effective date). | Current Record   Indicator |
| Change Date/Time                  | Optional                | High precision date/time   – nullable                 | A derived date/time field to standardize the main business effective date/time for more harmonised upstream processing. | Change Datetime            |
| <source attributes>               | Required                | According to data   type  conversion table - nullable | The source attributes as available. Note that if a primary hash key is not used the natural key (source primary key) needs to be set to NOT NULL. All other attributes are nullable. | N/A                        |

The ETL process from the Staging Area to the PSA checks the data based on the source key and the date/time information and compares all the attribute values. This can result in the following actions:

- No history record is found: insert a new record in the history table.
- A record is found and the source attribute values are different from the history attribute values: insert a new record. Update current row indicators if adopted.
- A record is found but there are no changes found in the attribute comparison: ignore.

Note: there are other suitable approaches towards a PSA. Depending on the requirements there can also be opted for a snapshot PSA where every run inserts a new record (instance) of the source data. This creates a more redundant dataset but arguably makes reloading data to the Integration Layer easier.

When loading data delta directly into the PSA (i.e. the Staging Area is not adopted) the same rules apply as for the Staging Area. 


## Considerations and Consequences


## Related Patterns
