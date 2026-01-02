---
uid: design-pattern-generic-fundamental-data-integration-requirements
---

# Design Pattern - Generic - Fundamental Data Integration Requirements

## Purpose

The purpose of this pattern is to define a set of minimal requirements that every single data logistics process (i.e. procedure, mapping, module, package, data pipeline) should conform to. These fundamental guidelines direct how all data integration processes should behave under the architecture of the Data Solution Framework.

## Motivation

Regardless of its place in the architecture or purpose, every data integration process should be created to follow a distinct set of base rules. Essential concepts such as making sure data logistics processes check if they have already run before inserting duplicates or corrupting data will simplify testing, maintenance and troubleshooting.

The overarching motivation is to develop data integration processes that cannot cause errors due to unplanned or unwanted execution. Essentially, data integration processes must be able to be run and re-run at any point in time to support a fully flexible scheduling and implementation.

## Applicability

This Design Pattern applies to every data integration process.

## Structure

The requirements are as follows:

* **Atomic**; data integration processes contain transformation logic for a single specific function or purpose. This design decision focuses on creating many processes that each address a specific function, as opposed to few processes that perform a wide range of activities. Every data integration process attempts to execute an atomic functionality. Examples of atomic Data Warehouse processes are key distribution, detecting changes and inserting records.
* **Target-centric**; A data integration process can read from one or more sources but only write to a single target. This further supports the atomicity principle.
* **Forward-only**. The direction of data logistics is always towards the target area or layer. It is always 'downstream'. No data integration process should write data back to an earlier layer, or use access this information using a (key) lookup. This implies that the required details are always available in the same layer in the architecture.
* **Idempotent**; each process should execute in a way that it ‘picks up where it left off’. This manifests itself in different ways throughout the data solution architecture. For instance, data integration processes should be able to rerun without the need to manually change settings. All processes must be able to execute without requiring manual intervention, and without causing data loss.
* **Deterministic**; data integration processes must always produce the same outcome based on the same input values. Deterministic processes greatly simplify the overall maintenance and reliability of the solution.
Independent; processes must be able to be executed at any time, and in any order, without any dependencies on other processes. This principle enables flexible orchestration, and addresses any scheduling or frequency constraints.
* **Auditability**; each unique process execution is numbered, identifiable, and can be related to the data that was involved. Using the control framework, and supported by the Persistent Staging Area, the flow of data can always be tracked.
* **Scalable**; data integration processes must be able to process multiple intervals (changes) in one run. Every time a data integration job runs, it needs to process all available data. This is a critical requirement for flexibility in orchestrating data integration processes, and to support near real-time processing. For instance, if the address of an person changes multiple times during the day and the process is run daily, all changes are still captured and correctly processed in a single execution of the process.
* **Fault-tolerant**; data integration processes must be robust and implement a certain degree of resilience. For instance, they should detect if data can be inserted and not fail on constraints. If they do fail, data integration processes should automatically recover ('rollback'). If an error does occur, the process automatically reverts. This is tracked by the control framework and helps to maintain the internal consistency of the solution – an important feature in delivering eventual consistency.

## Implementation guidelines

It is recommended to follow a 'sortable' folder structure to visibly order containers / folders where data logistics processes are stored in a way that represents the flow of data.

An example is as follows:

* 000_\<source systems\>, one for every source.
* 100_Staging_Area.
* 150_Persistent_Staging_Area.
* 200_Integration_Layer.
* 300_Presentation_Layer.

Data logistics processes are recommended to be placed in the directory/folder where they pull data _to_. For instance the data integration logic for ‘Staging to History’ exists in the '150_Persistent_Staging_Area' folder and loads data from the '100_Staging_Area'.

## Considerations and consequences

In some situations adding some of the listed functionality to data logistics processes may seem overkill or perhaps even redundant. This (perceived) additional effort will impact the developing duration.

But in the context of maintaining a generic design (e.g. to support code generation and maintenance) this will still be necessary. Concessions can be made per architectural layer (all processes within a certain architecture step). It is recommended to document any pattern exceptions in the Solution Architecture documentation.

## Related patterns

All  patterns related to data integration, logistics, data logistics/ELT/LETS, and other kinds of automated data movement and interpretation.

