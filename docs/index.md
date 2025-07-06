# Data Solution Framework

A documentation library containing reusable design- and solution patterns.

## Introduction

The Data Solution Framework provides a software and methodology independent, structured approach to developing data solutions. It is a collection of documents that can be uses as a 'recipe' for creating and managing data solutions. The framework contains example solution designs, design patterns, and implementation approaches which can be adopted to create various types of data solutions.

The framework is defined in a modular way, allowing different elements to be selected to suit the needs individual solutions. Individual components can be used in conjunction with each other, or as stand-alone additions to existing data solutions. As such, the framework is agnostic of technology and methodology, and designed to facilitate a platform-independent, flexible and manageable development cycle.

On several occasions, the Data Solution Framework mentions other frameworks, for instance the control framework for data logistics. Although other control frameworks can be adopted, the default option for this is the DIRECT framework as maintained in the [DIRECT repository](https://github.com/data-solution-automation-engine/DIRECT), which is another part of the [engine for data solution automation](https://github.com/data-solution-automation-engine).

## Why use a framework?

*‘If we want better performance we can usually buy better hardware, unfortunately we cannot buy a more maintainable or reliable system’.*

Design and implementation of data solutions can be a labour-intensive activity that often consumes large amounts of effort, time, and cost. In his book “Thank you for being late” (2016), Thomas Friedman describes and explains the drivers behind technology’s increasing rate of change, and how it enables innovation to a point where human adaptability it outpaced.

Over time, as requirements change and demand for data increases, the architecture faces challenges in the complexity, consistency and flexibility in the design (and maintenance) of the data integration processes.

These changes can include latency and availability requirements, a bigger variety of operational systems that generate data, and the need to expose information in different ways. At the same time, data tends to increate in volume, variety, and velocity.

These issues are compounded by an absence of agreed industry best practices; which in turn leads to various ad-hoc patterns being implemented based on an individual's experience (or lack thereof). Implications of (poor) design decisions are often not fully understood, and only become apparent when the investment in time and money has already been done.

The framework aims to provide the means to address these challenges, by consolidating past experiences into standardized and community-managed design patterns. Without a high quality library of patterns, the design, implementation and testing of data solutions is often sporadic, and not sufficient to provide high-quality results.

## How can the framework be used?

The core body of knowledge sits in the various *Design Patterns* (details of specific concepts) and *Solution Patterns* (implementation guides at technical level).

The idea is that Design- and Solution patterns are continuously updated and added. The framework defines a set of possible solution architectures -defined in 'layers' and 'areas'- that can be considered in the design. When adopting the framework, a set of patterns is selected and applied to the architecture - encoded in a Solution Architecture artefact that defines the solution and its reasoning, and which patterns apply.

## Contents

The framework contains the following components:

* **Reference Solution Architecture**; a blueprint for common data solution architectures such as Data Warehouses, Data Hubs, Data Vaults, Data Lakes etc. The corresponding documents outline the various layers and areas that define the data solution.
* **Design Patterns**; documentation of key design decisions and backgrounds on design principles: the conceptual 'how-to's' that capture intent and reasoning. This includes the application of data integration and modelling concepts.
* **Solution Patterns**; the practical details on how to implement Design Pattern concepts explained for a given technology. In many cases a single Design Pattern is referred to by multiple Solution Patterns, all of which document how to implement the concept for a specific technology.

## Standards for Design and Solution Patterns

The pattern structure (Design and Solution Pattern layout) always is as follows:

* **Title**, the name of the pattern
* **Purpose**, a short statement what the pattern is trying to achieve or explain. What is the intent?
* **Motivation**, a short overview of the background and relevance of the pattern. Why is there a need?
* **Applicability**, a listing of where this pattern can be expected to play a role.
* **Structure**, the main section with the pattern details.
* **Implementation guidelines**, any references to how to implement this pattern (Design Patterns only). Note that the Solution Pattern is intended to explain the specifics in a technical context. This is meant to capture any generic topics.  
* **Considerations and consequences**, meant to offer some alternative views and experiences as to what it means to take a certain decision.
* **Related patterns**, any references towards further reading and related content.
