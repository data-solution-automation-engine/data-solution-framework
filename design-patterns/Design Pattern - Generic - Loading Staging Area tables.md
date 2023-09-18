# Design Pattern - Generic - Loading Staging Area tables

## Purpose
This Design Pattern describes how to load data from source systems into the Staging Area.

## Motivation
The first step in the Data Warehouse process is bringing source data into the Data Warehouse environment. While no transformations to content are done during these processes the data is prepared for further processing. This pattern describes a consistent way to develop ETL for this purpose.
Also known as
Data Staging.
Source to Staging.

## Applicability
This pattern is only applicable for loading processes from source systems or files to the Staging Area (of the Staging Layer) only.

## Structure
The ETL process can be described as a truncate/insert process to copy all source data. This assumes the source applications have prepared their delta prior to this (see Design Pattern 016 – Delta Calculation). The process essentially copied all data into the Staging Area table as-is, while at the same time adding the operational metadata attributes.
The key points of interest are:
Defining the correct event date/time as part of the ETL process.
Using a character string to indicate the record source.
Perform data type streamlining:
Text fields smaller or equal than 100 will be mapped to VARCHAR (100).
Text fields > 100 but <= than 1000 will be mapped to VARCHAR (1000).
The rest of the text attributes will be mapped to VARCHAR (4000).
Dates, times, date times will be mapped to DATETIME.
All decimals or numeric values will be mapped to NUMBER.

Business Insights > Design Pattern 015 - Generic - Loading Staging Area Tables > BI3.png

Figure 1: Staging Area ETL process

## Implementation Guidelines
Use a single ETL process, module or mapping to load data from a single source system table in the corresponding Staging Area table.
A control table, parameter or restartable sequence in a mapping can be used to generate the Source Row ID numbers.
The data type conversion has many uses (as detailed in the A110 Staging Layer document); most notably limiting the variety of data types in the Integration Layer and creating a buffer against changes in the source system.

## Considerations and Consequences
Resolving the Record Source ID will be done in the Integration Layer because disk space is less an issue in the Staging Layer. Adding key lookups in the Staging Area will also overcomplicate the ETL design and negatively impact performance. Alternatively is can be discussed to hard-code the identifier instead of the Source System name (as the RECORD_SOURCE). This reduces the requirement for the key lookup but reduces visibility over the data.
For Staging Area ETL processes that use a CDC based source an extra step is added to control the CDC deltas (using the SOURCE_CONTROL table). This is explained in the ‘Using CDC’ Design Pattern and subsequent Implementation Patterns.
Known uses
This type of ETL process is to be used for all Staging Area tables.

## Related Patterns
Design Pattern 003 – Mapping requirements.
Design Pattern 006 – Using Start, Process and End dates.
Design Pattern 016 – Delta calculation.
Design Pattern 021 - Using CDC
Discussion items (not yet to be implemented or used until final)
Correct load window management combined with a Disaster Recovery function (typically a full outer join) can bypass the Staging Area in established environments. This means the Staging Area will become a logical area since the steps as described in the concept are still valid. It does replace the (requirement for) physical storage. 