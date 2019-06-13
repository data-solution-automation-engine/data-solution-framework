# Design Pattern - Generic - Initial Load and Reinitialisation

## Purpose
This design pattern describes the procedures for performing an Initial Load and any complete reloads (re-initialisations) after this.

## Motivation
Every Data Warehouse will see Initial Load(s) and Re-initialisations. If all goes well the Initial Load is a true initial load in the sense that it is done only once. Re-initialisations are more common and typically caused by progressing insight in the data model design (remodelling requirements) or troubleshooting. The History Area plays a large part in this, by defining an archive where source information is maintained to enable repopulation of the Data Warehouse.
Also known as
Full Load.
Regeneration.

## Applicability
This pattern concerns the full set of ETL and Data Warehouse architecture.

## Structure
The Initial Load is executed only once, and populates the entire Data Warehouse but most importantly the History Area with the complete set of available source data.
 Business Insights > Design Pattern 021 - Generic - Initial Load and Re-initialisation > BI10.png
Figure 1: Initial Load

Reloading (re-initialising) the DWH is done by copying all information from the HSTG into the STG. Source systems are not touched when re-initialising. This is shown in the following diagram:
Business Insights > Design Pattern 021 - Generic - Initial Load and Re-initialisation > BI11.png 
Figure 2: Re-initialisation

## Implementation Guidelines
A true Initial Load occurs only once to populate at least the History Area. To perform an Initial Load the Replicated Source or Full Source table (copy) is used to source the data. If replication is used, the replication agents must be stopped for this process.
For re-initialisation a single procedure or ETL process should be created to copy the initial load dataset from the History Area into the Staging Area.
At least the transactional tables must be truncated prior to a full re-initialisation to avoid creating duplicates.
For the initial load of non-CDC sources a proxy Load Date / Time Stamp must be defined.

## Considerations and Consequences
Running re-initialisations essentially discards and re-issues any Data Warehouse keys, which means that all related Cleansing Area and Presentation Layer datasets must be recalculated as well.
Known uses
An Initial Load has to be done at the end of the initial development stage for new information. Re-initialisations are typically part of bigger patches or troubleshooting. This also covers remodelling parts of the Integration Layer (the true Data Warehouse).
These are if-all-else-fails solutions.

## Related patterns
None.