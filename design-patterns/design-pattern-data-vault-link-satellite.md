# Design Pattern - Data Vault - Loading Link Satellite tables

## Purpose
This Design Pattern describes how to load data into Link-Satellite tables within a ‘Data Vault’ EDW architecture. In Data Vault, Link-Satellite tables manage the change for relationships over time.

## Motivation

To provide a generic approach for loading Link Satellites.

## Applicability

This pattern is only applicable for loading data to Link-Satellite tables from:

* The Staging Area into the Integration Area.
* The Integration Area into the Interpretation Area.
* The only difference to the specified ETL template is any business logic required in the mappings towards the Interpretation Area tables.

## Structure
 Standard Link-Satellites use the Driving Key concept to manage the ending of ‘old’ relationships.

## Implementation Guidelines
Multiple passes of the same source table or file are usually required. The first pass will insert new keys in the Hub table; the other passes are needed to populate the Satellite and Link tables.
Select all records for the Link Satellite which have more than one open effective date / current record indicator but are not the most recent (because that record does not need to be closed
WITH MyCTE (<Link SK>, <Driving Key SK>, <Effective Date/Time>, <Expiry Date/Time>, RowVersion)
AS (
  SELECT
     A.<Link SK>, B.<Driving Key SK>, A.<Effective Date/Time>, A.<Expiry Date/Time>,
     DENSE_RANK() OVER(PARTITION BY B.<Driving Key SK> ORDER BY B.<Link SK>, <Effective Date/Time> ASC) RowVersion
  FROM <Link Sat table> A
  JOIN <Link table> B ON A.<Link SK>=B.<Link SK>
  JOIN (
    SELECT <Driving Key SK>
    FROM <Link Sat table> A
    JOIN <Link table> B ON A.<Link SK>=B.<Link SK>
    WHERE A.<Expiry Date/Time> = '99991231'
    GROUP BY <Driving Key SK>
    HAVING COUNT(*) > 1
  ) C ON B.<Driving Key SK> = C.<Driving Key SK>
)
SELECT
  BASE.<Link SK>
  ,CASE WHEN LAG.<Effective Date/Time> IS NULL THEN '19000101' ELSE BASE.<Effective Date/Time> END AS <Effective Date/Time>
  ,CASE WHEN LEAD.<Effective Date/Time> IS NULL THEN '99991231' ELSE LEAD.<Effective Date/Time> END AS <Expiry Date/Time>
  ,CASE WHEN LEAD.<Effective Date/Time> IS NULL THEN 'Y' ELSE 'N' END AS <Current Row Indicator>
FROM MyCTE BASE
LEFT JOIN MyCTE LEAD ON BASE.<Driving Key SK> = LEAD.<Driving Key SK>
  AND BASE.RowVersion = LEAD.RowVersion-1
LEFT JOIN MyCTE LAG ON BASE.<Driving Key SK> = LAG.<Driving Key SK>
  AND BASE.RowVersion = LAG.RowVersion+1
WHERE BASE.<Expiry Date/Time> = '99991231'

## Considerations and Consequences
Multiple passes on source data are likely to be required.

## Related Patterns
* Design Pattern 006 – Using Start, Process and End Dates
* Design Pattern 009 – Loading Satellite tables.
* Design Pattern 010 – Loading Link tables.