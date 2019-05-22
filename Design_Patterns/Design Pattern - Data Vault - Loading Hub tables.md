# Design Pattern - Data Vault - Loading Hub tables

## Purpose
This Design Pattern describes how to load data into Data Vault Hub style entities.

## Motivation
Loading data into Hub tables is a relatively straightforward process with a set location in the architecture: it is applied when loading data from the Staging Layer to the Integration Layer. It is a vital component of the Data Warehouse architecture, making sure that Data Warehouse keys are distributed properly and at the right point in time. 

Decoupling key distribution and historical information is an essential requirement for reducing dependencies in the loading process and enabling flexible storage design in the Data Warehouse. 

This pattern specifies how the Hub ETL process works and why it is important to follow. 

In a Data Vault based Enterprise Data Warehouse solution, the Hub tables (and corresponding ETL) are the only places where Data Warehouse keys are distributed.

Also known as:

- Hub (Data Vault modelling concept)
- Surrogate Key (SK) or Hash Key (HSH) distribution
- Data Warehouse key distribution

## Applicability
This pattern is applicable for the process of loading from the Staging Layer into the Integration Area Hub tables. It is used in all Hub in the Integration Layer. Derived (Business Data Vault) Hub tables follow the same pattern, but with business logic applied.

## Structure
A Hub table contains the unique list of business key, and the corresponding Hub ETL process can be described as an ‘insert only’ of the unique business keys that are not yet in the the target Hub. 

The process performs a distinct selection on the business key attribute(s) in the Staging Area table and performs a key lookup to verify if the available business keys already exists in the target Hub table. If the business key already exists the row can be discarded, if not it can be inserted.

During the selection the key distribution approach is implemented to make sure a dedicated Data Warehouse key is created. This can be an integer value, a hash key (i.e. MD5 or SHA1) or a natural business key.

## Implementation Guidelines
Loading a Hub table from a specific Staging Layer table is a single, modular, ETL process. This is a requirement for flexibility in loading information as it enables full parallel processing. 

Multiple passes of the same source table or file are usually required for various tasks. The first pass will insert new keys in the Hub table; the other passes may be needed to populate the Satellite and Link tables.
The designated business key (usually the source natural key, but not always!) is the ONLY non-process or Data Warehouse related attribute in the Hub table.

The Load Date / Time Stamp (LDTS) is copied (inherited) from the Staging Layer. This improves ETL flexibility. The Staging Area ETL is designed to label every record which is processed by the same module with the correct date/time: the date/time the record has been loaded into the Data Warehouse environment (event date/time). The ETL process control framework will track when records have been loaded physically through the Insert Module Instance ID.

Multiple ETL processes may load the same business key into the corresponding Hub table if the business key exists in more than one table. This also means that ETL software must implement dynamic caching to avoid duplicate inserts when running similar processes in parallel.

By default the DISTINCT function is executed on database level to reserve resources for the ETL engine but this can be executed in ETL as well if this supports proper resource distribution (i.e. light database server but powerful ETL server).

The logic to create the initial (dummy) Satellite record can both be implemented as part of the Hub ETL process, as a separate ETL process which queries all keys that have no corresponding dummy or as part of the Satellite ETL process. This depends on the capabilities of the ETL software since not all are able to provide and reuse sequence generators or able to write to multiple targets in one process. The default and arguably most flexible way is to incorporate this concept as part of the Satellite ETL since it does not require rework when additional Satellites are associated with the Hub. This means that each Satellite ETL must perform a check if a dummy record exists before starting the standard process (and be able to roll back the dummy records if required).

When modeling the Hub tables try to be conservative when defining the business keys. Not every foreign key in the source indicates a business key and therefore a Hub table. A true business key is a concept that is known and used throughout the organisation (and systems) and is ‘self-standing’ and meaningful.

To cater for a situation where multiple OMD_INSERT_DATETIME values exist for a single business key, the minimum OMD_INSERT_DATETIME should be the value passed through with the HUB record. This can be implemented in ETL logic, or passed through to the database.  When implemented at a database level, instead of using a SELECT DISTINCT, using the MIN function with a GROUP BY the business key can achieve both a distinct selection, and minimum OMD_INSERT_DATETIME in one step.

## Considerations and Consequences
Multiple passes on the same Staging Layer data set are likely to be required: once for the Hub table(s) but also for any corresponding Link and Satellite tables. 

Defining Hub ETL processes as atomic modules, as defined in this Design Pattern, means that many Staging Layer tables load data to the same central Hub table. All processes will be very similar with the only difference being the mapping between the Staging Layer business key attribute and the target Hub business key counterpart.

## Related Patterns
Design Pattern 006 – Generic – Using Start, Process and End Dates
Design Pattern 009 – Data Vault – Loading Satellite tables
Design Pattern 010 – Data Vault – Loading Link tables
Design Pattern 023 – Data Vault – Missing keys and placeholders