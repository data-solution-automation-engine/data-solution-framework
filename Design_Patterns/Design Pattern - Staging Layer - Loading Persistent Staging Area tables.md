# Design Pattern - Staging Layer - Loading Persistent Staging Area tables

## Purpose
This Design Pattern describes how to load data from the Staging Area (STG) to the History Staging Area (HSTG) - also known as the Persistent Staging Area (PSA).

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
The Load Date / Time stamp is the logical ‘effective date’, and is copied from the Staging Area table. The Staging Area handles the correct definition of the time a change has occurred.
Because of the differences between source interfaces, relying on the CDC Operation (i.e. Insert, Update or Delete) to detect change is not always possible. For this reason all History Area ETL processes need to contain a key lookup to compare values (detect changes).

## Consequences and Considerations
Loading processes towards the Integration Area can either be sourced from the Staging Area or the History Area depending on the scheduling requirements.

The History Area can be loaded in parallel with the Integration Area, or between the Staging Area and Integration Area.

The 'prevent reprocessing' functionality can also be implemented using the Event Date / Time attribute instead of the Load Date / Time attribute. 





## Related Patterns
- [Design Pattern - Generic - Managing temporality by using Load, Event and Change dates]()
- Implementation Pattern for SSIS - Loading Persistent Staging Area tables
- Implementation Pattern - Generic - Re-initialisation process