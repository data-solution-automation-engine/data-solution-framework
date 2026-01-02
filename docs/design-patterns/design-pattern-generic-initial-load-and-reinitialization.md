---
uid: design-pattern-generic-initial-load-and-reinitialization
---

# Design Pattern - Generic - Initial Load and Reinitialisation

## Purpose
This design pattern describes the procedures for performing an Initial Load and any complete reloads (Re-initializations) after this.

## Motivation
Every Data Warehouse will see Initial Load(s) and Re-initializations. If all goes well the Initial Load is a true initial load in the sense that it is done only once. Re-initializations are more common and typically caused by progressing insight in the data model design (reModeling requirements) or troubleshooting. The Persistent Staging Area plays a large part in this, by defining an archive where source information is maintained to enable repopulation of the Data Warehouse.
Also known as
Full Load.
Regeneration.

## Applicability
This pattern concerns the full set of data logistics and Data Warehouse architecture.

## Structure
The Initial Load is executed only once, and populates the entire Data Warehouse but most importantly the Persistent Staging Area with the complete set of available source data.
 Business Insights > Design Pattern 021 - Generic - Initial Load and Re-initialization > BI10.png
Figure 1: Initial Load

Reloading (re-initialising) the DWH is done by copying all information from the PSA into the LND. Source systems are not touched when re-initialising. This is shown in the following diagram:
Business Insights > Design Pattern 021 - Generic - Initial Load and Re-initialization > BI11.png 
Figure 2: Re-initialization

## Implementation guidelines
A true Initial Load occurs only once to populate at least the Persistent Staging Area. To perform an Initial Load the Replicated Source or Full Source table (copy) is used to source the data. If replication is used, the replication agents must be stopped for this process.
For Re-initialization a single procedure or data logistics process should be created to copy the initial load dataset from the Persistent Staging Area into the Landing Area.
At least the transactional tables must be truncated prior to a full Re-initialization to avoid creating duplicates.
For the initial load of non-CDC sources a proxy Load Date / Time Stamp must be defined.

## Considerations and consequences
Running Re-initializations essentially discards and re-issues any Data Warehouse keys, which means that all related Cleansing Area and Presentation Layer datasets must be recalculated as well.
Known uses
An Initial Load has to be done at the end of the initial development stage for new information. Re-initializations are typically part of bigger patches or troubleshooting. This also covers reModeling parts of the Integration Layer (the true Data Warehouse).
These are if-all-else-fails solutions.

## Related patterns
None.


