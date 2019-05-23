# Solution Pattern - Data Modelling - Data Vault Integration Layer

## Purpose
This Implementation Pattern describes the data modelling conventions for a Data Vault based Integration Layer. It documents how the ETL process control attributes relate to support Data Vault based ETL
Structure
The modelling conventions for Data Vault are as follows:
Table Type
Table name convention
Mandatory attribute
Comments
Hub 
HUB_<name>

<table name without HUB_>_<key>
ETL Process Control Id
Load Date / Time
Record Source
<Business Key>
The first attribute (SK) is the primary key
An unique key / index is placed on the <Business Key> attribute (5)
Link
LNK_<name>
<table name without LNK_>_<key>
ETL Process Control Id
Load Date / Time
Record Source
<Hub Keys>
The first attribute (SK) is the primary key
An unique key / index is placed on the combination of Hub keys
Satellite
SAT_<name>
<Hub key – inherited from Hub table>
OMD_EFFECTIVE_DATETIME
OMD_CURRENT_RECORD_INDICATOR
OMD_EXPIRY_DATETIME
OMD_INSERT_MODULE_INSTANCE_ID
OMD_UPDATE_MODULE_INSTANCE_ID
OMD_DELETED_RECORD_INDICATOR
OMD_SOURCE_ROW_ID
OMD_CHECKSUM
The first 3 attributes compose the primary key
The Hub Key attribute is not set in this table, but inherited from the parent Hub table
Link Satellite
LSAT_<name>
<LNK key – inherited from Link table>
OMD_EFFECTIVE_DATETIME
OMD_CURRENT_RECORD_INDICATOR
OMD_EXPIRY_DATETIME
OMD_INSERT_MODULE_INSTANCE_ID
OMD_UPDATE_MODULE_INSTANCE_ID
OMD_DELETED_RECORD_INDICATOR
OMD_SOURCE_ROW_ID
OMD_CHECKSUM
The first 3 attributes compose the primary key
The Link Key attribute is not set in this table, but inherited from the parent Link table


The following data types apply:
OMD_EFFECTIVE_DATETIME; high precision datetime e.g. datetime2(7), date)
OMD_CURRENT_RECORD_INDICATOR; integer
OMD_EXPIRY_DATETIME
OMD_INSERT_MODULE_INSTANCE_ID
OMD_UPDATE_MODULE_INSTANCE_ID
OMD_DELETED_RECORD_INDICATOR
OMD_SOURCE_ROW_ID
OMD_CHECKSUM
Color schema
The following color schema modelling convention for Data Vault is used:
Table Type
Color
Hub 
Blue
Link
Red
Satellite
Light yellow
Link Satellite
Dark yellow

Kim Vigors fix the colours in table, or insert as pic. Check w/ Roelant Vos
Related Design Patterns
None.
Consequences
Index strategy is documented in dedicated RDBMS Solution Patterns
Discussion items (not yet to be implemented or used until final)
None.

## Motivation



## Applicability



## Structure



## Implementation Guidelines



## Considerations and Consequences



## Related Patterns

- 