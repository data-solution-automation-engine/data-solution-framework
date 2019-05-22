# Design Pattern - Generic - Interfacing to an operational (source) system

## Purpose
This Design Pattern describes the generic requirements, rationale and approach when there is a need to obtain information from a operational (OLTP) system. From the perspective of the Data Warehouse this is considered a 'source' or 'feeding' system.

## Motivation
The interfaces between operational systems and the Data Warehouse is one of the most complex and difficult areas to implement, because the solution are in many cases dependent on various external factors including (but not limited to) funding, security, IT processes and controls and technology. Still, to enable a full audit trail and prevent issues associated with incorrect or incomplete receiving of data there are some fundamental requirements that every interface should comply with. 
The scope of this pattern covers the access and retrieval of data delta (changes) into the Staging Layer of the Data Warehouse, including delta detection concepts such as Change Data Capture (CDC), Change Tracking (CT) and exposure of data via APIs.

The exact technical implementation of these concepts depends on an evaluation of technology options, costs and risk for each individual system. Each technology option is associated with a Solution Pattern.

This area of focus is also known as 'Sourcing' or 'Interfacing' with other systems.

## Applicability
This Design Pattern is applicable for every data that is presented to / loaded in the Data Warehouse (Staging Layer). This usually is a result of a project initiation, information analysis task or change request.

## Structure
The following list captures the fundamental requirements of an interface between an operational system and the Data Warehouse:
* Flexibility in expanding the scope of data access (adding additional artefacts / tables). The selected solution should aim to make it relatively easy to add additional information to the interface. This prevents all data to be staged as part of a delivery, which may incur a maintenance overhead for data that may not (yet) be required. The best balance is usually found when more data can be added to an interface on an iterative basis.
* Support for a pull mechanism (pulling the data delta into the Staging Layer from the Data Warehosue perspective) as opposed to a push mechanism (scheduled extracts from the source / feeding system 'pushing' the data into the Staging Layer). This allows the Data Warehouse to manage the loading frequency (i.e. increase ETL frequency). CDC solutions need to ensure adequate resource governance to prevent impact on OLTP performance.
* Granular access to data (original raw, atomic data as opposed to aggregated information).
* Capturing of all individual transactions (data changes). Some interfaces only allow a present-state view of information to be loaded, this needs to be avoided as this will not provide changes that happened between interface calls. For example, if a record changes 10 times then 10 records need to be received, not just the final state.
* Access to data delta as well as access to the full history of information / data. Full history is required for initial load purposes, but also to support the data recovery pattern that is built-in the ETL templates as a standard functionality. Data recovery allows for period reconciliation checks that provide an additional level of safety for CDC mechanisms (i.e. lost messages, outages that cause transactions to be lost etc.).
* Provided information needs to include at least the Event Date/Time information. This is the moment the data changes were applied in the source / feeding system.
* The receiving system should only be able to access information that it is allowed to use. In other words, security (ring-fencing of data) is a requirement for the interface provided by the source system.
* Source-system specific business logic (at a low / raw data level, including combining codes and references etc.) is provided by the source system as an interface. Care must be taken to avoid redeveloping application logic in ETL solution on the Data Warehouse side. Any application logic needs to be incorporated into the interfaces as managed by the operational system. Application logic in this context can vary from complex calculations to simple joins of reference codes to transactions.
* The interface needs to be able to detect and provide notice of record deletes. The Data Warehouse will store this information as a *logical delete*, which means the data row is understood to be closed in the source system (i.e. the most recent state of the record is 'deleted').

## Implementation Guidelines
* Consider agreeing on a data interfacing contract / agreement (Service Level Agreement) where possible.
* Consider scalability in terms of ability to add more data elements or tables to existing interfaces. In some cases significant effort may be required to add or modify interfaces, and this effort may be limited by increasing the scope of data in a single change. This is especially the case when dealing with third-party systems. It may be better to retrieve all the data in one go as in some cases the first contact is ‘free’, but subsequent efforts to add additional interfaces typically meet more resistance, suffer from politics or require additional funding. A downside to this approach is the extra effort both in terms of maintenance and development to load/ stage (and perhaps integrate) all tables. However, in some cases this trade-off is a positive one in the longer term when no extra communication to the source system owners is required.
* An additional consideration is that information that is captured by the Data Warehouse can be archived in the Persistent Staging Area (PSA), which enables the solution to collect information early on in the development cycle (for use later on).

## Considerations and Consequences
Not applicable.

## Related Patterns
* Design Pattern 006 - Generic - Managing Temporality by using Start, Process and End Dates.