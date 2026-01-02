# Design and Solution Patterns

A curated library of reusable design patterns and solution patterns that support the [Data Engine Thinking](https://dataenginethinking.com/en/) approach (as described in the book).

In Data Engine Thinking, a data solution is defined by the architecture you choose and the patterns you apply:

* Design patterns capture concepts, decisions, and best practices (the 'what' and 'why', supported by guidance on how to apply it in a technology-agnostic way).
* Solution patterns provide implementation guides for a specific stack or environment (the 'how', in a concrete, technical form).

These patterns are intended to evolve continuously: improved, expanded, refined, and added as new lessons and practices emerge.

> [!NOTE]
> This library is intentionally 'living'. If you spot gaps, have improvements, or want to add a new pattern, we’d love your help. Please submit a Pull Request with your proposed changes. Small edits (typos, clarity, examples) are just as valuable as new patterns.

## How this library fits into Data Engine Thinking

Data Engine Thinking provides a structured, software- and methodology-independent way to design and manage data solutions. It offers a set of reference architectures—defined in layers and areas—that you can use to reason about responsibilities, boundaries, and trade-offs.

When adopting the framework:

1. You select an architectural direction (layers/areas) that fits your context.
1. You select the patterns that apply to that architecture.
1. You document the outcome as a Solution Architecture artefact: the chosen structure, rationale, and the patterns applied.

This repository supports step 2 (and helps make step 3 repeatable).

On several occasions, the patterns mention other frameworks, for instance the control framework for data logistics. An example of this is the DIRECT framework as maintained in the [DIRECT repository](https://github.com/data-solution-automation-engine/DIRECT).

## Design Patterns

A design pattern describes a concept and how to apply it. It is the primary tool for a data solution architect to design, reason about, and govern the solution over time.

Design patterns typically include:

* Purpose and context (when to use it, and why).
* Options and trade-offs.
* Recommended approaches and decision points.
* Consequences and operational considerations.

Design patterns are technology-agnostic by default so they remain relevant as platforms change. Where unavoidable, they may include technical notes—but the intent is to avoid locking the pattern to a single vendor or implementation.

Examples include: essential data logistics requirements, handling logical deletes, key distribution, temporal/time-variant modelling, and methodology-specific patterns such as loading contextual data in Data Vault.

## Solution Patterns

A solution pattern is an implementation guide for a specific technology stack or environment. Where a design pattern defines the concept and the intent, a solution pattern defines the concrete execution.

A single design pattern can have multiple solution patterns, for example:

* One per database engine.
* One per orchestration platform.
* One per infrastructure approach.
* Variations driven by performance, scale, security, or operational constraints.

This separation helps you get the best results from each technology investment without rewriting the conceptual foundation every time a stack changes.

In Data Engine Thinking, we don't advocate a single universal implementation. Instead, we separate the what (design patterns) from the how (solution patterns) so the architecture remains stable while implementation can evolve.
