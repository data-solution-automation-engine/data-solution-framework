# Design Pattern - Presentation Layer - Using Views

## Purpose


## Motivation
- 

## Applicability

## Structure

#### Using views for decoupling

The Data Warehouse design incorporates views ‘on top off’’ the Presentation Layer (Information Mart). This is applied for the following reasons:

- Views allow a more flexible implementation of data access security (in addition to the security applied in the BI Layer)
- Views act as ‘decoupling’ mechanism between the physical table structure and the Semantic Layer (business model) 
- Views allow for flexible changing of information delivery (historical views)

These views are meant to be 1-to-1, meaning that they represent the physical table structure of the Information Mart. However, during development and upgrades these views can be altered to temporarily reduce the impact of changes in the table structure from the perspective of the BI platform. This way changes in the Information Mart can be made without the necessity to immediately change the Semantic Layer and/or reports. In this approach normal reporting can continue and the switch to the new structure can be done at a convenient moment.  

This is always meant as a temporary solution to mitigate the impact of these changes and the end state after the change should always include the return to the 1-to-1 relationship with the physical table.

A very specific use which includes the only allowed type of functionality to be implemented in the views is the way they deliver the historical information. Initially these views will be restricted to Type 1 information by adding the restriction of showing only the most recent state of the information (where the Expiry Date/Time = ‘9999-12-31’). Over time however it will be possible to change these views to provide historical information if required. On a full Type2 Information Mart, views can be used to deliver any type of history without changing the underlying data or applying business logic.

#### Using views for virtualisation

Another use case for view is for virtualising the Presentation Layer. As all granular and historic information is stored in the Integration Layer it is possible, if the hardware allows it, to use views to present information in any specific format. This removes the need for ETL – physically moving data – from the solution design. Applicability of virtualisation depends largely on the way the information is accessed and the infrastructure that is in place. Possible application includes when the BI platform uses the information to create cubes, when information is infrequently accessed or with a smaller user base.

## Implementation Guidelines


## Considerations and Consequences


## Related Patterns
