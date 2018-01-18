# Design Pattern - Generic - Fundamental ETL Requirements

## Purpose
The purpose of this Design Pattern is to define a set of minimal requirements every single ETL process (mapping, module or package) should conform to. These essential design guidelines impact how all ETL processes behave under the architecture of the ETL Framework.
Motivation
Regardless of (place in the) architecture or purpose every ETL process should be created to follow a distinct set of base rules. Essential concepts such as having ETL processes check if they have already run before inserting duplicates or corrupting data makes testing, maintenance and troubleshooting a more straightforward task. The ultimate motivation is to develop ETL which cannot cause errors due to unplanned or unwanted execution. Essentially, ETL must be able to be run and re-run at any point in time to support a fully flexible scheduling and implementation.
Also known as
ETL guidelines
Basic ETL architecture
Applicability
This Design Pattern applies to every ETL process.
Structure
The requirements are as follows:
ETL can always be rerun without the need to manually change settings. This manifests itself in many ways depending on the purpose of the process (i.e. its place in the overall architecture). An ETL process that truncates a target table to load form another table is the most straightforward example since this will invariably run successfully every time. Another example is the distribution of surrogate keys by checking if keys are already present before they are inserted, or pre-emptively perform checksum comparisons to manage history. This requirement is also be valid for Presentation Layer tables which merge mutations into an aggregate. Not only does this requirement make testing and maintenance easier, it also ensures that no data is corrupted when an ETL process is run by accident
Source data for any ETL can always be related to, or be recovered. This is covered by correctly using the metadata framework and concepts such as the History Area (HSTG). The OMD metadata framework covers the audit trail and its ability to follow data through the Data Warehouse while the History Area enables a new initial load in case of disaster
ETL processes exist in the directory/folder where they pull data to. For instance the ETL logic for ‘Staging to History’ exists in the ‘150_History_Area’ folder and loads data from the ‘100_Staging_Area’.
ETL processes detect whether they should insert records or not, i.e. should not fail on constraints. This is the design decision that ETL handles data integrity (and not the database)
The direction of data is always ‘up’. The typical process of data is from a source, to staging, integration and ultimately presentation. No regular ETL process should write data back to an earlier layer, or access information from previous layers (e.g. using a key lookup)
ETL lookup operations are always executed against information available in the same Layer as the ETL process itself. This way processes cannot ‘skip a step’ by, for instance, querying source data directly at a later stage. In other words; all information required for processing has to be available in each Layer.
ETL processes must be able to process multiple intervals (changes) in one run. This is an important criterion to allow ETL processes to be run at any point in time and to support real-time processing. It means that ETL processes should not only be able to load a single snapshot or change for a single business key, but to handle multiple changes in a single dataset.  For instance if the address of an employee changes multiple times during the day, and the ETL is run daily, all changes are still captured and correctly processed in a single execution of the ETL process. This requirement also prevents ETL to be run multiple times for ‘catch-up’ processing.
ETL processes should automatically recover /rollback when failed. This means that if an error has been detected the ETL automatically repairs the information from the erroneous run and inserts the correct data along with any new information that has been sourced.
ETL processes contain transformation logic for a single specific function or purpose. This design decision follow ETL best practises to create many ETL processes that each address a specific function as opposed to few ETL processes that perform a range of activities. Every ETL process attempts to execute an atomic functionality. Examples of atomic Data Warehouse processes (which therefore are implemented as separate ETL processes) are key distribution, detecting changes and inserting records and end-dating records.
Related to the above definition of designing ETL to suit atomic Data Warehouse processes every ETL process can read from one or more sources but only write to a single target.
Implementation guidelines
It is recommended to follow a ‘sortable’ folder structure to visibly order containers / folders where ETL processes are stored in a way that represents the flow of data. An example is as follows:
000_<source systems>, one for every source
100_Staging_Area
150_History_Area
200_Integration_Area
250_Interpretation_Area
300_Helper_Area
350_Datamart_Area
Consequences
In some situations specific properties of the ETL process may seem overkill or perhaps even redundant and this will have its impact on developing duration. But in regard of keeping the design generic (for generation and maintenance) this will still be necessary. Concessions are permitted per architectural layer (all mappings within a certain architecture step) but this has to be motivated in the ETL architecture documentation.
Known uses
All ETL processes designed and developed under the ETL Framework architecture.
Related patterns
In the various Design and Implementation Patterns where detailed ETL design for a specific task is documented the requirements in this pattern will be adhered to.
Discussion items (not yet to be implemented or used until final)
None.