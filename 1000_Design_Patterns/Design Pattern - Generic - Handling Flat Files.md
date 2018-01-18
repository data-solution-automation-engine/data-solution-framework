# Design Pattern - Generic - Handling Flat Files

## Purpose
This Design Pattern describes the requirements for extracting data from a flat file source.
## Motivation
Many source systems for a Data Warehouse still use flat files to deliver data. The processing of these files should be done in a standardised manner.
Also Known As
ASCII import
Sourcing text files
Applicability
This pattern is only applicable for loading processes from the source application into the Staging Area (of the Staging Layer) where the source provides the data in flat file format. This includes ASCII, CSV, XML or UNICODE formats.
Structure
With the delivering parties it should be agreed that all the files will be delivered to a central location where the ETL can process these files: the Landing Area. Within this landing area every source system has its own subdirectory.  Also, every subdirectory contains an Archive directory. The high level process is:
Unzip the file(s) in the landing area (if a zip file is delivered)
If control files are available the record counts should be checked against the number of records in the staging table
If the source files are being transferred via FTP a flag file should also be sent after actual source file has been sent completely to inform the ETL process that the source file has been transferred completely. Messaging applications (brokers) typically contain built-in functionality for this purpose
Run the import ETL process to the Staging Area (source to staging)
If the ETL process (Module) is successful the original files are moved to an archive directory. If the ETL process (Module) is unsuccessful the files are also removed but the original zip file is untouched
Implementation
Every separate source system has its own directory in the landing area
Every source directory has an archive directory
ETL process must be capable of loading multiple files in one execution. This may require file comparisons to detect the deltas to be loaded in the Staging Area if the file contains a full dataset (‘complete dump’). If the file already contains a delta it can be loaded into the Staging Area directly
The data types in the staging table are all VARCHAR (100) or VARCHAR (1000). This is done to simplify further processing and to prevent data type issues from flat file sources
If applicable the data types of the flat file source definition are set to the same length as defined in the previous bullet.  This is done to reduce the impact of field changes
A single Batch (workflow) may be created when a source systems delivers more than one file. This depends on the size of the files and the total processing time
Modules are only actively failed when record counts from the control files do not match.
Every source file should contain the date/time in the file name (YYYYMMDD_HHMM)
File lists may be used to select which files to load and to handle the changing filenames (date!). This also provides the opportunity to process multiple files at once
If files are delivered less frequently than the ETL process runs, for instance files are delivered weekly but the ETL process runs daily, a file list still needs to be created. This file list contains an empty dummy file so the process does not fail
Consequences
The decision not to copy the data types from the file definitions but to check and explicitly convert these in the ETL process will mean that explicit checks and data type conversions will have to be added later in the Integration Layer. Typically (and recommended) this is done as part of the Interpretation Area ETL processes. This also means that data that is dirty will be loaded into the Integration Area where it can be handled accordingly for issues such as missing business keys.
When the source system does not provide the file generation date time it will have to be determined by the Data Warehouse. For this scenario if the generation date of the file is equal to the ‘sysdate day’ when processing the daily batch data then one day should be subtracted from the generation date in order to process the correct date time of the event. The same process is applicable to any other used Batch interval.
A file date that is the same as the sysdate means that the file itself was generated on the same day, usually just past midnight. If the generation date is the sysdate minus one date, then the generation date can be used as date time event.
Related Patterns
Design Pattern 015 – Generic – Loading Staging Area Tables
Discussion items (not yet to be implemented or used until final)
None.