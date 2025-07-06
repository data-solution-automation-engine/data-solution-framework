---
uid: design-pattern-data-vault-satellite
---

# Design Pattern - Data Vault - Satellite

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This Design Pattern describes how to represent, or load data into, Satellite tables using Data Vault methodology.

## Motivation

The Design Pattern for Satellite tables contain context, descriptive properties that describe a Data Vault 'Hub' table. They 

## Applicability

This pattern is only applicable for loading data to Satellite tables from:
The Staging Area into the Integration Area.
The Integration Area into the Interpretation Area.
The only difference to the specified ETL template is any business logic required in the mappings towards the Interpretation Area Satellite tables.

## Structure

The ETL process can be described as a slowly changing dimension / history update of all attributes except the business key (which is stored in the Hub table). This is explained in the following diagram. Most attribute values, including some of the ETL process control values are copied from the Staging Area table. This includes:
Load Date / Time Stamp (used for the target Effective Date / Time and potentially the Update Date / TimeE attributes).
Source Row Id.

## Implementation Guidelines

Multiple passes of the same source table or file are usually required. The first pass will insert new keys in the Hub table; the other passes are needed to populate the Satellite and Link tables.

The process in Figure 1 shows the entire ETL in one single process. For specific tools this way of developing ETL might be relatively inefficient. Therefore, the process can also be broken up into two separate mappings; one for inserts and one for updates. Logically the same actions will be executed, but physically two separate mappings can be used. This can be done in two ways:

Follow the same logic, with the same selects, but place filters for the update and insert branches. This leads to an extra pass on the source table, at the possible benefit of running the processes in parallel.

Only run the insert branch and automatically update the end dates based on the existing information in the Satellite. This process selects all records in the Satellite which have more than one open EXPIRY_DATE (this is the case after running the insert branch separately), sorts the records in order and uses the EFFECTIVE_DATE from the previous record to close the next one. This introduces a dependency between the insert and update branch, but will run faster. An extra benefit is that this also closes off any previous records that were left open.

A sample query for this selection is:

SELECT satellite.DWH_ID, satellite.<Expiry Date/Time>
FROM  satellite
WHERE  (            satellite.<Expiry Date/Time> IS NULL AND
                            2 <= (SELECT COUNT(DWH_ID)
                                     FROM satellite A WHERE a.DWH_ID = satellite.DWH_ID
                                  AND a.FIRM_LEDTS IS NULL) 
                 )
ORDER BY 1,2 DESC

If you have a Change Data Capture based source, the attribute comparison is not required because the source system supplies the information whether the record in the Staging Area is new, updated or deleted.

Use hash values to detect changes, instead of comparing attributes separately. The hash value is created from all attributes except the business key and ETL process control values.

## Considerations and Consequences

Multiple passes on source data are likely to be required.

## Related Patterns

Design Pattern 006 - Generic - Using Start, Process and End Dates
Design Pattern 009 - Data Vault - Loading Satellite tables
Design Pattern 010 - Data Vault - Loading Link tables
