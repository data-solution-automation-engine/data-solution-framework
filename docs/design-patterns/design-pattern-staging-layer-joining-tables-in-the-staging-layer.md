---
uid: design-pattern-staging-layer-joining-tables-in-the-staging-layer
---

# Design Pattern - Staging Layer - Joining tables in the Staging Layer

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This Design Pattern describes what to consider when joining Staging Layer tables (Staging and Persistent / History Staging)

## Motivation

Loading data into Hub tables is a relatively straightforward process with a fixed location in the scheduling of loading data from the Staging Layer to the Integration Layer. It is a vital component of the Data Warehouse architecture, making sure that Data Warehouse keys are distributed properly and at the right point in time. Decoupling key distribution and historical information is an essential requirement for parallel processing and for reducing dependencies in the loading process. This pattern specifies how this process works and why it is important to follow. In a Data Vault based Enterprise Data Warehouse solution, the Hub tables (and corresponding data logistics) are the only places where Data Warehouse keys are distributed.
Also known as
Hub (Data Vault Modeling concept)
Surrogate Key (SK) or Hash Key (HSH) distribution
Data Warehouse key distribution

## Applicability

This pattern is applicable for the process of loading from the Staging Layer into the Integration Area Hub tables only.

## Structure

The data logistics process can be described as an 'insert only' set of the unique business keys. The process performs a SELECT DISTINCT on the Landing Area table and a key lookup to retrieve the Record Source ID based on the value in the Staging Layer table.  If no entry for the record source is found the data logistics process is set to fail because this indicates a major error in the data logistics Framework configuration (i.e. this must be tested during unit and UAT testing).
Using this value and the source business key the process performs a key lookup (outer join) to verify if that specific business key already exists in the target Hub table (for that particular record source). If it exists, the row can be discarded, if not it can be inserted.
Business Insights > Design Pattern 008 - Data Vault - Loading Hub tables > image2015-4-29 14:54:58.png

Additionally, for every new Data Warehouse key a corresponding initial (dummy) Satellite record must be created to ensure complete timelines. Depending on the available technology this can be implemented as part of the Hub or Satellite Module but as each Hub can contain multiple Satellites it is recommended to be implemented in the Satellite process. This is explained in more detail in the Implementation guidelines and Consequences section.
The Hub data logistics processes are the first ones that need to be executed in the Integration Area. Once the Hub tables have been populated or updated, the related Satellite and Link tables can be run in parallel. This is displayed in the following diagram:
 Business Insights > Design Pattern 008 - Data Vault - Loading Hub tables > BI2.png
Figure 2: Dependencies
Logically the creation of the initial Satellite record is part of the data logistics process for Hub tables and is a prerequisite for further processing of the Satellites.

## Implementation guidelines

Use a single data logistics process, module or mapping to load the Hub table, thus improving flexibility in processing. This means that no Hub keys will be distributed as part of another data logistics process.
Multiple passes of the same source table or file are usually required for various tasks. The first pass will insert new keys in the Hub table; the other passes may be needed to populate the Satellite and Link tables.
The designated business key (usually the source natural key, but not always!) is the ONLY non-process or Data Warehouse related attribute in the Hub table.
Do not tag every record with the system timestamp (sysdate as Load Date / Time Stamp) but copy this value from the Landing Area. This improves data logistics flexibility. The Landing Area data logistics is designed to label every record which is processed by the same module with the correct timestamp: the timestamp the record has been loaded into the Data Warehouse environment (event timestamp). The data logistics process control framework will track when records have been loaded physically through the Audit Trail Id.
Multiple data logistics processes may load the same business key into the corresponding Hub table if the business key exists in more than one table. This also means that data logistics software must implement dynamic caching to avoid duplicate inserts when running similar processes in parallel.
By default the DISTINCT function is executed on database level to reserve resources for the data logistics engine but this can be executed in data logistics as well if this supports proper resource distribution (i.e. light database server but powerful data logistics server).
The logic to create the initial (dummy) Satellite record can both be implemented as part of the Hub data logistics process, as a separate data logistics process which queries all keys that have no corresponding dummy or as part of the Satellite data logistics process. This depends on the capabilities of the data logistics software since not all are able to provide and reuse sequence generators or able to write to multiple targets in one process. The default and arguably most flexible way is to incorporate this concept as part of the Satellite data logistics since it does not require rework when additional Satellites are associated with the Hub. This means that each Satellite data logistics must perform a check if a dummy record exists before starting the standard process (and be able to roll back the dummy records if required).
When modeling the Hub tables try to be conservative when defining the business keys. Not every foreign key in the source indicates a business key and therefore a Hub table. A true business key is a concept that is known and used throughout the Organization (and systems) and is self-standing and meaningful.
To cater for a situation where multiple Load Date / Time stamp values exist for a single business key, the minimum Load Date / Time stamp should be the value passed through with the HUB record. This can be implemented in data logistics logic, or passed through to the database.  When implemented at a database level, instead of using a SELECT DISTINCT, using the MIN function with a GROUP BY the business key can achieve both a distinct selection, and minimum Load Date / Time Stamp in one step.

## Considerations and consequences

N/A

## Related patterns

N/A





