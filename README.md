# HandsonProject:
Prodect description: Implementing a data engineering solution using all services available in Azure Synapse Analytics (Synapse Pipelines, Synapse Data Flows, Dedicated SQL Pool, Serverless SQL Pool, Azure Data Lake Gen2, Meta Store, Synapse Link, Power BI). 

Using dataset from Taxi Trip in New York State (website: https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

Solution architecture:
- Import taxi-data manually into Data Lake raw container (bronze layer)
- Discover data requirement on this data, use the OPENROWSET() function offered by Serverless SQL Pool. Also look at creating external tables and views to make the job of the analyst easier
- Take data from bronze layer and process using Serverless SQL Pool to ingest into the silver layer (ingestion). Data here will have the schema, format parquet applied. Then create partitions and create external tables/views
- Data from silver layer go through transformation where I create the aggregations required to satify business requirements and output go to gold layer
- In gold layer, create external tables/views so I can use T-SQL to query
- Use PowerBI to report from the gold layer. Power BI connect to Severless SQL Pool to query from the gold layer.
- Use Synapse Pipelines to schedule, monitor, send alerts to keep the pipeline run on regular basis without interuption.
- Later I want to swap Severless SQL Pool with Spark Pool for the ingestion and transformation
- Look at how the metadata share between Spark Pool and Serverless SQL Pool
- Integrate the execution of Spark notesbooks withon Synapse Pipelines
- Look at accessing the data created by Spark Pool using Power BI with the help of Serverless SQL Pool
- Look at the use case for a Dedicated SQL Pool and replace the query engine for the reporting needs from Serverless SQL Pool to Dedicated SQL Pool
- use Synapse Link in Cosmo DB container to create analytical store and then use Spark Pool as well as Serverless SQL Pool to carry the data via SYnapse Links, Hybrid Transactional and Analytical Processing (HTAP) capability.

Create Synapse workspace:
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/752534ea-01e4-4cbc-8d2f-876fa93b01d6)

Datasets:
- I have my datasets saved in many file format to serve the purpose of transformation later in the project (CSV: Taxi zone, Calendar; Delta/CSV/Parquet: Trip data; TSV: Trip type, classic JSON: Rate code; single line JSON: Payment type; quoted CSV: Vendor)

In Azure Storage Explorer, ADLS Gen2/Blob container, create a new blob container name "taxi-data" so I can upload the whole folder of data into.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/0b9d9514-6d30-4dea-834f-31d0abace4cc)

Serverless SQL Pool
- How to cost control: workspace studio -> Develop -> SQL scripts -> (query as in the snip) -> save and publish.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/34395413-16e3-476a-9554-038c73580c66)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/845fb79c-2458-479c-b5eb-6d4ceb50ccd0)

Connect Synapse Serverless SQL Pool from Azure Data Studio (can connect to SQL SSMS alternative).

Get the server name and login info from Synapse workspace
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/f0fe1bda-c54e-47d8-9637-e7b14a6f5c00)

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/f5211ad6-6912-4c78-8b31-54a045dd7022)

Serverless SQL Pool - Query CSV
OPENROWSET() function: read the content of the remote data source without loading them into tables. It returns a set of rules with columns. It is used to read csv, parquet, delta, json format files.
To learn more on OPEROWSET(), read this documentation: https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-openrowset
Synapse Studio -> Data -> ADSL2 -> taxi-data -> raw -> taxi_zone.csv
- Extract data: 
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/f730b58c-e007-4433-a443-dba9fd4111f0)

- Examine data types:
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/2d85a822-ada7-44ba-82ed-56bdfa65030b)
I dont want to keep the default data type.
- Use WITH CLAUSE to provide explicit data types
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/97db6a6f-e107-4d9f-89d9-c5c9824a1405)
- If character (VARCHAR) columns have an UTF8 encoded text in them by default, I have to specify that UTF 8 collation by adding COLLATE Latin1_General_100_CI_AI_SC_UTF8 behind the column (1st way)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/a8beffd7-b28b-400b-9e57-b7d24373557b)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/e71329be-85be-4fde-abeb-824fb6ebad9f)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/f870d4f8-caad-48d8-9f0a-d1ec710888d9)
(2nd way) is changing at database level
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/36147d8e-e9d9-45cb-a594-b9181b01ccc2)
- Work with csv file without header and what if I want to select a few column only
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/dfb8590d-1098-4431-ac09-aa824616a093)
- Use External Data Source: use container name as the data source name. External data source can be created on the databases that I created, NOT on the master database.  
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/ef489fc4-e3d3-413e-ac43-7bbb96e199dd)
- Find location of data source
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/d762131f-b9ca-4f37-833f-44c12799cec9)




















