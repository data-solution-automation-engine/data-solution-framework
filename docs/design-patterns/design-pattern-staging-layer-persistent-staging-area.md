# Design Pattern - Staging Layer - Persistent Staging Area

## Purpose

This Design Pattern describes how to process data from the Staging Area (STG) to the Persistent Staging Area (PSA) - also known as the Persistent Staging Area (PSA).

## Motivation

The History Area is a generic concept that serves multiple purposes in the Data Warehouse architecture. The structure is source specific (similar to the Staging Area) but the History Area is designed to track changes over time and contains additional attributes for this purpose. While in most cases the structure of the STG and HSTG areas are similar in terms of attributes (with some specific exceptions in constraints), this is not a requirement. There may be scenarios where some attributes are omitted, and the templates used should be able to support this.
The STG to HSTG process is in many ways a straightforward and consistent process that can be implemented without additional metadata - naming conventions are sufficient. The HSTG does not even need to be a relational database (RDBMS) - it can be a flat file or other type of storage. HSTG provides a raw archive of source data that was presented to the Data Warehouse environment and can be used for refactoring, auditing or 'replaying history' in upstream Data Warehouse layers.
Also known as
Archiving
Staging to History ETL
Persistent Staging Area (PSA)

## Applicability
This pattern is only applicable for loading processes from source systems or files to the History Area (via the Staging Layer) only.

## Structure

The following is a list of development conventions for the Persistent Staging Area (PSA):

- When loading the PSA from the Staging Area, always start a PSA ETL process as soon as the Staging Area is finished to ensure that there are no ‘gaps’ in the history. Since the Staging Area has the ‘truncate/insert’ load strategy, PSA data has to be processed before the next Staging Area run. During normal loads, the Integration Area has no dependency on the History Area and loading into history and integration can be done in parallel if the Data Warehouse has capacity for concurrent jobs. This is handled by the ‘Batch’ concept which guarantees the unit of work; e.g. making sure all data delta has been processed
- If the ETL platform supports it, prefix the ‘schema’, ‘area’ or ‘folder’ in the RDBMS and ETL software with ‘150_’. The PSA is part of the first layer in the architecture, but is updated after the Staging Area (if adopted). This forces most ETL tools to sort the folders in the way the architecture handles the data
- Everything is copied as-is, no transformations are done.
- There is no requirement for error handling in PSA ETL.
- When using files, changes in file formats (schema) over time can be handled in separate file mask metadata versions.



The structure of the HSTG tables is similar to the STG tables, with the exception of the following:
The Primary Key is a composite of:
The source natural key
The Load Date / Time Stamp (Insert Date/Time)
The Source Row ID
The attributes that are part of the key are not nullable (e.g. the source natural key columns)
An optional Current Record Indicator may be added
Some attribute may be removed from HSTG (non-standard)
The reason the Primary Key is composite is to track the changes to records over time as per the Load Date / Time stamp (the time recorded when the record is loaded into the Data Warehouse environment). In principle, only having the natural key and Load Date / Time is sufficient. But in some scenarios the speed of records being presented to the Data Warehouse is so fast that the Load Date / Time may not be unique (due to accuracy limitations of the high precision date/time data type), and due to this the Source Row ID is also added the key. This obviously relies on the interface (source to Staging) manages the order in which data arrives to ensure a deterministic process.
The key requirements for the HSTG template are to:
Load multiple changes in a single transaction
Store changes of record states over time
Prevent errors to occur when run individually (including out of order) and maintain dependency
To support these requirements, there are three main processes in place within the loading approach for the HSTG (pattern):
A safety check to prevent reloading of already loaded data. This is to avoid accidental reruns or out-of-order processing causes errors due to key violations, in combination with the requirement to load multiple changes in a single run.
A verification if the change provided is really a change. This is to allow the scope of attributes to change, for instance if a specific attribute is removed from the HSTG table (and still exists in STG). As part of the standard ETL requirements, the information provided is always compared against the target scope of attributes.
An ordering of changes (per key, over time) in which only the latest change is compared against the most recent change as available in HSTG. 

The above three components together satisfy the HSTG template requirements. The ETL process can be described as loading delta sets into the source historical archive.


## Implementation Guidelines
Use a single ETL process, module or mapping to load data from a single source system table in the corresponding History Area table.
The Load Date / Time stamp is the logical �effective date�, and is copied from the Staging Area table. The Staging Area handles the correct definition of the time a change has occurred.
Because of the differences between source interfaces, relying on the CDC Operation (i.e. Insert, Update or Delete) to detect change is not always possible. For this reason all History Area ETL processes need to contain a key lookup to compare values (detect changes).

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

## Consequences and Considerations
Loading processes towards the Integration Area can either be sourced from the Staging Area or the History Area depending on the scheduling requirements.

The History Area can be loaded in parallel with the Integration Area, or between the Staging Area and Integration Area.

The 'prevent reprocessing' functionality can also be implemented using the Event Date / Time attribute instead of the Load Date / Time attribute. 





## Related Patterns
- [Design Pattern - Generic - Managing temporality by using Load, Event and Change dates]()
- Implementation Pattern for SSIS - Loading Persistent Staging Area tables
- Implementation Pattern - Generic - Re-initialisation process