# Design Pattern — Dimensional Modelling - Loading a Dimension from the Persistent Staging Area

## Purpose

This design pattern supports a **2-tiered (Staging and Presentation Layer only) Data Warehouse architecture**. In most scenarios, this solution depends on loading processes that select information from the **Persistent Staging Area** to update the Presentation Layer.

The Persistent Staging Area acts as a "safety catch" recording all transactions and changes passing through the Data Warehouse and provides the functionality to rebuild the Presentation Layer star and/or snowflake models.

## Motivation

Using the Persistent Staging Area as the source table for Presentation Layer updates in a 2-tiered approach offers several advantages:

* **Decouples delta gathering from Data Mart ETL processes**  
  Delta processes can run at different frequencies from Presentation Layer ETL processes because the Persistent Staging Area always holds the most recent data. This enables more frequent and independent staging.

* **Single source for all loading types** (initial load, re-initialization, regular runs)  
  ETL can merge historical changes with the target Dimension, allowing a single template for all load types. Load windows limit the Data Mart updates to changed records only.

* **Enables full Presentation Layer reloads**  
  For example, when changing an attribute from Type 1 to Type 2 with history backload, changing Fact table grain, or adding a new Dimension level.

**Also known as:**

* Data Staging  
* Dimensional Loading


## Applicability

This pattern is applicable for loading the Presentation Layer in a **2-tiered Data Warehouse architecture**.  
While not directly applicable to 3-tiered architectures, much of the logic and SQL applies to any set of historical records.


## Structure

The logic to populate a Dimension from two or more historical sources is similar to loading from the Integration Layer (see Design Pattern 019). However, the Persistent Staging Area lacks certain Integration Layer features such as:

* Dummy record handling  
* Placeholder / unknown value taxonomy  
* Expiry dates

Therefore, the logic is slightly more complex. Joining Persistent Staging Area tables results in overlapping timelines being split into multiple smaller interval records.

### Example Datasets

| PSA Table 1 | Key | INSERT_DATETIME | Fund Code | Amount     |
|--------------|-----|-----------------|-----------|------------|
|              | 1   | 2012-01-01      | ABC       | $1,000,000 |
|              | 2   | 2013-06-02      | ABC       | $1,500,000 |

| PSA Table 2 | Key | INSERT_DATETIME | Fund Code | Short Name | Additional Amount |
|--------------|-----|-----------------|-----------|------------|-------------------|
|              | 1   | 2012-04-05      | ABC       | ABC Corp   | $5,000            |
|              | 2   | 2013-07-07      | ABC       | ABC Pty    | $5,000            |

* **Fund Code** is the logical key in both tables.
* Persistent Staging Area only tracks changed records, no dummy records or expiry dates.

### Resulting Dimension

| Key | Fund Code | Short Name | Amount     | Additional Amount | Effective Date | Expiry Date   |
|-----|-----------|------------|------------|-------------------|----------------|---------------|
| 1   | ABC       | NULL       | $1,000,000 | $5,000            | 2012-01-01     | 2012-04-05    |
| 2   | ABC       | ABC Corp   | $1,000,000 | $5,000            | 2012-04-05     | 2013-06-02    |
| 3   | ABC       | ABC Corp   | $1,500,000 | $5,000            | 2013-06-02     | 2013-07-07    |
| 4   | ABC       | ABC Pty    | $1,500,000 | $5,000            | 2013-07-07     | 9999-12-31    |

**Explanation:**

- Record 1: From Table 1, no matching record in Table 2 yet, so `Short Name` is NULL (to be handled later).  
- Record 2: First record from Table 2, inherits `Amount` from previous record.  
- Record 3: Second record from Table 1 changes the `Amount`.  
- Record 4: Second record from Table 2 changes the `Short Name`.

---

## Sample SQL (ANSI SQL)

```sql
-- Select all variations of the available time intervals
WITH TimeIntervals AS (
  SELECT INSERT_DATETIME FROM PSA_Table1
  UNION
  SELECT INSERT_DATETIME FROM PSA_Table2
),

-- Calculate the ranges (time intervals / slices) between available time intervals
Ranges AS (
  SELECT
    INSERT_DATETIME AS EFFECTIVE_DATETIME,
    (
      SELECT ISNULL(MIN(INSERT_DATETIME), '99991231')
      FROM TimeIntervals
      WHERE INSERT_DATETIME > TimeIntervals1.INSERT_DATETIME
    ) AS EXPIRY_DATETIME
  FROM TimeIntervals AS TimeIntervals1
),

-- Connect source table 1
Table1 AS (
  SELECT
    c.PSA_Table1_SK,
    c.Fundcode,
    c.Total_Amount,
    c.INSERT_DATETIME AS EFFECTIVE_DATETIME,
    COALESCE(MIN(c2.INSERT_DATETIME), CONVERT(DATETIME, '99991231')) AS EXPIRY_DATETIME
  FROM PSA_Table1 c
  LEFT JOIN PSA_Table1 c2 ON
    c.Fundcode = c2.Fundcode AND
    c.INSERT_DATETIME < c2.INSERT_DATETIME
  GROUP BY c.PSA_Table1_SK, c.Fundcode, c.Total_Amount, c.INSERT_DATETIME
),

-- Connect source table 2
Table2 AS (
  SELECT
    c.PSA_Table2_SK,
    c.Fundcode,
    c.Short_name,
    c.Additional_amount,
    c.INSERT_DATETIME AS EFFECTIVE_DATETIME,
    COALESCE(MIN(c2.INSERT_DATETIME), CONVERT(DATETIME, '99991231')) AS EXPIRY_DATETIME
  FROM PSA_Table2 c
  LEFT JOIN PSA_Table2 c2 ON
    c.Fundcode = c2.Fundcode AND
    c.INSERT_DATETIME < c2.INSERT_DATETIME
  GROUP BY c.PSA_Table2_SK, c.Fundcode, c.Short_Name, c.Additional_Amount, c.INSERT_DATETIME
)

-- Join tables to time ranges
SELECT
  Table1.Fundcode,
  Table1.Total_Amount,
  Table2.Short_Name,
  Table2.Additional_Amount,
  R.EFFECTIVE_DATETIME,
  R.EXPIRY_DATETIME
FROM Ranges R
LEFT JOIN Table1 ON NOT (Table1.EFFECTIVE_DATETIME >= R.EXPIRY_DATETIME OR Table1.EXPIRY_DATETIME <= R.EFFECTIVE_DATETIME)
LEFT JOIN Table2 ON NOT (Table2.EFFECTIVE_DATETIME >= R.EXPIRY_DATETIME OR Table2.EXPIRY_DATETIME <= R.EFFECTIVE_DATETIME)
```

## Related patterns

* Design Pattern 019 - Data Vault - Creating Dimensions from Hub tables.
* Design Pattern 006 - Generic - Managing temporality by using Start, Process and End dates.
