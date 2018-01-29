# Design Pattern - Generic - Logical Partition Keys

## Purpose
This design pattern describes how to handle large data volumes by using logical partition keys. It is a technique which may help loading large datasets faster; an alternative approach to handling large data volumes.

## Motivation
A common challenge encountered in Data Warehousing is how to handle large volumes of data. Typically, limited batch windows, restricted source extraction rules, large and complicated data sets among others contribute to long ETL processing times. This design pattern aims to reduce and optimize ETL processes for big data sets. 
Also known as
Logical data partitioning.
Horizontal portioning.

## Applicability
This pattern applies to all ETL processes that load big data sets. The technique described in this design pattern is generic and can be applied using any ETL tool.

## Structure
The central point in logical partitioning is to find ways to break down large data sets into logical groupings which can be processed independently from other data groupings coming from the same source. The possibility to logically slice data into multiple data sets and process them independently will improve flexibility in arranging the jobs to run in sequence or in parallel. This depends on the batch window allowed to process data and the overall available time to load the entire data set in the Data Warehouse.
The first step in using the logical partition technique is to identify the key that can be used to create the partition key that will logically group the data together. It is ideal if the source system can provide the partition key that can be used to load the data into stream. This will enable the partitioned load at the beginning when the staging area is populated. However, if this is not an option, the partition keys can be assigned to the source file via a script in the file system before loading in the Staging Area.
Another option is to introduce the partition key as data is loaded in the Staging Area. It is recommended that you implement the logical partitions at the earliest time possible to take advantage of the streamed loading process at the early phases of loading.
The attribute you nominate as the partition key should have the following characteristics:
Even distribution of number of rows based on the partition key.
Ideally, the nominated key that will determine the partition key should be   the primary key or form part of the primary key.
Numeric data type (if possible).
 In the example below, the source file CUSTOMER.txt contains the following:
Customer Key            Score Type                 Score        
123456                        Churn                           3.7
222222                        Churn                           4.1
333333                        Churn                           3.5
987659                        Churn                           2.9
1234567                      Churn                           8.7
The sample source file CUSTOMER.txt contains scores for all customers. In the example above, the last digit in the customer key field is used as the partition key. The partition key column is added before loading into staging area. This will facilitate reading of data in streams from the staging area.
The table that contains the customer data will be contain the partition key column derived from the last digit of the customer key.
Customer Key            Score Type                 Score         Partition Key
123456                        Churn                           3.7              6
222222                        Churn                           4.1              2
333333                        Churn                           3.5              3
987659                        Churn                           2.9              9
1234567                      Churn                           8.7              7
If the number of rows is still voluminous even after streaming them in to single digit partition keys (0-9), consider the option of increasing the number of partitions from (0-20) to group the data into smaller chunks.
Once the data has been logically grouped into logical partition groups, the ETL job reading from the staging job can read the job in stream or in succession, depending on the optimal load balance in the ETL tool used.
The exercise of finding the optimal ETL load is a trial and error process depending on the resources available, number of data, how big the existing data is in the target table (where applicable), and other factors.

## Implementation Guidelines
If the ETL load will be doing a lookup in the target table, consider using the partition keys when building the lookup cache. The records in the lookup cache can be built using the partition key that is currently being processed. This may lessen the build time of the lookup cache if the number of records is lessened.
Consider using the partition keys as part of the index in the target table to return results more quickly from the target table when doing lookups.
If one of the partition streams fails, only the failed stream needs to be re-run. Existing streams do not need to be re-run if they have succeeded.

## Considerations and Consequences
In some cases there is no useable field in the source data that can be used as part of the logical key. In this case, a character attribute can be used to determine the logical partition. However, the selected field must still form part of the natural key of the source file so that there are no processing errors due to records being processed multiple times within the same run.
Known uses
Any table that requires stream loading due to big data sets.

## Related Patterns
None.