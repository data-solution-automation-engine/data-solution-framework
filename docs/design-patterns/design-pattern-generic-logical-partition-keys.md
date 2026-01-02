---
uid: design-pattern-generic-logical-partition-keys
---

# Design Pattern - Generic - Logical Partition Keys

> [!WARNING]
> This design pattern requires a major update to refresh the content.

## Purpose

This design pattern describes how to handle large data volumes by using logical partition keys. It is a technique which may help loading large datasets faster; an alternative approach to handling large data volumes.

## Motivation

A common challenge encountered in Data Warehousing is how to handle large volumes of data. Typically, limited batch windows, restricted source extraction rules, large and complicated data sets among others contribute to long data logistics processing times. This design pattern aims to reduce and optimize data logistics processes for big data sets.

Also known as:

* Logical data partitioning.
* Horizontal partitioning.

## Applicability

This pattern applies to all data logistics processes that load big data sets. The technique described in this design pattern is generic and can be applied using any data logistics tool.

## Structure

The central point in logical partitioning is to find ways to break down large data sets into logical groupings which can be processed independently from other data groupings coming from the same source. The possibility to logically slice data into multiple data sets and process them independently will improve flexibility in arranging the jobs to run in sequence or in parallel. This depends on the batch window allowed to process data and the overall available time to load the entire data set in the Data Warehouse.
The first step in using the logical partition technique is to identify the key that can be used to create the partition key that will logically group the data together. It is ideal if the source system can provide the partition key that can be used to load the data into stream. This will enable the partitioned load at the beginning when the Landing Area is populated. However, if this is not an option, the partition keys can be assigned to the source file via a script in the file system before loading in the Landing Area.
Another option is to introduce the partition key as data is loaded in the Landing Area. It is recommended that you implement the logical partitions at the earliest time possible to take advantage of the streamed loading process at the early phases of loading.
The attribute you nominate as the partition key should have the following characteristics:
Even distribution of number of rows based on the partition key.
Ideally, the nominated key that will determine the partition key should be   the primary key or form part of the primary key.
Numeric data type (if possible).

In the example below, the source file CUSTOMER.txt contains the following:

| Customer Key | Score Type | Score |
|--------------|------------|-------|
| 123456       | Churn      | 3.7   |
| 222222       | Churn      | 4.1   |
| 333333       | Churn      | 3.5   |
| 987659       | Churn      | 2.9   |
| 1234567      | Churn      | 8.7   |


The sample source file CUSTOMER.txt contains scores for all customers. In the example above, the last digit in the customer key field is used as the partition key. The partition key column is added before loading into Landing Area. This will facilitate reading of data in streams from the Landing Area.
The table that contains the customer data will be contain the partition key column derived from the last digit of the customer key.

| Customer Key | Score Type | Score | Partition Key |
|--------------|------------|-------|----------------|
| 123456       | Churn      | 3.7   | 6              |
| 222222       | Churn      | 4.1   | 2              |
| 333333       | Churn      | 3.5   | 3              |
| 987659       | Churn      | 2.9   | 9              |
| 1234567      | Churn      | 8.7   | 7              |

If the number of rows is still voluminous even after streaming them in to single digit partition keys (0-9), consider the option of increasing the number of partitions from (0-20) to group the data into smaller chunks.
Once the data has been logically grouped into logical partition groups, the data logistics job reading from the staging job can read the job in stream or in succession, depending on the optimal load balance in the data logistics tool used.
The exercise of finding the optimal data logistics load is a trial and error process depending on the resources available, number of data, how big the existing data is in the target table (where applicable), and other factors.

## Implementation guidelines

* Push partitioning as early as possible (source extract or landing) so downstream stages can parallelize.
* Choose a partition key that yields even distribution; if skewed, increase cardinality (for example, two digits instead of one).
* Keep the partition key with the data through Staging/PSA so lookups and caching can reuse it.
* Partition-aware lookups and indexes can reduce cache build time and improve join performance.
* Design jobs so a failed partition can be retried independently without replaying successful partitions.

## Considerations and consequences

* If no usable field exists, deriving a partition from a non-key attribute can introduce duplicates; ensure it is part of the natural key or add deterministic hashing.
* Over-partitioning can increase orchestration overhead; tune the number of partitions to available resources and batch windows.
* Skewed partitions limit benefits; monitor distribution and adjust the partitioning scheme as data evolves.

## Related patterns

* [Design Pattern - Generic - Managing temporality by using Load, Event and Change dates](xref:design-pattern-generic-managing-multi-temporality).
* [Design Pattern - Generic - Using checksums for row comparison](xref:design-pattern-generic-using-checksums).


