# Design Pattern - Dimensional Model - Loading a Dimension from the Persistent Staging Area

## Purpose
This Design Pattern is defined to support a 2-tiered (Staging and Presentation Layer only) Data Warehouse architecture. In most scenarios this type of solution depends on loading processes that select information from the History Area to update the Presentation Layer. The History Area provides a ‘safety catch’ that records all transactions and changes that pass through the Data Warehouse and amongst other things provides the functionality to rebuild the Presentation Layer Star- and/or Snowflake models.

## Motivation
There are various advantages using the History Area as the source table for the Presentation Layer updates in a 2-tiered approach:
This design decouples delta gathering and the subsequent Data Mart ETL processes. This means delta processes can run at a different frequency than the Presentation Layer ETL Processes as the History Area always provides the most recent information (i.e. you can schedule the data staging to run more often and independent of the Data Mart updates.).
Only one source (the HSTG table) is required for all loading types (initial load, re-initialisation and regular runs). ETL can be designed in such a way that the history of changes can be merged with target Dimension thus only a single template is required. This approach uses load windows to only update the Data Mart with a limited set of (changed) records.
This approach allows the Data Warehouse team to truncate and reload the Presentation Layer in order to recalculate it completely. For instance when changing an attribute from Type 1 to 2 and supporting a history backload, changing the grain of the Fact table or adding a new Dimension level.
 Also known as
Data Staging.
Dimensional Loading.

## Applicability
This pattern is applicable for loading the Presentation Layer in a 2-tiered Data Warehouse architecture. While not directly applicable to the loading processes in a 3-tiered architecture the logic can be reused to some extend as the provided SQL applies to any set of historical records.

## Structure
The type of logic that is required to populate a Dimension based on two or more historical sources of information is very similar to the approach of loading a Dimension from the Integration Layer as documented in Design Pattern 019. However, the History Area does not have additional features related to the Integration Layer such as (but not limited to):
Dummy record handling.
Placeholders / unknown value taxonomy.
Expiry dates.
For this reason the logic is slightly more complex. Creating Dimensions by joining History Area tables means that the overlap in timelines will be ‘cut’ in multiple records with smaller intervals. This is explained using the following sample datasets (only the ETL process control attributes which are required for this query are shown):
HSTG table 1:
Key
INSERT_DATETIME
Fund Code
Amount
1
2012-01-01
ABC
$1.000.000
2
2013-06-02
ABC
$1.500.000
As opposed to the Integration Area, the History Area only keeps track of changed records and does not support dummy records and other derived information such as expiry dates.
In both tables, the Fund Code is the logical key (source primary key).
HSTG table 2:
Key
INSERT_DATETIME
Fund Code
Short Name
Additional Amount
1
2012-04-05
ABC
ABC Corp
$5.000
2
2013-07-07
ABC
ABC Pty
$5.000
The merging of both the historic data sets where each one contains two records (time periods) results into a new set that has 4 (‘smaller’) records. This should result in the following data set:
Dimension
Key
Fund Code
Short Name
Amount
Additional Amount
Effective Date/Time
Expiry Date/Time
1
ABC
NULL
$1.000.000
$5.000
2012-01-01
2012-04-05
2
ABC
ABC Corp
$1.000.000
$5.000
2012-04-05
2013-06-02
3
ABC
ABC Corp
$1.500.000
$5.000
2013-06-02
2013-07-07
4
ABC
ABC Pty
$1.500.000
$5.000
2013-07-07
9999-12-31
The first record originates from Table 1 and at this point in time there is no match for any record in Table 2. For this reason the values are left as NULL, although they are replaced or handled with default values in the subsequent Presentation Layer ETL operations.
The second record in the Dimension is actually the first record from Table 2. But as the second record in the history timeline it inherits the ‘Amount’ attribute values from the previous record.
The third record is actually the second record from Table 1 which introduces a change in the ‘Amount’ attributes. This has triggered a change in history and this is reflected in the Dimension output.
Similarly, the fourth record is the second record from Table 2 where the ‘Short Name’ attribute is changed.
There are various methods in SQL to achieve this result, and ETL software provides additional flexibility in achieving this but the ANSI SQL solution is displayed here:
-- Select all variations of the available time intervals
WITH TimeIntervals AS
(
  SELECT INSERT_DATETIME FROM HSTG_Table1
  UNION 
  SELECT INSERT_DATETIME FROM HSTG_Table2
),
-- Calculate the ranges (time intervals / slices) between the available time intervals
Ranges AS
(
 SELECT
   INSERT_DATETIME AS EFFECTIVE_DATETIME,
   (
    SELECT
       ISNULL (MIN (INSERT_DATETIME),'99991231') EXPIRY_DATETIME
    FROM TimeIntervals
    WHERE INSERT_DATETIME > TimeIntervals1.INSERT_DATETIME
   ) EXPIRY_DATETIME
 FROM TimeIntervals AS TimeIntervals1
),
-- Connect the source tables (table 1)
Table1 AS (
    SELECT
       c.HSTG_Table1_SK,
       c.Fundcode,
       c.Total_Amount,
       c.INSERT_DATETIME as EFFECTIVE_DATETIME,     
COALESCE
(
                 MIN(c2.INSERT_DATETIME),CONVERT(DATETIME,'99991231')
            ) AS EXPIRY_DATETIME
    FROM HSTG_Table1 c
    LEFT OUTER JOIN HSTG_Table1 c2
    ON   c.Fundcode = c2.Fundcode
    AND  c.INSERT_DATETIME < c2.INSERT_DATETIME
    GROUP BY
       c.HSTG_Table1_SK, c.Fundcode, c.Total_Amount, c.INSERT_DATETIME
),
-- Connect the source tables (table 2)
Table2 AS (
    SELECT
       c.HSTG_Table2_SK,
       c.Fundcode,
       c.Short_name,
       c.Additional_amount,
       c.INSERT_DATETIME as EFFECTIVE_DATETIME,     
       COALESCE(
                            MIN(c2.INSERT_DATETIME),CONVERT(DATETIME,'99991231')
                       ) AS EXPIRY_DATETIME
       FROM HSTG_Table2 c
       LEFT OUTER JOIN HSTG_Table2 c2
       ON     c.Fundcode = c2.Fundcode
       AND    c.INSERT_DATETIME < c2.INSERT_DATETIME
       GROUP BY
          c.HSTG_Table2_SK, c.Fundcode, c.Short_Name, c.Additional_Amount,
          c.INSERT_DATETIME
)
-- Join the various tables to the available time ranges
SELECT
   Table1.Fundcode,
   Table1.Total_Amount,
   Table2.Short_Name,
   Table2.Additional_Amount,
   R.EFFECTIVE_DATETIME,
   R.EXPIRY_DATETIME
FROM Ranges R
LEFT JOIN Table1 ON
   NOT Table1.EFFECTIVE_DATETIME >= R.EXPIRY_DATETIME
   AND NOT Table1.EXPIRY_DATETIME <= R.EFFECTIVE_DATETIME
LEFT JOIN Table2 ON
   NOT Table2.EFFECTIVE_DATETIME >= R.EXPIRY_DATETIME
   AND NOT Table2.EXPIRY_DATETIME <= R.EFFECTIVE_DATETIME
The above SQL statement creates a full historical view of both complete History Area tables. In most cases however the History Area also tracks changes for attributes that might not be relevant for the specific Dimension and omitting these from the subquery creates redundant records in the selection. This is not incorrect in terms of the information contents but does create additional records in the Dimension table which do not contribute to the overall solution and result in performance and storage costs. It is possible to prevent this from happening by filtering these records in the subquery sections where the History Area tables are added to the Common Table Expression in the example:
ROW_NUMBER() OVER (PARTITION BY Table2.key, Table2.Additional_Amount ORDER BY Table2.INSERT_DATETIME) AS RowNumber
The final selection requires a corresponding filter to be applied:
WHERE Table2.RowNumber IS NULL OR Table2.RowNumber=1
In the above function the ‘Short Name’ attribute has been excluded from the selection by making sure only the first record, the one with the earliest effective date, is loaded in case there are duplicate records. This provides the correct outcome in terms of changes over time, but the resulting expiry date will not correspond anymore. This is not an issue as the expiry date is only used inside the Common Table Expression and not used in the rest of the Dimension ETL. Instead, ‘End Dating’ will be handled by a separate ETL process. For this reason, the expiry date should not be loaded into the ETL in production situation and is shown here for demonstration purposes only.
Additionally, performance considerations often prevent a full rebuild of the Dimension table. In these scenarios the SOURCE_CONTROL table can be used to manage load windows to only select the changes that occurred since the last time the ETL process was run. This can be achieved by altering the ‘Timelines’ section of the above mentioned query as follows:
WITH TimeIntervals AS
(
SELECT DISTINCT INSERT_DATETIME
FROM
       (
              SELECT INSERT_DATETIME FROM HSTG_ACURITY_ACU_DEF
              UNION
              SELECT INSERT_DATETIME FROM HSTG_ACURITY_ACU_MIC
       ) TimeSubQuery
WHERE INSERT_DATETIME> <place your control date here>
)
This control data can be passed on to the selection query through the ETL software.
 After this selection the base Dimension structure is defined and subsequent ETL tasks within the same Module can focus on the remaining actions to complete the Dimension:
Replacing NULL values with appropriate default values.
Ensuring the existence of placeholder records.
Implementing specific business logic and/or derivations.
Slowly Changing Dimension comparisons, inserts and updates.
Renaming / beautification of attribute names and contents.
The full overview of corresponding ETL logic is displayed on the next page.
 

Figure 1: ETL Flow for loading the Dimension

## Implementation Guidelines
The logic to calculate the overlaps in the timelines can be implemented both in SQL (source selection SQL) or using ETL components. This depends on the specific software capabilities.
The End Dating mechanism is best implemented as a separate (post) ETL process to support additional functionality such as parallel inserts and post processing of dummy records. It also makes it a lot easier to handle late arriving information as the separate process will also ‘repair’ records that are inserted between existing rows. This Design Patterns contains the Expiry Date/Time for demonstration purposes. However, for the interval calculations the Expiry Date/Time is required and calculated at runtime (and then discarded).
Depending on the database platform used the self-join can be replaced by a LEAD Analytical SQL function. Some platforms including SQL Server 2008R2 do not support this.
Some database platforms support nested Common Table Expressions which can further simplify the SQL.
ETL processes must be able to handle multiple changes / comparisons in one run which means both a potential comparison against existing records which are not the most recent (current) as well as comparisons between records in the selection query. How this is handled varies greatly between the capabilities of the specific ETL platforms.
It is recommended to split the handling of a typical Dimension that consists of a mix of Type 1 and 2 attributes into multiple separate ETL processes:
One that handles all Type 2 changes.
One that handles all Type 1 changes, to be run after the Type 2 ETL process if applicable.
This simplifies the ETL process. Defining a combined ETL template is overly complex as the Type 1 attributes do not require the historical information to be available while the Type 2 attributes require additional handling of multiple change processing in one run.

## Considerations and Consequences
Loading from the History Area requires the availability and implementation of this optional component as part of the overall solution.
Ultimately the decision to load the Presentation Layer from the Staging or History Area depends on the following considerations:
Loading from Staging Area potentially provides the fastest end-to-end loading process since the various Integration Area and History Area ETL processes can be loaded in parallel. Loading HSTG before INT essentially add a 'link in the chain' and add time to the end-to-end process as it requires serial processing (you need to wait for HSTG to finish). So, loading from STG brings you closer to mini batches or Near Real Time kind of loading.
Loading from Staging Area in this context introduces a dependency between the Staging Area and the Integration Area processes. All of the subsequent Integration Area processes must be completed before the delta can be refreshed (i.e. the Staging Area ETL reloads the table with the next delta set). This can be managed by creating a Batch that contains these dependent processes but this also means there is a set turnaround time for the entire Batch. In some ways this omits some of the advantages as mentioned in the previous bullet. However, if you run the Batch often (typically intra-day) or record volumes are low then this is less of an issue, especially if you can persist and update your caches in memory.
There is likely to be a performance penalty on bigger History Area tables requiring implementation of load windows and/or dynamic /persistent caching.
In short:
If the History Area processing take a fair amount of time, uses a lot of cache and/or cannot be configured to use dynamic/persistent caching it is probably better to load from the STG directly.
If the ETL tool does not have adequate caching options (update and persist cache in memory) it is probably better to load from STG.
In very specific situations related to messaging or Change Data Capture it may be possible that changes arrive very late from the operational system. This can happen if for instance an ETL process which reads information is paused for a number of days after which it receives all the changes that have occurred from that point onwards. These records may precede the Dimension load window (control date) at this point and will not be automatically be loaded. There is no easy method of handling this specific kind of late arriving information but alternatives include removing and reloading a small selection of the target table. Care has to be taken to correctly manage the fact table keys in this case.
An alternative solution to the above situation is to extend the example selection query with a comparison of the already processed Module Instance IDs from the SOURCE_CONTROL table. The proposed logic would be: select all records from the HSTG tables where the INSERT_DATETIME is greater than the control date or any Module Instance IDs that are not already present in the SOURCE_CONTROL table but are greater than the earliest available control date.
During the designated recovery (rollback) process the SOURCE_CONTROL table must be rolled back as well.
Known uses
None.

## Related Patterns
Design Pattern 019 – Data Vault – Creating Dimensions from Hub tables.
Design Pattern 006 – Generic – Managing temporality by using Start, Process and End dates.
Discussion items (not yet to be implemented or used until final)
None.
