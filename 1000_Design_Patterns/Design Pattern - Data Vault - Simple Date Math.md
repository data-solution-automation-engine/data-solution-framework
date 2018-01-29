# Design Pattern - Data Vault - Simple Date Math (Joining two Time-Variant Tables

## Purpose
This design pattern describes how to create a typical ‘Type 2 Dimension’ table (Dimensional Modelling) from a Data Vault or Hybrid EDW model.

## Motivation
To move from a Data Vault (or other Hybrid) model to a Kimball-style Star Schema or similar requires various tables that store historical data to be joined to each other. This is a recurring step which, if done properly, makes it easy to change dimension structures without losing history. Merging various historic sets of data is seen as one of the more complex steps in a Data Vault (or similar) environment. The pattern is called ‘creating Dimensions from Hub’ tables because Hubs are the main entities which are linked together to form a Dimension using their historical information and relationships.
Also known as
Dimensions / Dimensional Modelling
Gaps and islands
Timelines

## Applicability
This pattern is only applicable for loading processes from source systems or files to the Reporting Structure Area (of the Presentation Layer). The Helper Area may use similar concepts but since this is a ‘free-for-all’ part of the ETL Framework it is not mandatory to follow this Design Pattern.

## Structure
Creating Dimensions from a Data Vault model essentially means joining the various Hub, Link and Satellite tables together to create a certain hierarchy. In the example displayed in the following diagram the Dimension that can be generated is a ‘Product’ dimension with the Distribution Channel as a higher level in this dimension.

 Business Insights > Design Pattern 019 - Creating Dimensions from Hub tables > BI7.png

Figure 1: Example Data Vault model
Creating dimensions by joining tables with history means that the overlap in timelines (effective and expiry dates) will be ‘cut’ in multiple records with smaller intervals. This is explained using the following sample datasets, only the tables which contain ‘history’ are shown.

SAT Product

Key|Product Name|Effective Date|Expiry Date	
--|---|---|---|---
73|- (dummy)|01-01-1900|01-01-2009
73|Cheese|01-01-2009|05-06-2010	
73|Cheese – Yellow|05-06-2010|04-04-2011
73|Cheese – Gold|04-04-2011|31-12-9999

The first record is a dummy record created together with the Hub record. This was updated as part of the history / SCD updates.
Before being joined to the other sets this Satellite table is joined to the Hub table first. The Hub table maps the Data Warehouse key ‘73’ to the business key ‘CHS’.

SAT Product –Channel (Link-Satellite):
Link Key
Product Key
Channel Key
Effective Date
Expiry Date

This set indicates that the product has been moved to a different sales channel over time.

1
73
-1 (dummy)
01-01-1900
01-01-2010
2
73
1
01-01-2010
04-03-2011
3
73
2
04-03-2011
31-12-9999

When merging these to data sets into a dimension the overlaps in time are calculated:

 Business Insights > Design Pattern 019 - Creating Dimensions from Hub tables > BI8.png
Figure 2: Timelines

In other words, the merging of both the historic data sets where one has 4 records (time periods) and the other one has 3 records (time periods) results into a new set that has 6 (‘smaller’) records. This gives the following result data set (changes are highlighted):\
Dimension Key
Product Key
Product
Channel Key
Effective Date
Expiry Date
1
73
-
-1
01-01-1900
01-01-2009
2
73
Cheese
-1
01-01-2009
01-01-2010
3
73
Cheese
1
01-01-2010
05-06-2010
4
73
Cheese-Yellow
1
05-06-2010
03-04-2011
5
73
Cheese-Yellow
2
03-04-2011
04-04-2011
6
73
Cheese-Gold
2
04-04-2011
31-12-9999

This result can be achieved by joining the tables on their usual keys and calculating the overlapping time ranges:
SELECT 
 B.PRODUCT_NAME,
 C.CHANNEL_KEY,
(CASE
   WHEN B.EFFECTIVE_DATE > D.EFFECTIVE_DATE
   THEN B.EFFECTIVE_DATE
   ELSE D.EFFECTIVE_DATE
 END) AS EFFECTIVE_DATE, -- greatest of the two effective dates
(CASE
   WHEN B.EXPIRY_DATE < D.EXPIRY_DATE
   THEN B.EXPIRY_DATE
   ELSE D.EXPIRY_DATE
 END) AS EXPIRY_DATE -- smallest of the two expiry dates
FROM HUB_PRODUCT A
JOIN SAT_PRODUCT B ON A.PRODUCT_SK=B.PRODUCT_SK
JOIN LINK_PRODUCT_CHANNEL C ON A.PRODUCT_SK=C.PRODUCT_SK
JOIN SAT_LINK_PRODUCT_CHANNEL D ON D.PRODUCT_CHANNEL_SK=C.PRODUCT_CHANNEL_SK
WHERE
(CASE
   WHEN B.EFFECTIVE_DATE > D.EFFECTIVE_DATE
   THEN B.EFFECTIVE_DATE
   ELSE D.EFFECTIVE_DATE
 END) -- greatest of the two effective dates
 <
(CASE
   WHEN B.EXPIRY_DATE < D.EXPIRY_DATE
   THEN B.EXPIRY_DATE
   ELSE D.EXPIRY_DATE -- smallest of the two expiry dates
 END)

## Implementation guidelines
The easiest way to join multiple tables is a cascading set based approach. This is done by joining the Hub and Satellite and treating this as a single set which is joined against another similar set of data (for instance a Link and Link-Satellite). The result of this is a new set of consistent timelines for a certain grain of information. This set can be treated as a single set again and joined with the next set (for instance a Hub and Satellite) and so forth.
When creating a standard Dimension table it is recommended to assign new surrogate keys for every dimension record. The only reason for this is to prevent a combination of Integration Layer surrogate keys to be present in the associated Fact table. The range of keys can become very wide. This also fits in with the classic approach towards loading Facts and Dimensions where the Fact table ETL performs a key lookup towards the Dimension table. Using Data Vault as Integration Layer opens up other options as well but this is a well-known (and understood) type of ETL.
The original Integration Layer keys remain attributes of the new Dimension table.
Creating a Type 1 Dimension is easier; only the most recent records can be joined.
Joining has to be done with < and > selections, which not every ETL tool supports (easily). This may require SQL overrides.
Some ETL tools or databases make the WHERE clause a bit more readable by providing a ‘greatest’ or ‘smallest’ function.
This approach requires the timelines in all tables to be complete, ensuring referential integrity in the central Data Vault model. This means that every Hub has to have a record in the Satellite table with a start date of ‘01-01-1900’ and one which ends at ‘31-12-9999’ (can be the same record if there is no history yet). Without this dummy record to complete the timelines the query to calculate the overlaps will become very complex. SQL filters the records in the original WHERE clause before joining to the other history set. This requires the selection on the date range to be done on the JOIN clause but makes it impossible to get the EXPIRY_DATE correct in one pass. The solution with this approach is to only select the EFFECTIVE_DATE values, order these, and join this dataset back to itself to be able to compare the previous row (or the next depending on the sort) and derive the EXPIRY_DATE. In this context the solution to add dummy records to complete the timelines is an easier solution which also improves the integrity of the data in the Data Vault model.

## Considerations and consequences
This approach requires the timelines in all tables to be complete, ensuring referential integrity in the central Data Vault model. This means that every Hub has to have a record in the Satellite table with a start date of ‘01-01-1900’ and one which ends at ‘31-12-9999’ (can be the same record if there is no history yet). Without this dummy record to complete the timelines the query to calculate the overlaps will become very complex. SQL filters the records in the original WHERE clause before joining to the other history set. This requires the selection on the date range to be done on the JOIN clause but makes it impossible to get the EXPIRY_DATE correct in one pass. The solution with this approach is to only select the EFFECTIVE_DATE values, order these, and join this dataset back to itself to be able to compare the previous row (or the next depending on the sort) and derive the EXPIRY_DATE. In this context the solution to add dummy records to complete the timelines is an easier solution which also improves the integrity of the data in the Data Vault model.
Known uses

This type of ETL process is to be used to join historical tables together in the Integration Layer.

## Related patterns
Design Pattern 002 – Generic – Types of history
Design Pattern 006 – Generic – Using Start, Process and End dates.
Design Pattern 008 – Data Vault – Loading Hub tables
Design Pattern 009 – Data Vault – Loading Satellite tables
Design Pattern 010 – Data Vault – Loading Link tables.
