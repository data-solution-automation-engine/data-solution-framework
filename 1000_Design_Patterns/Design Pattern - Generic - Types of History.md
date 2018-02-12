# Design Pattern - Generic - Types of History

## Purpose
This design pattern describes the definitions for the commonly used history storage concepts.

## Motivation
Due to definitions changing over time and different definitions being made by different parties there usually is a lot of discussion about what exactly constitutes the different types of history. This design pattern aims to define these history types in order to provide the common ground for discussion.

This is also known as:
* SCD; Slowly Changing Dimensions
* Type 1,2,3,4 etc.

## Applicability
Every situation where historical data is needed / stored or a discussion arises. 

Depending on the Data Warehouse architecture, this can be needed in a variety of situations. But typically these concepts are applied in the integration and presentation layer of the Data Warehouse.

## Structure
The following history types are defined, some distinction is made where there are multiple viable explanations. All definitions can be valid coming from a specific background and in order to cater for every situation some history types are tagged with specific letters indicating a slightly different approach.

**Type 0**. No change, while uncommon it has to be mentioned that this passive approach sometimes is implemented when storage space is to be saved or only the initial state has to be preserved.

**Type 1 – A**. Change only the latest record. This implementation of type 1 is implemented if there is limited interest in keeping a specific kind of history. A good example is spelling errors; only the latest record is updated in that case (if you’re not interested in the wrong spelling for data quality purposes). 

An example of the first instance of a type 1-A change:
Old situation; a record exists for the logical key CHS (Cheese). The attribute Name is defined as a type 1(A) attribute.

DWH Key	| Logical Key | Name | Colour | Start date | End date | Update date
--- | --- | --- | --- | --- | --- | ---
3 | CHS | Cheese | Golden | 05-01-2000 | 31-12-9999 | 05-01-2000
2 | CHS | Cheese | Yellow | 11-01-1996 | 04-01-2000 | 11-01-1996
1 | CHS | Cheese | Yellow | 07-03-1994 | 10-01-1996 | 10-01-1996

When at some point (at 24-06-2006) the name is changed to *Old Cheese* and the Name attribute is defined as type 1(A) the name is overwritten, resulting in the following: 

DWH Key	| Logical Key | Name | Colour | Start date | End date | Update date
--- | --- | --- | --- | --- | --- | ---
3 | CHS | Old Cheese | Golden | 05-01-2000 | 31-12-9999 | 24-06-2006
2 | CHS | Cheese | Yellow | 11-01-1996 | 04-01-2000 | 11-01-1996
1 | CHS | Cheese | Yellow | 07-03-1994 | 10-01-1996 | 10-01-1996

**Type 1 – B**. Update the entire history based on the latest situation. The previous example for the second version of type 1 is as follows:
Old situation; a record exists for the logical key CHS (Cheese). The attribute Name is defined as a type 1(B) attribute.

DWH Key	| Logical Key | Name | Colour | Start date | End date | Update date
--- | --- | --- | --- | --- | --- | ---
3| CHS | Cheese | Golden | 05-01-2000 | 31-12-9999 | 05-01-2000
2 | CHS	| Cheese | Yellow | 11-01-1996 | 04-01-2000 | 11-01-1996
1 | CHS | Cheese | Yellow |07-03-1994 | 10-01-1996 | 10-01-1996

When at some point (at 24-06-2006) the name is changed to Old Cheese and the Name attribute is defined as type 1(B) the name is overwritten, resulting in the following: 

DWH Key	| Logical Key | Name | Colour | Start date | End date | Update date
--- | --- | --- | --- | --- | --- | ---
3| CHS | Old Cheese | Golden | 05-01-2000 | 31-12-9999 | 24-06-2006
2 | CHS | Old Cheese | Yellow | 11-01-1996 | 04-01-2000 | 24-06-2006
1 | CHS | Old Cheese | Yellow | 07-03-1994 | 10-01-1996 | 24-06-2006

**Type 2** / also known as SCD-type2. The slowly changing dimension type 2 concept tracks history by inserting a new record and closing the most recent corresponding record whenever a change occurs.
A new record is inserted in the Data Warehouse table.

DWH Key	| Logical Key | Name | Colour | Start date | End date | Update date
--- | --- | --- | --- | --- | --- | ---
1 | CHS | Cheese | Golden | 05-01-2000 | 31-12-9999 | 05-01-2000

In this case you have basic information of a product; the name is the attribute that can change over time. This record has been inserted on the 1st of January 2000 and is still active. But now, on the 20th July 2008 the name changes to Old Cheese. This will lead to a new record and an updated previous record for the same DWH key.

DWH Key	| Logical Key | Name | Colour | Start date | End date | Update date
--- | --- | --- | --- | --- | --- | ---
2 | CHS | Cheese | Golden | 20-07-2008 | 31-12-9999 | 20-07-2000
1 | CHS | Cheese | Golden | 05-01-2000 | 19-07-2008 | 05-01-2000

**Type 3** history stores history in a separate attribute. As many attributes can be added to a record as the previous states that need to be captured. Typically only the previous state is recorded in the separate attribute. An example would be:
A new record is inserted in the Data Warehouse table on 12-10-2009:

DWH Key	| Logical Key | Name | Previous Name | Colour | Update date
--- | --- | --- | --- | --- | --- 
1 | CHS | Cheese | NULL | Golden | 12-10-2009

When the name is changed to Old Cheese on February 2010 it leads to the following results:

DWH Key	| Logical Key | Name | Previous Name | Colour | Update date
--- | --- | --- | --- | --- | --- 
1 | CHS | Old Cheese | Cheese | Golden | 02-02-2010

**Type 4**. This history tracking mechanism operates by using separate tables to store the history. One table contains the most recent version of the record and the history table contains some or all history.

**Type 5**. The type 5 method of tracking history uses versions of tables for every period in time. Also known as ‘snapshotting’. No example is supplied since it’s basically a copy of the entire table.

**Type 6 / hybrid**. Also known as ‘twin time stamping’, the type 6 approach combines the concepts of type 1-B, type 2 and type 3 mechanisms (1+2+3=6!). In the following example the attribute combination is the name. It consists of two attributes.
A new record is inserted in the Data Warehouse table.

DWH Key	| Logical Key | Name | Current Name | Colour | Start date | End date 
--- | --- | --- | --- | --- | --- | ---
1 | CHS | Cheese | Cheese | Golden | 05-01-2000 | 31-12-9999

After some time the name is changed to Old Cheese. This leads to a SCD2 event where a new record is inserted and an old one is closed off. At the same time, the history of the existing type 3 attribute is overwritten by a type 1-B event.        

DWH Key	| Logical Key | Name | Current Name | Colour | Start date | End date 
--- | --- | --- | --- | --- | --- | ---
2 | CHS | Old Cheese | Old Cheese | Golden | 20-07-2008 | 31-12-9999
1 | CHS | Cheese | Old Cheese | Golden | 05-01-2000 | 19-07-2008

Now you can see the previous record and all related facts against both the current and historical name. When a new change occurs, the following happens:

DWH Key	| Logical Key | Name | Current Name | Colour | Start date | End date 
--- | --- | --- | --- | --- | --- | ---
3 | CHS | A+ Cheese | A+ Cheese | Golden | 13-03-2010 | 31-12-9999
2 | CHS | Old Cheese | A+ Cheese | Golden | 20-07-2008 | 12-03-2010
1 | CHS | Cheese | A+ Cheese | Golden | 05-01-2000 | 19-07-2008

## Implementation Guidelines
* Obviously, corresponding records are identified by the logical key.
* Type 1-B and the corresponding concept in Type 6 usually require separate mappings to update the entire history. Special care from a performance perspective because it has to be avoided that the entire history will be rewritten over and over again when really only the latest situation for that logical key. This mapping will have to aggregate the dataset to merge the latest state per natural key with the target table, and it will have to run after the regular Type 2 processes.
* Avoid using NULL in the end date attribute of the most recent record to indicate an open / recent record date. Some databases have troubles handling NULL values and it is best practice to avoid NULL values wherever possible, especially in dimensions.
* It is advised to add an ‘current record indicator’ for quick querying and easy understanding.
* Depending on the location in the Data Warehouse either tables or attributes may be defined for a specific history type. For instance, defining a table as SCD Type 2 means that a change in every attribute will lead to a new record (and closing an old one). In Data Marts the common approach is often to specify a history type per attribute. So a change in one attribute may lead to an SCD Type 2 event, but a change in another one may cause the history to be overwritten.

## Considerations and Consequences
Not applicable.

## Related Patterns
* Design Pattern 011 – Kimball – Multiple SCD2 time periods.
* Design Pattern 005 – Generic – Current view on historical data.
* Design Pattern 007 – Kimball – Receiving order of information and late and early arrivals.