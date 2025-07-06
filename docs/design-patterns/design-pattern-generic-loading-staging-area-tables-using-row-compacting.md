# Design Pattern - Generic - Loading Staging Area tables using Record Condensing

## Purpose
This Design Pattern specifies how a data source that contains multiple changes for the same business key is processed. For instance when using ‘net changes’ within a Change Data Capture interval or when the source application supplies redundant records.
Motivation
This process is optional for the Staging Area; its application depends on the specific (nature of the) data source itself. The reason to implement a ‘condense’ process in the Staging Area ETL is to prevent implementing this logic in multiple locations when loading data out of the Staging Area (to the History and Integration Areas). During this process no information is lost, only redundant records are removed. These are records that are, in reality, no changes at all in the Data Warehouse context.
Also known as
Condensing Records
Net changes
Applicability
This pattern is only applicable for loading processes from source systems or files to the Staging Area (of the Staging Layer) only. Also, this process should only be added to the Staging Area ETL when the data source shows this particular behaviour or when a history of changes is loaded in one run (catch-up for instance).
Structure
Depending on the nature of the source data, the following situation may occur. In this example these are the original records as they appear in the source system:
Key
Value
Event Date Time
CHS
Cheese
28-10-2011 15:00
CHS
Cheese – Yellow
29-11-2011 11:00
CHS
Cheese – Gold
29-11-2011 13:00
CHS
Cheese – Yellow
29-11-2011 17:00
CHS
Cheese
29-11-2011 23:00

In this example a user has changed the name of the particular product with the key CHS multiple times in a single day and afterwards the value has been reset to the original value.
If the ETL interval is daily only these two values are selected from the source (with the Load Date / Time stamp being the ‘Event Date Time’).
Key
Value
Event Date Time
CHS
Cheese
28-10-2011 15:00
CHS
Cheese
29-11-2011 23:00

This is a situation where the condensing process can be implemented so that the record will not be inserted into the Data Warehouse as a new record (without there being a change).
The process to do this is as follows:


 Figure 1: Record condensing in STG
Implementation guidelines
The condensation process should be part of the Staging Area ETL process.
Depending on the available ETL software this process can be defined as a reusable or generic object.
This Design Pattern attempts to avoid ETL design where you have to run a source which contains multiple intervals (typically days) of data multiple times to correctly record the history. With this concept the entire history can be loaded in one run.
If all changes from a CDC source are captured this process is not required.
There is a performance overhead when processing larger deltas or when running an initial load.
Typically CDC sources where not all changes are processed but only the net changes for an interval. For instance when only the last change per day should be processed.
Message sources can have the same issue when treated the same way (only last record state per interval).
Design Pattern 015 – Generic – Loading Staging Area tables.
Design Pattern 021 – Generic – Using CDC.
Consequences
There is a performance overhead when processing larger deltas or when running an initial load.
Known uses
Typically CDC sources where not all changes are processed but only the net changes for an interval. For instance when only the last change per day should be processed.
Message sources can have the same issue when treated the same way (only last record state per interval).
Related patterns
Design Pattern 015 – Generic – Loading Staging Area tables.Design Pattern 015 - Generic - Loading Staging Area Tables
Design Pattern 021 – Generic – Using CDC.
Discussion items (not yet to be implemented or used until final)
None.

## Motivation



## Applicability



## Structure



## Implementation Guidelines



## Considerations and Consequences



## Related Patterns

- 