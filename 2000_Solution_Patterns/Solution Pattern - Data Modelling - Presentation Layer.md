# Solution Pattern - Data Modelling - Presentation Layer

## Purpose
This Implementation Pattern describes the data modelling conventions and architecture for the Presentation Layer.

##Structure
In principle, there are two mechanisms towards preparing information for consumption in the Presentation Layer (created in the Presentation Layer database):
* Direct view on top of the Integration Layer (virtual information mart). In by far the most scenarios the first option (direct view / virtual) option is preferred as the subsequent layers in the (BI) architecture are typically MOLAP or in-memory.
* Table / persistence / physical storage using a view to join and prepare the data in the format that matches the table (logic) and can be used to incrementally load the table.

In both cases these Presentation Layer objects will require one or more views to decouple the Business Intelligence (BI) and Data Warehouse (DWH) environments. These decoupling views are also intended to apply the history perspective at attribute level; e.g. how every attribute is displayed in time (e.g. Type1, Type2, Type 6).
The following are general guidelines:
The logic views and tables contain all history (Type2) by default. Any interpretation of history, such as ‘current state view’, can be queried using the decoupling views.
There may be multiple decoupling views, but two views is the guideline, to represent history using different perspectives (e.g. current state / Type1 or mixed / Type2).
The decoupling views are faced towards the BI / business side and aims to present information in the way it is easiest to consume.
The decoupling views should be generated from metadata, and use Extended Properties defined at the view logic view to generate an accurate representation of history.
The logic views used to populate tables are geared towards ETL and follows the rigor of naming conventions to support automation. BIML scripts are available to generate SSIS packages from the logic view to the Presentation Layer tables.
The pres schema is the enterprise information / mart schema and contains the decoupling views, so this contains what is effectively the complete dimensional model exposed to the BI environment. This also allows the decoupling views to have the same name as the accompanying Dimension or Fact table.
The ben schema contains the logic views and tables since objects (views and tables in this case) cannot be named identically (as they would be in the pres schema). The views are named with the ‘_VW’ suffix.
Normal casing is used, with underscores (no spaces) for all tables and attributes.
Definitions are maintained in the Confluence Business Glossary.
Logic views are primarily manually developed (with some history merge scripts to assist) as these views handle the change from data handling to business use.
Tables and decoupling views are generated from metadata.
Decoupling views are used to expose history using additional metadata (‘extended properties’).
The extract schema is used for data provision to support external systems (e.g. non-BI) and is therefore considered not to be part of the standard Presentation Layer.
There is also a va schema which is specifically there to expose information to SAS Visual Analytics.
There also is a temp schema which is strictly only used to store ETL required information / to support the performance and workings of the ETL.
This is displayed in the following diagram:

The modelling conventions for the Presentation Layer tables are outlined in the table below. 
Table Type
Table name convention
Mandatory attribute
Comments
Dimension
ben.DIM_<name>
table name>_SK
OMD_INSERT_MODULE_INSTANCE_ID
OMD_DELETED_RECORD_INDICATOR
OMD_UPDATE_MODULE_INSTANCE_ID
OMD_CHECKSUM_TYPE_1
OMD_CHECKSUM_TYPE_2
OMD_EFFECTIVE_DATETIME
OMD_EXPIRY_DATETIME
OMD_CURRENT_RECORD_INDICATOR
<attributes>
The first attribute (SK) is the primary key, and is a hash value (32 byte character)
Optionally, a unique key / index is placed on the combination of level natural keys and the OMD_EFFECTIVE_DATETIME. This represents a unique point in time record. See the consequences section for more details
Every attribute is specified as Type 0, Type 1, Type 2 (can be combined to type 3 or 6 - check the relevant pattern). This is specified in the model / database as an extended property
Fact Table
ben.FACT_<name>
<table name>_SK
<Dimension Keys>
OMD_INSERT_MODULE_INSTANCE_ID
OMD_INSERT_DATETIME
OMD_INSERT_MODULE_INSTANCE_ID
OMD_DELETED_RECORD_INDICATOR
OMD_UPDATE_MODULE_INSTANCE_ID
OMD_CHECKSUM_TYPE_1
OMD_CHECKSUM_TYPE_2
OMD_EFFECTIVE_DATETIME
OMD_EXPIRY_DATETIME
OMD_CURRENT_RECORD_INDICATOR
<attributes>
The first attribute (SK) is the primary key
A unique key / index is placed on the combination of Dimension keys.
Other
ben.<name>
<table name>_SK
OMD_INSERT_MODULE_INSTANCE_ID
OMD_INSERT_DATETIME
<any OMD attributes required>
<attributes>
Not every delivery of information is necessarily in the form of a Star Schema / Dimensional Model. If a dataset is better delivered in a different format (wide table, normalised) this is preferred.
 
The modelling conventions for the Presentation Layer views are outlined in the table below. 
View Type
Table name convention
Mandatory attribute
Comments
Logic View
ben.<name>_VW
<view name>_SK
OMD_INSERT_MODULE_INSTANCE_ID
OMD_INSERT_DATETIME
OMD_INSERT_MODULE_INSTANCE_ID
OMD_DELETED_RECORD_INDICATOR
OMD_UPDATE_MODULE_INSTANCE_ID
OMD_CHECKSUM_TYPE_1
OMD_CHECKSUM_TYPE_2
OMD_EFFECTIVE_DATETIME
OMD_EXPIRY_DATETIME
OMD_CURRENT_RECORD_INDICATOR
<attributes>
Used to load a standard Dimension or Fact table supported by BIML.
The name of the view needs to match the name of the target table (except for the _VW suffix)
The _VW suffix is required as there may be a table with the original name in the ben schema
The checksums for Type 1 and Type 2 calculations will be handled by the BIML, and do not need to be present in the views. This allows for a more automated update if required
All other OMD attributes required in the target table are handled by the BIML scripts
Decoupling View
pres.<name> (for regular views)
pres.<name>_history (for history or mixed-history views)
Underlying ‘ben’ table or logic view, but without OMD attributes.
Surrogate keys optional.
Business-facing, e.g. DIM_CUSTOMER, or DIM_CUSTOMER_HISTORY.
 
Related Design Patterns
Design Pattern 002 - Generic - Types of History
Consequences
Related to having Data Vault Surrogate (Hub) Keys (SK) in the Dimensional Model: it is OK to add Hub keys (Surrogate Keys) in the Presentation Layer for tracing and auditability purposes. However they cannot be adequately used as level keys as a level in a Dimension may not 100% map a business concept. For instance a 'business unit type' may not be modelled as a Hub in the Data Vault, but could be a level in a Dimension. By using Hub Keys for Dimension lookups a dependancy between the Integration and Presentation Layers is created that should be avoided. An example is where you have Business Unit Type, State, Counter and Ownership in the same Satellite (e.g. SAT_BUSINESS_UNIT). If these attributes are modelled in separate Dimensions in the Presentation Layer the Hub Key (from HUB_BUSINESS_UNIT) cannot be used, rather a separate Dimension Key must be created and a dedicated natural key must be selected appropriate for the dimension. In other words, lookups and constraints should be using natural keys.
Discussion items (not yet to be implemented or used until final)
None.