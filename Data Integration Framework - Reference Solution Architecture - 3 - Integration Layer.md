#  Integration Layer overview

The Integration Layer is the second layer in the reference Data Warehouse solution architecture. This Layer is not designed to be accessible by (end) users of the information but serves as the true Data Warehouse Layer, where information is maintained in such a way that it is both resilient and flexible. The Integration Layer sources its information from the Staging Area and stores it in a consistent and atomic way, without applying business logic. This data can then be presented in a consumable form in the Presentation Layer.

This document defines the Integration Layer and describes the involved steps and techniques. Various references to other documents of the ETL Framework will be made, including error handling and metadata management. Ultimately, this document provides the information to configure the Integration Layer in a flexible and modular way.

The design and approach for modelling the Integration Layer is a project specific decisions which needs to be captured in the Solution Architecture (including supporting reasoning). Examples are 3NF or Data Vault approaches, but other hybrid techniques can be applied as well as long as they are in sync with the framework guiding principles.

If the Solution Architecture for a project is defined as ‘2-tiered’ – the classic Kimball approach – the Integration Layer is not implemented.

The Integration Layer, or the process from staging to integration, is comprised of two parts (or areas): the Integration Area and the Interpretation Area. The Integration Layer is a persistent Layer.

The Integration Area is the phase where data from the Staging Layer is re-modelled and changes in attributes are captured and tracked using the Slowly Changing Dimension (SCD) 2 technique. Surrogate keys for new records are also identified and assigned prior to the loading of the attributes.

An optional step is then available for the data to be manipulated and for business rules (or logic) to be applied. Data in the Integration Area can be de-duplicated, standardised, validated and/or cleansed, and stored away in the Interpretation Area. This design provides a flexible approach to data cleansing and interpretation at a granular level, while still keeping a flexible yet consistent design. 

This is because in the Integration Area the data remains largely pristine and nothing gets changed or altered. In doing so, it offers the flexibility of presenting the data in more than one way, while the original data remains unchanged. Auditing of data is also made easy as original image of data (obtained from source system) is retained and therefore traceable.

As documented in the detailed architecture overview, data from the Staging Area is loaded into an Integration Area with the option of being further transformed into an Interpretation Area. While each area serves a specific purpose, the underlying data model / structure in both the Integration and Interpretation Areas remain the same. 

In the Integration Area, data from the Staging Area is first processed by identifying new business keys and assigning surrogate keys for newly discovered business keys. This management of Data Warehouse keys is vital and it is the first ETL process that happens in the Integration Area. Data (from staging) is then further processed by loading them into the remaining integration (hybrid) model tables. Changes to the historical data are tracked by means of using Slowly Changing Dimension Type-2 (SCD2) approach. Effectively, this decouples key distribution from managing history.

It is worthwhile to note that data captured in the Integration Area remains in its pristine, raw form. Data taken from Staging Area is modelled to support the Data Warehouse, but the content does not change. 

In the Interpretation Area, enterprise wide business rules are applied to the data residing in the Integration Area. This is done to accomplish any of the following objectives:

- Data cleansing / scrubbing
- Data enrichment
- Data standardisation
- Deduplication
- Validation

The intention of the Interpretation Area is to reduce replication of the rules in ETL processes towards the Presentation Layer. An example would be the formatting of addresses where abbreviations such as “st” or “rd” get expanded to “street” and “road”, and the validation of phone numbers. It also could be combining various data elements from Human Resource related tables into Payroll information.

In separating the state of the data into two physical forms (original and modified), it gives the flexibility of applying multiple sets of rule to a specific data set, in order to suit specific needs. 

## Integration Layer

The Integration Layer consists of the **Raw Data Vault** area and the **Business Data Vault** area. The main purpose of this Layer is to function as the ‘core Data Warehouse layer’ where all the data is collected in a normalised Data Warehouse model. To achieve optimal flexibility and error handling business rules are implemented as late as possible in the ETL process. 

The Raw Data Vault stores the source data without changing the contents in the core Data Warehouse model. The system collects the data from all source systems in a generic way which is suitable for further expansion. The main Data Warehouse functionalities such as surrogate key distribution, storing history and maintaining relationships are done in this area.  

The Business Data Vault uses the same modelling standards as the Raw Data Vault but provided interpretations or alternate views on the granular data. Both areas link closely to each other and in most cases provides separate cleaned or changed instances of tables that already exist in the Raw Data Vault. 

The Business Data Vault is not a full copy of the Raw Data Vault. In most cases the Interpretation Area tables will refer to Integration Area surrogate key tables and provide an alternative perspective to Integration Area historical tables. 

Examples of logic that can be applied in the Business Data Vault are generic business rules such as de-duplication or determining a single customer view. Additionally, the Business Data Vault is also used to design cross-references between similar datasets from different source systems. These cross-references are essentially recursive or intersection entities between business entities in the Raw Data Vault, but contain (business) rules to identify the main keys. 

The important factor is that in this layer, business rules that alter the contents of the data are not yet applied. In the case of derivations, for example in the Business Data Vault, this means the original values will always need to stay available. Also, records are not checked for errors to keep the system as flexible as possible towards the Information Marts.  

The Integration Layer (Integration Area and Interpretation Area) will be created using a **Data Vault 2.0** model which decouples key distribution using main entities (Hubs) but de-normalises reference information (Satellites) for these entities. Relationships between the main entities (Links) can be managed and tracked over time. This is a loosely-coupled data modelling approach which reduces dependencies and timing issues which are expected to occur in the data delivery. 

As an example, this approach allows information related to the same customer or prospect to be delivered and integrated independently. It also supports ongoing linking of customer information to tie in various elements of information to the unique prospect or customer over time without losing flexibility; the logic for de-duplication can be changed and/or recalculated across historical information if required.

![1547521558900](D:/Git_Repositories/Data_Integration_Framework/Images/558900.png)

Objects in the Integration Layer are not accessible for end-users or Business Intelligence and analytics software. This is because for most scenarios information has not yet been prepared for consumption; only Data Warehouse logic is implemented.  There is an exception to this rule; for specific data mining or statistical analysis it is often preferable for analysts to access the raw / unprocessed data. This means this access can be granted for the Integration Layer which contains essentially raw, but indexed and time variant data in the right context (e.g. related to the correct business keys). This is an ideal structure for statistical analysis.

## Principles

The Integration Layer can be modelled using a hybrid (Data Vault, Anchor Modelling) technique. For the Enterprise Data Warehouse, which integrates many sources and is subject to change a Data Vault 2.0 approach is adopted

# The Integration Area

The Integration Area is modelled differently from the Staging Area. In the Integration Area data is divided into common entities which form the core of the Data Warehouse model. Various modelling techniques can be applied for the Integration Layer (3NF, Data Vault, Anchor) as long as the same technique is used for both areas. Regardless of the approach the tables in the Integration Area are either:

- Entity / Surrogate key tables. The function of a surrogate key table is to create a unique list of instances for that particular entity. For example: all employee numbers. The source key / logical keys are the only attributes which are copied from the Staging Area. This includes 3NF key tables (if applied this way), Data Vault Hub tables and Anchor tables
- Other tables. The rest of the model depends on the chosen modelling technique, but examples of other Integration Area tables are:
  - History or reference type tables which contain all the attributes but the logical key. These tables inherit the surrogate key of the main entity. This is the most common other entity type and will be used as an example. This includes Data Vault Satellite tables, 3NF History tables and Anchor attributes.
  - Relationship tables which contain (information about) the relationships between main entities. This includes 3NF intersection tables and Data Vault Link tables

## Surrogate Key table structure

The following metadata attributes are mandatory for the Surrogate Key tables:

| **Column Name**               | **Data Type**                | **Reasoning**                                                |
| ----------------------------- | ---------------------------- | ------------------------------------------------------------ |
| <entity>_SK                   | INTEGER or CHAR(32) /   Hash | The Data Warehouse key; an unique identifier and also the primary key   which is issued for each record in the table. It can be a meaningless key   (sequence) or hashed value |
|  | INTEGER                      | Default; logging   which process has inserted the record |
|        | DATETIME    (high precision) | This is the time that the   record has been presented to the Data Warehouse environment. This is not the system date/time for insert however, but the original processing time for the   records to be loaded into the Staging Area.    The Insert Date/Time is   the conceptual Event Date/Time; the date time when the source event was   triggered or the change in the source has taken place. It can be the moment a   user updated a record in a source system, or the trigger which caused a   message to be sent. |
| Record Source Id          | INTEGER                      | The relation to the ETL process control table which contains the identification of the source system that originally supplied the information. |
| <business key>                | Depending                    | The business key value                                       |

The following attributes are optional for the Surrogate Key tables depending on the approach for Data Modelling:

| **Column Name**               | **Data Type**                | **Reasoning**                                                |
| ----------------------------- | ---------------------------- | ------------------------------------------------------------ |
| Effective date / time        | DATETIME    (high precision) | Start of the validity period for the record. Equal to the Load Date / Time; this is not the  ystem date/time, but the information recorded during the Staging Area ETL process. |
| Expiry date / time         | DATETIME    (high precision) | The date time when the record was closed. Records are closes based on changes in the history (alteration or deletion). The value of this attribute is the value of the valid start date time of the previous related. The default value is 99991231   23:59:59. |
| Current record indicator  | VARCHAR(100)                 | The flag (Y/N) whether this record is active. This makes selection and querying easier, but is essentially twice redundant. If possible use the Expiry Date/Time for this purpose. |
| ETL Process control Id | INTEGER                      | The module ID of the ETLvprocess which has updated the record. |

 The use of a ‘valid period of time’ (start and end date time) including the current record indicator is optional. There can be sound reasons for including these metadata attributes in a surrogate key table when source systems can reuse their own keys and specific logic has to be created to determine if a reused key is in fact a new instance of an entity or that an old one has been reopened.

## History table structure

The following metadata attributes are present in the rest of the integration area tables. This does depend on the chosen modelling technique. For this paragraph history or reference type tables are used as an example. 

The following metadata attributes are mandatory for the history tables:

| **Column Name**               | **Data Type**                | **Reasoning**                                                |
| ----------------------------- | ---------------------------- | ------------------------------------------------------------ |
| <entity>_<key>         | INTEGER or CHAR(32) /   Hash | The Data Warehouse key; an unique identifier and also the primary key   which is issued for each record in the table. It can be a meaningless key (sequence) or hashed value. This is inherited from the parent table as   Foreign Key |
| Effective Date / Time        | DATETIME    (high precision) | Start of the validity period for a record. Populated by the Load Date / Time value from the Staging Area this is not the system date/time but the information recorded during the Staging Area ETL process. |
| ETL process control Id | INTEGER                      | Default ETL process control attribute for any table for logging which process has inserted the record. |
| ETL process control Id | INTEGER                      | The module ID of the ETL process which has updated the record. |
| Record Source Id          | INTEGER                      | The relation to the ETL process control table which contains the identification of the source system that originally supplied the information. |
| Source Row Id             | INTEGER                      | Copied from the Staging Area. The combination of ETL process control Id and Source Row Id always relate back to a single History Area record |
| Deleted Record Indicator  | VARCHAR(100)                 | This flag (Y/N) indicates that the record has been deleted from the source system. |

 The following attributes are optional for the history tables in the Integration Layer: 

| **Column Name**              | **Data Type**                | **Reasoning**                                                |
| ---------------------------- | ---------------------------- | ------------------------------------------------------------ |
| Expiry Date / Time          | DATETIME    (high precision) | The date time when the   record was closed. Records are closes based on changes in the history   (alteration or deletion). The value of this attribute is the value of the   valid start date time of the previous related record minus 1 second. The   default value is 99991231 23:59:59. |
| Current Record Indicator | VARCHAR(100)                 | The flag (Y/N) whether   this record is active. This makes selection and querying easier. |
|                              |                              |                                                              |
| Hash Full Record         | CHAR(32)                     | A checksum for record   comparison requires storing a checksum value as an attribute. |

In history tables the Primary Key is composed of the <entity_SK> and the Expiry Date / Time attributes.  

The optional attributes include all reference data which relates to the entity Data Warehouse key. In the example of an employee record the person ID would lead to the generation of a new surrogate key, while all descriptive attributes are placed in the history table. Depending on considerations regarding volume or width of the table (in terms of records, bytes) different history records can be placed in different history tables, but always with the same structure as described in the above table.

## Relationship table structure

The relationship table structure is largely dependent on the applied modelling technique, but in the most concise definition contains the following attributes:

| **Column Name**                                | **Data Type**                | **Reasoning**                                                |
| ---------------------------------------------- | ---------------------------- | ------------------------------------------------------------ |
| <relationship>_<key>                              | INTEGER or CHAR(32) /   Hash | The Data Warehouse key; an unique identifier and also the primary key   which is issued for each record in the table. It can be a meaningless key   (sequence) or hashed value |
| <entity>_<key> (one   side of the relationship)   | INTEGER or CHAR(32) /   Hash | A unique identifier; the Data Warehouse key obtained from the   Surrogate Key table. |
| <entity>_<key> (other   side of the relationship) | INTEGER or CHAR(32) /   Hash | A unique identifier; the Data Warehouse key obtained from the   Surrogate Key table. |
| ETL Process Control id                  | INTEGER                      | Default; logging   which process has inserted the record |
| Load Date / Time Stamp                        | DATETIME    (high precision) | This is the time that the   record has been presented to the Data Warehouse environment. This is not the   system date/time for insert however, but the processing time for the records   to be moved into the Staging Area. |
| Source Row Id                         | INTEGER                      | The relation to the ETL process control table which contains the identification of the source system that originally   supplied the information. |

 

This structure essentially links two types of information together, and can be expanded to include relationship attributes or relations to history or properties specific for that relationship (including transactions or historical facts). 

## Integration Area relation to error handling

Once configured and tested properly there are almost no possible technical errors in the Integration Layer (except infrastructure issues). As all staging tables that contain business keys get processed or pre-processed in the Integration area, failure in the lookup of a business key against a surrogate key (SK) table can never occur. 

In the event of timing issues (early arriving facts or later arriving dimensions) where factual data gets processed before dimensional data, pre-processing of the fact data will result in the business key itself being added to the surrogate key table. This means every workflow/Batch requires the capability to issue surrogate keys itself. 

This record in the surrogate key table acts as a placeholder and can optionally be identified as being erroneous or incomplete. This is optional since the metadata attributes enable the system to view which interface has loaded the data. If this is deemed a serious error then the error bitmap is available to filter the records at the presentation layer if there is a requirement to do so. 

## Integration Area development guidelines

- Source keys, also known as business keys, natural keys or logical keys usually lead to surrogate key tables. A composite key might indicate separate surrogate key entities, but not necessarily so. Also, a business key might be composed of more than one source attribute
- If the ETL platform allows it, prefix the ‘area’ or ‘folder’ in the ETL tool with ‘200_’ because this is the first area in the second layer in the architecture. This forces most ETL tools to sort the folders in the way the architecture handles the data, making the development environment better readable
- The Integration Area folder or module in ETL contains all table definitions, all mappings loading from any source (folder) to the integration area folder and all relevant objects for this process
- The Integration Area only loads data from the Staging Layer
- Everything is copied as-is, no transformations are done other than formatting data types. This may never lead to errors! Errors can only be caused by incorrect dependencies (i.e. loading history before distributing surrogate keys), this is not something that can occur in a properly tested environment
- The Staging Area may contain multiple loads of data which could do not necessarily have arrived in the correct order (messaging!) data has to be sorted based on the conceptual Event Date/Time before further processing

# The Interpretation Area

The structure of the Interpretation Area closely follows the conventions of the Integration Area; the same modelling technique is used and the same type of tables, metadata attributes and rules apply. Because of this similarity, only the differences will be documented. 

The Interpretation Area is an optional area where enterprise wide business rules can be effectuated and only sources its data from the Integration Area. From an ETL perspective there is no set ‘pattern’ to load data into the Interpretation Area. Because all data is derived (and therefore redundant) this area can even be virtualised and/or dropped and recreated. Additionally, 3rd party matching or fuzzy logic software can be implemented in addition to standard ETL processing.

 The only real difference is that for a certain table not every sibling has to be present in this area. Depending on the applied business rules the Interpretation Area dataset can be a subset of the integration layer dataset, or a fully updated one. 

The Interpretation Area contains derived data from the Integration Area and this can be done for specific tables or for complete or mixed entity sets. For instance, if only cleansing of a specific history table is necessary it will be sufficient to only store a new derived history table which still links to the integration area surrogate key table. If advanced algorithms like deduplication / match and survive are used this might require the creation of a new main entity with both a surrogate key table, history table and perhaps a relationship table.
