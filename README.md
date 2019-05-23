# Data Integration framework - Overview
## Introduction

The Data Integration framework provides a software and methodology independent, structured approach to developing data processes. 

The framework is designed to facilitate a platform-independent, flexible and manageable development cycle. It covers pre-defined documents, templates, design decisions, implementation approaches, auditability and process control (orchestration approaches).

The framework should not be seen as a one-size-fits all solution; everything is defined in a modular way as much as possible, and different elements can be applied to suit the needs of each individual solution. The Data Integration framework is set up as a variety of components which can be used in conjunction with each other, or as stand-alone additions to existing management information solutions.

The fundamental principle of the framework is to design for change by decoupling 'technical' logic and 'business' logic and ensuring every data integration process can run independently and in parallel - as well as recover at any point in time without impacting dependencies to other processes. 

The framework aims to provide standards for decoupling (functional separation) so new or changed requirements in information delivery can be met without re-engineering the foundations of the data solution. These standards are developed and maintained in MarkDown format on Github to enable collective maintenance of this body of knowledge.

On several occasions, this framework makes mention of an ETL Process Control framework. The default option for this is the DIRECT framework as maintained in the [DIRECT Github](https://github.com/RoelantVos/DIRECT) (private at the moment while being finalised).

## Why need a Data Integration framework?

*‘If we want better performance we can buy better hardware, unfortunately we cannot buy a more maintainable or reliable system’.*

The design and implementation of the data integration is still largely a labour-intensive activity and typically consumes large amounts of effort in Data Warehouse and data integration projects. 

Over time, as requirements change and enterprises become more data-driven, the architecture faces challenges in the complexity, consistency and flexibility in the design (and maintenance) of the data integration flows. 

These changes can include changes in latency requirements, a bigger variety of sources or the need for more parallel processing to a previously rigid serial pipeline. All of this typically occurs when adoption of Data Warehousing and BI matures within an organisation, and having up-to-date information become more and more mission critical.

Using a flexible data integration approach will meet these challenges by providing structure, flexibility and scalability for the design of data flows.

In a more traditional sense, data solutions are often designed to store structured data for strategic decision making, which allows a small number of (expert) users analyse (historical) data and reports. Data is typically periodically extracted, cleansed, integrated and transformed in a centralised Data Warehouse from a heterogeneous set of sources. The focus for ETL in these design is typically on ‘correct functionality’ and ‘adequate performance’ - but this focus misses key elements that are equally important for success. 

These elements, including consistency, degree of atomicity, ability to rerun, scalability and durability are addressed in the Data Integration framework. For example, data solutions may be required to cater for sending back cleansed or interpreted data to the operational systems. They also may need to handle unstructured data next to the structured data, as well as able to quickly respond to changes in (business) requirements. Lastly, they may need to support a ‘feedback loop’ to incorporate changes made by (authorised) end-users in the front-end environments. 

By providing architecture patterns and templates, design decisions and guidelines for error handling and process control the Data Integration framework intends to provide a structured approach to data integration design - for a flexible and affordable development cycle.

## Key contents

The framework contains several high level documents providing an overview of the definitions, intent as well as layers and areas in the architecture.

The core body of knowledge sits in the various *Design Patterns* (details of specific concepts) and *Solution Patterns* (implementation guides at technical level). 

The idea is that Design- and Solution patterns are continuously updated and added to. A typical solution design would select the relevant patterns to define the architecture.

# Data Integration framework components

The diagram below outlines the components that are considered part of the Data Integration framework. These are all considered required to support a data solution and enable Data Warehouse Automation. 

The intent is to enable a standard and structured way for documenting decisions made related to system design and intended operation.

 ![1547519339316](C:/Files/Data_Integration_Framework/Images/5C1547519339316.png)

   

- **Reference Solution Architecture**; a blueprint for a common Data Warehouse / Information Hub architecture. The corresponding documents outline the various layers and areas that form the data solution.
- **Reference Technical Architecture**; capturing the common technical requirements relevant to the Solution Architecture. The intent for this template is to capture the infrastructure and software specifics, as well as context around the physical data models and database / data platform configuration. The Technical Architecture also covers details around the implementation of security, encryption and retention approaches.
- **Design Patterns**; documentation of key design decisions and backgrounds on design principles: the 'how-to's'. This includes the application of data integration and modelling concepts. Design Patterns follow a defined template and are centrally stored and managed.
- **Solution Patterns**; the practical details on how to implement concepts explained in a Design Pattern for a given technology. Similar to Design Patterns, the Solution Patterns all follow the same template. In many cases a single Design Pattern is referred to by multiple Solution Patterns, all of which document how to implement the concept for a specific technology.
- **Documentation templates**, standards and conventions; modelling and technical conventions.
- **ETL templates**; technical templates that can be used as blueprints to generate data integration processes with or against.
- **ETL mapping metadata**; approaches for managing the source-to-target mappings - vital ETL metadata to enable Data Warehouse Automation / ETL generation.
- **ETL process control framework**; this is the runtime execution, logging and monitoring of data integration processes, including recovery and orchestration. This is further detailed in the DIRECT Github (Data Integration Runtime Execution and Control framework). DIRECT includes a repository for ETL control, integration hooks for ETL processes and automation scripts.

In short, the reference Solution Architecture and the corresponding Technical Architecture provide a common framework for all design and development effort. Design Patterns provide the details of how selected concepts are approaches, including considerations and pros and cons. Solution Patterns describe how these approaches are best translated in the selected technology.  

# Standards for Design and Solution Patterns

The pattern structure (Design and Solution Pattern layout) always is as follows:

* **Title**, the name of the pattern
* **Purpose**, a short statement what the pattern is trying to achieve or explain. What is the intent?
* **Motivation**, a short overview of the background and relevance of the pattern. Why is there a need?
* **Applicability**, a listing of where this pattern can be expected to play a role.
* **Structure**, the main section with the pattern details.
* I**mplementation guidelines**, any references to how to implement this pattern (Design Patterns only). Note that the Solution Pattern is intended to explain the specifics in a technical context. This is meant to capture any generic topics.  
* **Considerations and consequences**, meant to offer some alternative views and experiences as to what it means to take a certain decision.
* **Related patterns**, any references towards further reading and related content.

The Title is in Header 1 format, the sections are in Header 2 format.
