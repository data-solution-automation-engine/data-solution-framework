---
uid: design-pattern-staging-layer-landing-area
---

# Design Pattern - Staging Layer - Landing Area Guidelines

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This Design Pattern describes how to load data from source systems into the landing area of the staging layer.

## Motivation

A first step in data processing on the Data Platform process is bringing operational ('source') data into the data platform environment. In some cases, data needs to be 'staged' first for further processing. This may happen, for example, in the case of receiving full data copies or files.

The intent is to ensure a data delta (differential) can be derived for further processing into the Persistent Staging Area.  This pattern describes a consistent way to develop data logistics for this purpose.

Also known as:

* Data Staging.
* Source to Staging.

## Applicability

This pattern is only applicable for loading processes from source systems or files to the Landing Area (of the Staging Layer).

## Structure

The following is a list of conventions for the Landing Area:

* When loading data from the source, always load the lowest level (grain) of data available. If a summary is required and even available as a source, load the detail data anyway.
* If the data logistics platform allows it, prefix the ‘area’, ‘folder’ or ‘namespace’ in the data logistics platform with ‘100_’ because this is the first Layer in the architecture. Source definition folders, if applicable, are labelled ‘000_’. This forces most data logistics tools to sort the folders in the way the architecture handles the data.
* Source to Landing Area data logistics processes use the truncate/insert load strategy. When delta detection is handled by the DWH (i.e. using a Full Outer Join) a Landing Area table can be incorporated.
* Everything is copied as-is, no transformations are done other than formatting data types. The Landing Area processing may never lead to errors!
* The Landing Area is an optional area in the Data Platform architecture. If correct data deltas are received already these can be directly written to the Persistent Staging Area (PSA) and can therefore bypass the Landing Area.

However, it is possible to store data in the Landing Area even when a correct data delta is received. In this case, the delta will be overwritten once the next delta arrives (and the current delta has been successfully committed for further processing such as in the PSA).

The only reason to use the Landing Area is for pre-processing required to derive a data delta or when receiving data via flat files or equivalent.

No data transformations are done in the Landing Area.

The data logistics process can be described as a truncate/insert process which copies all source data. The process essentially copied all data into the Landing table *as-is*, while at the same time adding the operational metadata attributes as specified by the data logistics process control framework.

The key points of interest are:

* Defining the correct event timestamp as part of the data logistics process.
* Using a character string to indicate the record source.
* Perform data type streamlining:.
  * Text fields smaller or equal than 100 will be mapped to NVARCHAR (100).
  * Text fields > 100 but <= than 1000 will be mapped to NVARCHAR (1000).
  * The rest of the text attributes will be mapped to NVARCHAR (4000).
  * Dates, times, date times will be mapped to DATETIME2(7).
  * All decimals or numeric values will be mapped to NUMBER(38,20).

## Implementation guidelines

* Use a single data logistics process, module or mapping to load data from a single source system table in the corresponding Landing table.
* A control table, parameter or restartable sequence in a mapping can be used to generate the Source Row ID numbers.
* The data type conversion has many uses (as detailed in the Data Integration Framework Staging Layer document); most notably limiting the variety of data types in the Integration Layer and creating a buffer against changes in the source system.

### Landing Area table structure

The Landing Area is modelled after the structure of the application that supplies the data. This excludes all constraints or referential integrity the source tables might have; the Data Warehouse assumes the source systems handle this, and otherwise this will be handled by the Integration / Presentation Layer processes of the Data Warehouse.

The structure of the Landing Area therefore is the same as the source table, but always containing the following process attributes:

| Column Name              | **Required / Optional** | **Data Type / constraint**                            | **Reasoning**                                                | **DIRECT equivalent**     |
| ------------------------ | ----------------------- | ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------- |
| Load timestamp Stamp     | Required                | High precision timestamp   – not null                 | The timestamp that the record has been presented to the Data Warehouse environment. |                           |
| Event timestamp          | Required                | High precision timestamp   – not null                 | The timestamp the change occurred in the source system.      | Event Datetime            |
| Source System ID / Code  | Required                | Varchar(100) – not null                               | The code or ID for the source system that supplied the data. | Record Source             |
| Source Row ID            | Required                | Integer – not null                                    | Audit attribute that captures the row order within the data delta as provided by a unique data logistics execution. The combination of the unique execution instance and the row ID will always relate back to a single record. Also used to distinguish order if the effective timestamp itself is not unique for a given key (due to fast-changing data). | Source Row ID             |
| CDC Operation            | Required                | Varchar(100) – not null                               | Information derived or received by the data logistics process to derive logical deletes. | CDC Operation             |
| Full row hash            | Optional                | Character(32), when using   MD5 – not null            | Using a checksum for record comparison requires storing a checksum value as an attribute. Can be   made optional if column-by-column comparison is implemented instead. | Hash Full Record          |
| data logistics Process Execution ID | Required                | Integer – not null                                    | Logging which unique data logistics process has inserted the record.    | Audit Trail Id |
| Upstream Hash Values     | Optional                | Character(32), when using   MD5 – not null            | Any pre-calculated hash values one may like to add to optimize upstream parallel loading (i.e. pre-hashing business keys), | Hash <>                   |
| <source attributes>      | Required                | According to data   type  conversion table - nullable | The source attributes as available. Note that if a primary hash key is not used the natural key (source primary key) needs to be set to NOT NULL. All other attributes are nullable. | N/A                       |

## Considerations and consequences

* Resolving the Record Source ID will be done in the Integration Layer because disk space is less an issue in the Staging Layer..
* Adding key lookups in the Landing Area will also overcomplicate the data logistics design and negatively impact performance. Alternatively is can be discussed to hard-code the identifier instead of the Source System name (as the RECORD_SOURCE). This reduces the requirement for the key lookup but reduces visibility over the data.
* For Landing Area data logistics processes that use a CDC based source an extra step is added to control the CDC deltas (using a load window table). This is explained in the Using CDC Design Pattern and subsequent Implementation Patterns.

## Related patterns

* [Design Pattern 003 - Mapping requirements](xref:design-pattern-generic-data-integration-into-a-data-warehouse).
* [Design Pattern 006 - Using Start, Process and End dates](xref:design-pattern-generic-managing-multi-temporality).
* [Design Pattern 016 - Delta calculation](xref:design-pattern-generic-delta-calculations).
* [Design Pattern 021 - Using CDC](xref:design-pattern-generic-loading-landing-from-transactional-cdc).


