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










