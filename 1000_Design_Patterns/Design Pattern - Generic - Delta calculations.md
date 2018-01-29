# Design Pattern - Generic - Delta Calculations

## Purpose
This design pattern describes how to load data into Hub or Surrogate Key style tables.

## Motivation
Loading data into Hub tables is a relatively straightforward process with a fixed location in the process of loading from the Staging Layer to the Integration Layer. It is a vital component of the Data Warehouse architecture, making sure that Data Warehouse keys are distributed properly and at the right point in time. This pattern specifies how this process works and why it is important to follow.
Also known as
Hub (Data Vault modelling concept)
Surrogate Key (SK) distribution.
Data Warehouse key distribution.

## Applicability
This pattern is only applicable for loading processes from the Staging Layer into the Integration Area (of the Integration Layer) only.

## Structure
The ETL process can be described as an ‘insert only’ set of the unique business keys. The process performs a SELECT DISTINCT on the staging area table and performs a key lookup (outer join) to verify if that specific business key already exists in the target Hub table. If it exists, the row can be discarded, if not it can be inserted. This is explained in the following diagram.
 


Figure 1: Hub ETL process
 
The SK ETL processes are the first ones that need to be executed in the Integration Area. Once the SK tables have been populated or updated, the related INT and RELN tables can be run in parallel. This is displayed in the following diagram:
 
Figure 2: Dependencies

## Implementation Guidelines
Use a single ETL process, module or mapping to load the Hub table, thus improving flexibility in processing. This means that no Hub keys will be distributed as part of another ETL process.
Multiple passes of the same source table or file are usually required. The first pass will insert new keys in the Hub table; the other passes are needed to populate the Satellite and Link tables.
The designated business key (usually the source natural key, but not always!) is the ONLY non-process or Data Warehouse related attribute in the Hub table.
Do not tag every record with the system date/time (sysdate), but copy this from the Staging Area. This improves ETL flexibility. The Staging Area ETL is designed to label every record which is processed by the same module with the same date/time: the date/time the record has been loaded into the Data Warehouse environment.
The Hub table only contains the business key as the non-Data Warehouse attribute, details are specified in the Integration Layer specification document.

## Consequences and Considerations
Multiple passes on source data are likely to be required.
Known uses
This type of ETL process is to be used in all Hub or SK tables in the Integration Area. The Cleansing Area Hub tables, if used, have similar characteristics but the ETL process contains business logic.

## Related Patterns
Design Pattern 006 – Using Start, Process and End Dates.
Design Pattern 009 – Loading Satellite tables.
Design Pattern 010 – Loading Link tables.