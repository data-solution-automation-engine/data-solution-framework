# Design Pattern - Interfacing - Using transaction log based Change Data Capture

## Purpose
This design pattern describes the purpose and configuration of Change Data Capture (CDC), the relationships it has with other ETL Framework concepts and the place in the overall architecture.
Motivation
Change Data Capture is the most flexible way to access source data (changes) for the Data Warehouse. Using CDC provides flexibility and is a future-proof approach to data integration. This is because many database support native CDC and various commercial software suites are available as well. CDC tracks every change to the source data and provides options to process the data as it was at a certain point in time (i.e. latest state at the end of the day if there are intra-day changes). This functionality enables the Data Warehouse to change the loading frequency over time, typically if a more near real-time update frequency is required at some point.
Also known as
Delta detection.
Interfacing.
Change Tracking (ambiguous).
Applicability
This pattern concerns the full scope of loading processes between source systems and (and in) the Staging Layer. There is a strong relationship between CDC and the Staging and History Areas.
Structure
In combination with Replication, CDC will capture any changes into the dedicated CDC tables. The Staging Area ETL processes select the delta and load this into the Staging Area tables for further processing. Either all changes or the ‘net’ changes (the most recent version of the record) which have changed in a set time interval. For instance if a record has changed five times on a day, the net change is the fifth change if the time interval is a day.
The following diagram shows the process of using CDC for loading delta sets into the Staging Area.
 Business Insights > Design Pattern 020 - Generic - Using CDC > BI9.png
Figure 1: Using CDC
 The key elements of this process are:
The Staging Area is sourced from the CDC table and uses the ETL process control CDC Control Table to determine what data to process. This is also to make sure that data is not accidentally removed after the Staging Area process has run successfully. As a first step for a CDC Staging Area ETL the maximum value that will be loaded is queried and stored in the ETL process framework CDC Control table. The Staging Area ETL can now select the delta based on (up to) this value and remove the corresponding dataset (up to this value) from the CDC table after it has run successfully.
Most default / native CDC solutions do not have Disaster Recovery. If for some reason CDC is disabled for a period of time or CDC details are accidentally lost there is no solution to recover the resulting ‘gap’ between the source and the Data Warehouse. If a value changes multiple times during the downtime these changes are always lost, but the Data Warehouse must be brought back in sync with the (most recent value of the) source. The CDC Catch-up Process makes sure that the most recent version of the source records are in line with the Data Warehouse information.
If this process finds discrepancies, changes must have occurred and this information is sent to the Staging Area as a missed delta set to be processed further. The CDC Catch-up Process works by comparing the most recent state of the source (in the Replicated Source table) against History Staging Area (HSTG).
Granularity of the delta selection from the CDC table can be controlled to some extent using built-in functions that interpret CDC information. A ‘Net Change’ is always for the period of time for loading (which can span multiple days) so an extra function is required to make sure that catch-up processes work correctly if the CDC data has not been picked up for some time(intervals). This is explained in the following example (for a day interval):  
Default
net changes
CDC event
Source Primary Key
Value Change controlled
net changes 




Day3, GYR, 75
Day 1: 10am
GYR
462	

Day1, GYR, 642

Day 1: 5pm
GYR
642
Day 2: 2pm
GYR
275	
Day2, GYR, 985
Day 2: 4pm
GYR
985
Day 3: 9am
GYR
235	
Day3, GYR, 75
Day 3: 11am
GYR
75

The example shows that if CDC information is not picked up for a couple of days, the net change for the primary key (‘GYR’ in this example) is only the most recent record by default: the latest one in the table. This regardless of the way the Data Warehouse is designed to be run; on a day interval (daily batch). When passing the ETL process control parameter (day in this example) the same dataset can be used to calculate the net change for the particular interval as defined for the Data Warehouse. These design decisions are in line with the Design Pattern to ensure that every process can be run at any time without breaking the process or corrupting data. Even if you do not run the daily processing for some time, starting it will deliver the same results in one go as they would have been Business As Usual.
Implementation guidelines
Use built-in functionality to access CDC table data.
Create a reusable procedure to calculate the Net Change per time interval based on an ETL process control parameter (stored in the PARAMETER table).
An alternative to the above mentioned reusable procedure is to run the Staging Area mapping multiple times (one for each time interval to catch-up).
In a CDC environment, the Staging Area delta selection process can only select a dataset up to the maximum previous date/time value of the defined interval. For example if the interval is set to day, only CDC records up to the end of the previous day can be selected. Omitting this results in additional CDC information being selected (or truncated) and linked incorrectly to the Load Date / Time stamp (which relates to the effective and expiry dates).
Consequences
This design decision focuses on establishing a scheduled (i.e. time-interval driven) delta selection based on CDC information. Although it is possible in theory to make dumps of the most granular level of CDC details, the approach taken here is to design an approach which makes it flexible to adjust the grain but at the same time doesn’t record the atomic CDC data. In other words, if at some point in time the grain is adjusted (from day to half-day for instance) the intra-day history will be built from that point onwards and cannot be initial loaded.
Known uses
This type of ETL process is to be used for all Staging Area tables where the source table has CDC enabled.
Related patterns
Design Pattern 006 – Generic – Using Start, Process and End dates.
Design Pattern 015 – Generic – Loading STG tables.
Design Pattern 021 – Generic – Initial Load and Re-initialisation.
Discussion items (not yet to be implemented or used until final)
None.

## Motivation



## Applicability



## Structure



## Implementation Guidelines



## Considerations and Consequences



## Related Patterns

- 