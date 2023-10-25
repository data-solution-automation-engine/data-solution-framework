# Design Pattern - Generic - Using checksums for row comparison

## Purpose
This Design Pattern aims to clarify and support the implementation of checksum values for record comparisons.
Motivation
Checksums can provide a performance and maintenance efficient method for detecting changes. A fundamental design principle of the ETL Framework is restartability and the general ability for ETL to be run at any time. A key design decision to support this principle is the implementation of multiple change detection mechanisms to avoid duplicate inserts, for instance when ETL is executed multiple times for the same source set.
Also known as
Delta detection.
Cyclic redundancy checks / Hash Bytes
Applicability
This pattern is applicable in any ETL process that performs record comparisons against historical records. In some cases it may be required to calculate the checksum in the Staging Area as well, for instance as part of a Full Outer Join interface or Disaster Recovery for native CDC.
Structure
The key aspect regarding the role of record comparisons (with or without using checksums to achieve this) is to be aware of the role the CDC_OPERATION plays.  
The possible scenarios as listed in the table below:
Process
Approach
Reasoning
Source to Staging (Full Outer Join or Disaster Recovery only)
Checksum is created and stored in the Staging Area table.
The checksum does not need to be recalculated from Staging to History as the full outer join process will always correctly identify the CDC operation.
In case of a Logical Delete, all attributes will retain their last known value (copied from the History Area table) thus the checksum will be calculated with these values.
Source to Staging (CDC, push or pull)
No checksums are required.
Other interfaces rely on load windows to select the delta sets.
Staging to History
If the checksum is available in the Staging Area the value can be copied into the History Area. Otherwise the checksum is calculated based on the source attributes.
Checksum values can be identical between inserts/updates and records that have been identified as a logical delete. Therefore record comparison must include the CDC operation.
If the checksums are different or the checksums are the same but the <CDC operation> is different; continue the SCD2 operation.
If the checksums are identical and the <CDC operation> is the same as well; discard (filter).
Staging to Integration
The checksum is always calculated within the Staging to Integration ETL process based on the necessary attributes in the Staging Area table. This also requires the <CDC operation> to be part of the checksum attributes.
The comparison is executed based on the new checksum and the existing Integration Area checksum. As with the Staging to History process the <CDC operation> is evaluated to identify Logical Deletes, but in this scenario this happens as part of the checksum creation (i.e. source attributes and the CDC operation will be the new checksum).

The following diagram displays how checksum values are used:

Figure 1: Checksums and comparisons
Implementation guidelines
Using checksums is an alternative to attribute-per-attribute comparison of records. Each method has its advantages and disadvantages and the applicability is ultimately based on the functionality the ETL or database specific functions can provide.
Advantages of attribute comparison include avoiding checksum ‘collision’; i.e. detecting too many changes or (worse) missing changes. The collision rate depends on the used algorithm but no checksum algorithm is 100% perfect.
Disadvantages of attribute comparison include issues regarding NULL handling (requiring dummy values) and rounding of numeric values.
Different ETL platforms or databases have varying functionality regarding to checksums. Careful testing is required when not sure of the behavior of the specific checksum.
When deciding between the checksum options, storage must be taken into account as well. Checksum data types range between integer values to 24 byte character (hash) values.
Consequences
Using checksums limits cache usage in ETL tools. In some cases only a combination of integer values is loaded as opposed to complete target tables.
Checksum values have proven to be convenient while troubleshooting as the values are available as attributes. This is opposed to attribute-per-attribute comparisons which are done at runtime and require ETL debugging to troubleshoot.
Known uses
None.
Related patterns
Design Pattern 009 – Data Vault – Loading Satellite tables.
Design Pattern 014 – Generic – Handling Logical Deletes.
Design Pattern 015 – Generic – Loading Staging Area tables.
Design Pattern 017 – Generic – Loading History Area tables.
Design Pattern 028 – Generic – Full Outer Join interfaces.

## Motivation



## Applicability



## Structure



## Implementation Guidelines



## Considerations and Consequences



## Related Patterns

- 