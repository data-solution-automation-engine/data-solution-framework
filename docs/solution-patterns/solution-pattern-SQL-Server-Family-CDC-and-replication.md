---
uid: solution-pattern-sql-server-family-cdc-and-replication
---

# Solution Pattern - SQL Server Family - CDC and Replication

## Purpose

This pattern documents how SQL Server Replication and native Change Data Capture (CDC) are configured.

## Motivation

Transactional Replication can provide a powerful mechanism to receive data delta. Consistency on how to interpret the information is required to properly integrate the information.

## Applicability

SQL Server environments providing data to a (central) data platform or solution.

## Structure

The intent is to be as less intrusive as possible for the operational system when using Replication and CDC. This can be achieved by configuring transactional replication from the operational system to the data solution environment, and applying native CDC there. This way, the data solution 'pulls' changes in the operational system to a local copy.

## Implementation guidelines

To implement this. Configure the Subscribing Agent on the data solution server, as a pull mechanism for more flexibility (when splitting the location of the Distribution and Subscribing Agent). The main reason for this configuration is less intrusive as possible for the operational system. The services (agents) can later be hosted on other severs than the data solution server if required (for example in a central distribution hub).

By creating a Subscribing Agent on your data solution server you automatically create a replicated table, on which you can enable native CDC.

The resulting database structure is as follows:

* Source database (typically on another server).
* Replicated source database (on the data solution server). This database has CDC enabled, and because of this will contain the replicated source but also the log of changes on this source (corresponding CDC table).
* Landing Area database (on the data solution server) as part of the default data logistics Framework (100_Staging_Area). This database will ultimately receive the CDC delta.

SQL Server's native CDC functionality reads the transaction log to record changes in system tables associated with each table for which CDC is enabled. It writes those files to system tables in the same database, and those system tables are accessible through direct queries or system functions.

CDC can be enabled using the available functions in SQL Server:

* Execute `sys.sp_cdc_enable_db` on the Replicated Source database.
* Execute `sys.sp_cdc_enable_table` on the table that should have CDC enabled.

The following minimum parameters are required:

* Source_schema: database schema if available, otherwise `dbo` as the default schema.
* Source_name: the name of the source table.
* Supports_Net_Changes : 1 (enable).

The newly created CDC table is created under the CDC schema as part of the System Tables.

Information about the state of CDC, or disabling the mechanism is available using similar procedures to the creation statement:

```sql
EXEC sys.sp_cdc_disable_table
      @source_schema          = N'dbo',
      @source_name            = N'Employee',
      @capture_instance       = N'dbo_Employee'
 EXEC sys.sp_cdc_help_change_data_capture
```

## Considerations and consequences

* A preference exists for pull subscriptions to reduce source impact; host the distributor close to the subscriber for network resilience. Separate publisher/distributor roles if the source is sensitive.
* This approach requires changes to the source systems, which may not always be possible or allowed. It has to be verified if you are allowed to install the publisher agent on the source system.
* Create a transactional publication with only required articles; avoid unnecessary columns and tables. Use row filters where appropriate to reduce change volume.
* Configure the distribution database with appropriate retention and cleanup jobs; monitor agent latency. On the subscriber (data platform), enable CDC on replicated tables to capture changes locally.
* After replication, enable CDC per database and per article using `sys.sp_cdc_enable_db` and `sys.sp_cdc_enable_table`. Set `supports_net_changes = 1` when downstream needs net change queries. Align capture jobs with source transaction volume.
* Use `cdc.fn_cdc_get_all_changes_<capture_instance>` or net change functions to extract deltas into the Landing Area. Maintain watermarks and process windows deterministically.
* Ensure source log backups and disk I/O can handle CDC; size distribution and CDC tables appropriately. Monitor VLF count to avoid log fragmentation impacting latency.
* Restrict replication and CDC roles; avoid running agents with excessive privileges. Encrypt connections between publisher, distributor, and subscriber if required.
* Set up alerting on replication agent failures, latency thresholds, and CDC job failures. Document failover/runbook steps, including how to reinitialize subscriptions and resync CDC.
* Plan for DDL on published tables; add articles or reinitialize carefully to avoid data loss. When columns are added, ensure CDC capture instance includes them or create a new capture instance as needed.

## Related patterns

* [Design Pattern - Generic - Interfacing from an operational system](xref:design-pattern-generic-interfacing-from-an-operational-system).