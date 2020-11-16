# Design Pattern - Generic - Fundamental ETL Requirements

## Purpose
The purpose of this Design Pattern is to define a set of minimal requirements every single data integration 'Extract Transform Load' - or ETL) process (i.e. mapping, module,  package, data pipeline) should conform to. These essential design guidelines direct how all ETL processes behave under the architecture of the Data Integration Framework.

## Motivation
Regardless of its place in the architecture or purpose, every data integration process should be created to follow a distinct set of base rules. Essential concepts such as making sure ETL processes check if they have already run before inserting duplicates or corrupting data will simplify testing, maintenance and troubleshooting. 

The overarching motivation is to develop data integration processes that cannot cause errors due to unplanned or unwanted execution. Essentially, data integration processes must be able to be run and re-run at any point in time to support a fully flexible scheduling and implementation.

## Applicability
This Design Pattern applies to every data integration process.

## Structure
The requirements are as follows:
* **Atomicity**; data integration processes contain transformation logic for a single specific function or purpose. This design decision focuses on creating many processes that each address a specific function, as opposed to few processes that perform a wide range of activities. Every data integration process attempts to execute an atomic functionality. Examples of atomic Data Warehouse processes are key distribution, detecting changes and inserting records.
* A data integration process can read from one or more sources but **only write to a single target**. This further supports enabling data integration processes to become atomic as outlined in the first bullet point.
* The direction of data loading and selection is **always related to the target area or layer**. In other words, it is always 'upstream'’. The typical processing of data is from a source to the Staging, Integration and ultimately Presentation layers. No regular data integration process should write data back to an earlier layer, or use access this information using a (key) lookup. This implies that the required information is always available in the same layer of the reference architecture.
* Data Integration processes **detect whether they should insert records or not**, i.e. should not fail on constraints. This is the design decision that data integration processes manage referential integrity and constraints (and not the RDBMS). A data integration process that truncates a target table to load form another table is the most straightforward example since this will invariably run successfully every time. Another example is the distribution of surrogate keys by checking if keys are already present before they are inserted, or pre-emptively perform checksum comparisons to manage history. This requirement is also be valid for Presentation Layer tables which merge mutations into an aggregate. Not only does this requirement make testing and maintenance easier, it also ensures that no data is corrupted when a process is run by accident.
* Data integration processes should be able to **rerun without the need to manually change settings**. This manifests itself in many ways depending on the purpose of the process (i.e. its place in the overall architecture).
* **Auditability**; source data for any data integration can always be related to, or be recovered. This is covered by correctly implementing an ETL control / metadata framework and concepts such as the Persistent Staging Area. A control framework metadata model covers the audit trail and its ability to follow data through the Data Warehouse while the Persistent Staging Area enables a new initial load in case of disaster or to reload (parts of) the data solution.
* Data integration processes must be able to **process multiple intervals** (changes) in one run. In other words, every time a data integration job runs it needs to process all data that it can (is available). This is an important requirement for data integration processes to be able to be run at any point in time and to support near real-time processing. It means that a process should not just be able to load a single snapshot or change for a single business key, but to correctly handle multiple changes in a single data set. For instance if the address of an employee changes multiple times during the day and the process is run daily, all changes are still captured and correctly processed in a single run of the process. This requirement also prevents processes to be run many times for catch-up processing and makes it possible to easily change loading frequencies.
* Data integration processes should **automatically recover /rollback when failed**. This means that if an error has been detected the process automatically repairs the information from the erroneous run and inserts the correct data along with any new information that has been sourced.

## Implementation guidelines
It is recommended to follow a ‘sortable’ folder structure to visibly order containers / folders where ETL processes are stored in a way that represents the flow of data. 

An example is as follows:
* 000_\<source systems\>, one for every source
* 100_Staging_Area
* 150_Persistent_Staging_Area
* 200_Integration_Layer
* 300_Presentation_Layer

ETL processes are recommended to be placed in the directory/folder where they pull data _to_. For instance the ETL logic for ‘Staging to History’ exists in the ‘150_History_Area’ folder and loads data from the ‘100_Staging_Area’.

## Consequences and considerations
In some situations adding some of the listed functionality to ETL processes may seem overkill or perhaps even redundant. This (perceived) additional effort will impact the developing duration. 

But in the context of maintaining a generic design (e.g. to support ETL generation and maintenance) this will still be necessary. Concessions can be made per architectural layer (all ETL processes within a certain architecture step). It is recommended to document any pattern exceptions in the Solution Architecture documentation.

## Related patterns
All ETL related patterns.
