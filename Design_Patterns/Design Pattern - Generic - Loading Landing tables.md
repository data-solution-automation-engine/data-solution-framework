# Design Pattern - Generic - Loading Landing tables

## Purpose
This Design Pattern describes how to load data from source systems into the Staging Area.

## Motivation
A first step in data processing on the Data Platform process is bringing operational ('source') data into the data platform environment. In some cases, data needs to be 'staged' first for further processing. This may happen, for example, in the case of receiving full data copies or files.

The intent is to ensure a data delta (differential) can be derived for further processing into the Persistent Staging Area.  This pattern describes a consistent way to develop ETL for this purpose.

Also known as:

* Data Staging
* Source to Staging

## Applicability
This pattern is only applicable for loading processes from source systems or files to the Landing Area (of the Staging Layer).

## Structure
The Landing Area is an optional area in the Data Platform architecture. If correct data deltas are received already these can be directly written to the Persistent Staging Area (PSA) and can therefore bypass the Landing Area. 

However, it is possible to store data in the Landing Area even when a correct data delta is received. In this case, the delta will be overwritten once the next delta arrives (and the current delta has been successfully committed for further processing such as in the PSA).

The only reason to use the Landing Area is for pre-processing required to derive a data delta or when receiving data via flat files or equivalent. 

No data transformations are done in the Landing Area. 

The ETL process can be described as a truncate/insert process which copies all source data. The process essentially copied all data into the Landing table *as-is*, while at the same time adding the operational metadata attributes as specified by the ETL process control framework.

The key points of interest are:

* Defining the correct event date/time as part of the ETL process
* Using a character string to indicate the record source
* Perform data type streamlining:
  * Text fields smaller or equal than 100 will be mapped to NVARCHAR (100)
  * Text fields > 100 but <= than 1000 will be mapped to NVARCHAR (1000)
  * The rest of the text attributes will be mapped to NVARCHAR (4000)
  * Dates, times, date times will be mapped to DATETIME2(7)
  * All decimals or numeric values will be mapped to NUMBER(38,20)

## Implementation Guidelines
* Use a single ETL process, module or mapping to load data from a single source system table in the corresponding Landing table.
* A control table, parameter or restartable sequence in a mapping can be used to generate the Source Row ID numbers.
* The data type conversion has many uses (as detailed in the Data Integration Framework Staging Layer document); most notably limiting the variety of data types in the Integration Layer and creating a buffer against changes in the source system.

## Considerations and Consequences
* Resolving the Record Source ID will be done in the Integration Layer because disk space is less an issue in the Staging Layer. 
* Adding key lookups in the Staging Area will also overcomplicate the ETL design and negatively impact performance. Alternatively is can be discussed to hard-code the identifier instead of the Source System name (as the RECORD_SOURCE). This reduces the requirement for the key lookup but reduces visibility over the data.
* For Staging Area ETL processes that use a CDC based source an extra step is added to control the CDC deltas (using a load window table). This is explained in the ‘Using CDC’ Design Pattern and subsequent Implementation Patterns.

## Related Patterns
* Design Pattern 003 – Mapping requirements
* Design Pattern 006 – Using Start, Process and End dates
* Design Pattern 016 – Delta calculation
* Design Pattern 021 - Using CDC