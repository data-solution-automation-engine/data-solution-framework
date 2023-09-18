# Solution Pattern - SQL Server 2008-2016 - CDC and Replication

## Purpose
This pattern documents how SQL Server Replication and native Change Data Capture (CDC) are configured.

## Motivation

Transactional Replication can provide a powerful mechanism to receive data delta. Consistency on how to interpret the information is required to properly integrate the information.

## Applicability

SQL Server systems providing data to a (central) data platform or solution.

## Structure

The aim is to be as less intrusive on the source systems as possible when using Replication and CDC, this is done by:
Configuring a replication Publication Agent on the source system for the required tables and databases. This is a transactional replication.

### Configuring the Distribution Agent (distribution database) on the Data Warehouse server.

Configure the Subscribing Agent on the Data Warehouse server as a pull mechanism for more flexibility (when splitting the location of the Distribution and Subscribing Agent).
The main reason for this configuration is to be as less intrusive as possible for the source system. The services (agents) can later be hosted on other severs than the Data Warehouse server if required (for example in a central distribution hub).
By creating a Subscribing Agent on your Data Warehouse server you automatically create a replicated table, on which you can enable native CDC. 

The resulting database structure is as follows:

* Source database (typically on another server)
* Replicated source database (on the Data Warehouse Server). This database has CDC enabled, and because of this will contain the replicated source but also the log of changes on this source (corresponding CDC table).
  Staging Area database (on the Data Warehouse server) as part of the default ETL Framework (100_Staging_Area). This database will ultimately receive the CDC delta.
  The following diagram shows the overview of this implementation of replication and CDC:

SQL Server’s native CDC functionality reads the transaction log to record changes in system tables associated with each table for which CDC is enabled. It writes those files to system tables in the same database, and those system tables are accessible through direct queries or system functions. 

CDC can be enabled using the available functions in SQL Server:

- Execute sys.sp_cdc_enable_db on the Replicated Source database.
- Execute sys.sp_cdc_enable_table on the table that should have CDC enabled. 

The following minimum parameters are required:

- Source_schema: database schema if available, otherwise ‘dbo’ as the default schema.
- Source_name: the name of the source table.
- Supports_Net_Changes : 1 (enable).

The newly created CDC table is created under the ‘cdc’ schema as part of the System Tables.


## Implementation Guidelines



## Considerations and Consequences

This approach requires changes to the source systems, which may not always be possible or allowed. It has to be verified if you are allowed to install the publisher agent on the source system.

Information about the state of CDC, or disabling the mechanism is available using similar procedures to the creation statement:

EXEC sys.sp_cdc_disable_table
      @source_schema          = N'dbo',
      @source_name            = N'Employee',
      @capture_instance       = N'dbo_Employee'
 EXEC sys.sp_cdc_help_change_data_capture

## Related Patterns

- 