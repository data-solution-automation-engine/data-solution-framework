# Design Pattern - Generic - Handling Logical Deletes

## Purpose
This Design Pattern standardises the way that physical deletes from source systems are handled in the Data Warehouse. A *physical* delete in the operational system should lead to a *logical* delete in the Data Warehouse.

## Motivation
Correctly processing logical deletes is important to maintain an accurate state of the historical information. If a record is deleted from a source system and this event is captured (typically using some sort of CDC or full table compare) this should lead to an update of the Data Warehouse data, and not a physical delete. 

This impacts how history is stored: the effective period changes as well as some of the indicators. 


## Applicability
This type of ETL implementation is valid in any situation where logical deletes are captured, in tables that store or represent a historical view of information such as the Persistent Staging Area, and most of the Integration and Presentation Layer tables.

## Structure
Essentially the logical delete is handled as an insert/update to the target table. This means that a new row is inserted using the Load Date/Time Stamp as the Effective Date of the new record.

This concept introduces the Deleted Row Indicator. The effective record will be closed (updated) and the new record with the delete indicator will become the new valid and most recent record from then on for eternity or until the record will be reopened by the source.
This also means that the new record which contains the ‘deleted’ indicator will contain all the information / attribute values from when the record was last active.

For example:
Initially a new record is inserted.

DWH Key | Logical Key | Name | Colour| Current |Deleted | Effective Date | Expiry Date
---|---|---|---|---|---|---|---
1 | CHS | Cheese | YEL | Y| N | 10-01-2006| 31-12-9999

After a while, the record is deleted from the source and captured using a CDC mechanism. The new row which represents the deleted record is inserted and the original row is updated for the current row indicator and expiry date.

DWH Key | Logical Key | Name | Colour| Current |Deleted | Effective Date | Expiry Date
---|---|---|---|---|---|---|---
1 | CHS | Cheese | YEL | N | N | 10-01-2006 | 04-02-2010
2 | CHS | Cheese | YEL | Y | Y	| 04-02-2010 | 31-12-9999

The Data Warehouse now contains all the correct information to reopen the record if that situation occurs.

DWH Key | Logical Key | Name | Colour| Current |Deleted | Effective Date | Expiry Date
---|---|---|---|---|---|---|---
1 | CHS | Cheese | YEL | N | N | 10-01-2006 | 04-02-2010
2 | CHS | Cheese | YEL | Y | Y | 04-02-2010 | 02-06-2011
3 | CHS | Cheese | YEL | Y | N | 02-06-2011 | 31-12-9999

The Deleted Row Indicator (‘Deleted’ in this example) keeps the design straightforward and provides the information that a record has been deleted from an effective date onwards including the image when it was deleted.

## Implementation Guidelines
* Treat the Deleted Row Indicator as any attribute that triggers a SCD2 mechanism.
* If you have a Change Data Capture based source the attribute comparison may potentially be skipped because the CDC mechanisms inform the system whether the record in the Staging Area is new, updated or deleted.

## Considerations and Consequences
* A deleted record in the source system leads to an extra record in the Data Warehouse. With this in mind the meaning of the (Data Warehouse) effective and expiry dates should be very clear: they indicated the time interval for when a value/record was active.
* This approach stores the last state of the record when it was deleted. This is a design decision. As an alternative the values can also be set to NULL although this may cause some problems when re-opening records. 

## Related Patterns
* Design Pattern 006 - Generic - Managing Temporality by using Start, Process and End Dates.
