---
uid: design-pattern-generic-full-outer-join-interfaces
---

# Design Pattern - Interfacing - Full Outer Join interfaces

## Purpose

There are a limited number of options available to bring information into the Enterprise Data Warehouse environment, all of which are compliant with the overall reference architecture and the requirements of the Staging Layer (Landing Area). The Full Outer Join mechanism is one of these potential interfaces. It is particularly well suited to interface smaller datasets without the overhead that Change Data Capture introduces.  This Design Pattern aims to define how the Full Outer Join mechanism can be best implemented using available options as provided by the data logistics Framework reference architecture.

## Motivation

The Full Outer Join mechanism is often implemented in Data Warehouse implementations, usually alongside other interfaces. In some cases it is the only viable option due to restrictions provided by the Source systems. The data logistics Framework provides useful options to be used for the Full Outer Join mechanisms and provides detailed requirements to ensure consistency using this mechanism with the reference architecture and concepts including the Event/Date time and the fundamental data logistics requirements.
Also known as
Delta detection.

## Applicability

This pattern is only applicable to designated Source-to-Staging (Staging Layer) data logistics processes. Application of this interface type is the default for all interfaces belonging to a particular source.

## Structure

The process of the Full Outer Join mechanism to detect delta information is as follows:
Load the Source table dataset (without filters).
Load the Persistent Staging Area dataset, but only for the most recent version of each record / key. This is the record with the most recent INSERT_DATETIME (effective date). Additionally, only records which are not labeled as a logical delete (CDC_OPERATION <> Delete) must be selected to avoid redundant processing as these records will not be available in the Source system anymore.
Perform the Full Outer Join by joining the Source and History datasets on the table keys. For the Persistent Staging Area this not the Primary Key but the original Source system key, also identified as being part of the Unique Index on the History table.
Interpret the join results, either by using a checksum function or attribute comparison.  The interpretation is as follows:
FULL_ROW_CHECKSUM (if used): if the Source key is NULL then use the History Checksum going forward, otherwise use the Source Checksum.
CDC_OPERATION uses the following logic:
If the History key is NULL then Insert.
Otherwise if the Source key is null then Delete.
Otherwise if the checksums and/or attributes are different then Update.
If none of the above then No Change.
Key attributes(s): if the Source key is NULL select the History key value, otherwise use the Source key value.
Other attributes(s): if the Source key is NULL select the History attribute value, otherwise use the Source attribute value.
Any records that have been labeled as No Change are filtered out (discarded) as this stage.
The remaining records are inserted into the Landing Area table as the delta set to be processed further. In these steps the standard metadata attributes such as the RECORD_SOURCE and INSERT_DATETIME are defined.
In the case of a Full Outer Join mechanism the Event / Date Time is dependent on the execution of the data logistics process since this defines the point in time where the comparison is done. For this purpose, the logic to derive the INSERT_DATETIME is:
If the Module (Instance) is run as part of a Batch, the start timestamp of the Batch (BATCH_INSTANCE_START_DATETIME) is used as INSERT_DATETIME.
If the Module is run independently, the start timestamp of the Module is used as INSERT_DATETIME.
Implementation guidelines
Typical implementations of the Full Outer Join mechanism cover the direct comparison between the source system and the Persistent Staging Area although alternative are possible to avoid potentially unwanted intrusion from the perspective of the operational system. These alternatives include the handling of a full dump of the source systems tables into a separate section of the Landing Area (landing zone) from which the default process can continue.
When comparing datasets, either by using a checksum or attribute comparison, the impact of the data type differences between the source and the Persistent Staging Area have to be taken into account. The exact behavior for this is dependent on the combination of the data logistics and database platforms that are being used. In all scenarios the checksum should be calculated after the data type conversion (which is part of the Landing Area requirements).
Consequences
The Full Outer Join mechanism is an easy to manage interface type that has little overhead in terms of system control.
The effectivity of the Full Outer Join mechanism is dependent on the record counts in the tables being sourced via this interface. For (very) big datasets the Full Outer Join is not the ideal mechanism and for this reason it is not scalable to big datasets (with the exception of MPP platforms).
Compared to other Source-to-Staging mechanisms the Full Outer Join is relatively inaccurate to detect the actual moment that changes occurred in the operational system (Event timestamp) as the detected change is related to the execution time.
The design decision to only load the delta record set into the Landing Area ensures that subsequent processes are all handled the same way, as most source-to-staging mechanism are delta based.
By its nature, the Full Outer Join can only provide a single snapshot / historical interval for each run. This is opposed to more granular transaction log based Change Data Capture mechanisms which capture all changes that have occurred and typically contain multiple changes for a single key in the Landing Area before it is processed further.

## Implementation guidelines

## Considerations and consequences

## Related patterns

* [Design Pattern 015  Generic  Loading Landing Area tables](xref:design-pattern-staging-layer-landing-area).


