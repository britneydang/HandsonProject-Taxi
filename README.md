# HandsonProject:
Prodect description: Implementing a data engineering solution using all services available in Azure Synapse Analytics (Synapse Pipelines, Synapse Data Flows, Dedicated SQL Pool, Serverless SQL Pool, Azure Data Lake Gen2, Meta Store, Synapse Link, Power BI). 

Using dataset from Taxi Trip in New York State (website: https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

Solution architecture:
1. Import taxi-data manually into Data Lake raw container (bronze layer)
2. Discover data requirement on this data, use the OPENROWSET() function offered by Serverless SQL Pool. Also look at creating external tables and views to make the job of the analyst easier
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

1. Import data:
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

2. Discover the data:
Serverless SQL Pool - Query CSV
OPENROWSET() function: read the content of the remote data source without loading them into tables. It returns a set of rules with columns. It is used to read csv, parquet, delta, json format files.
To learn more on OPEROWSET(), read this documentation: https://learn.microsoft.com/en-us/azure/synapse-analytics/sql/develop-openrowset
Query taxi_zone.csv file: Synapse Studio -> Data -> ADSL2 -> taxi-data -> raw -> taxi_zone.csv
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

Query calendar.csv file
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/24cced00-3400-459f-b227-ce17b630f2f9)

Query vendor.csv file: what if one cell data has a comma?. 2 ways to do: add a \ in front of that comma or add " " for the whole cell data
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/090249be-58e7-4ecd-b72e-bda1a6750a99)

Query trip_type.tsv file: delimited by a tab(space)

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/557cecdf-9edb-4587-9cc7-860a2a9b111f)

Serverless SQL Pool - Query JSON
OPENROWSET() function doesnt have a specific paser for JSON document, but can use a combination of the CSV parser and few functions to query JSON files 
- Line-delimited JSON:
  FIELDTERMINATOR = 0x0b
  FIELDQUOTE = 0x0b
- Standard JSON:
  FIELDTERMINATOR = 0x0b
  FIELDQUOTE = 0x0b
  ROWTERMINATOR = 0X0b
Query payment_type.json - single line JSON and use JSON_VALUE function
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/ea61a117-f0de-4eeb-946e-82170ce15e44)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/d949add6-1328-410a-bf42-c989cafb35bb)
- Add JSON_VALUE() function and specify data type for each
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/cd824f22-1280-4bde-915a-17169fb15047)

Query payment_type.json - single line JSON and use OPENJSON function. Because JSON_VALUE has limitation and cant do well with array and large amount of data, but OPENJSON turn objects and properties into rows and columns.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/3754d878-37aa-47aa-8a37-4dd5a84b2820)

Query JSON Array: {} {} {}
- 1st way using JSON_VALUE(): but not deal because of having 2 separated columns.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/83eaaf6f-4419-4e4b-a2eb-757b07a723eb)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/a6629d2d-57d5-4867-9b53-eac37207b570)

- 2nd way using OPENJSON(): ideally to use for nested
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/28f36690-d195-4063-9b06-190e4c1825b3)

Query Standard JSON: [{},{},{}]
- Use OPENJSON() and ROWDETERMINATOR = '0x0b'
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/666caa06-a45b-47ce-8e17-365fad938ced)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/5b822396-5e5a-4d0a-a540-2ce682138aad)

Query Multi Line JSON: [\n {} \n,{} \n,{} \n]
- multi line JSON file is no different from the standard JSON file

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/ee824bf6-f5d2-49ac-988c-6805a122c7c8)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/61a3ba8c-5935-4bbd-9b89-81bb32c9771c)

Serverless SQL Pool - PARTITION - Query folders and Multiple files
- Usually transactional data (high volume) is partitioned into time based partition (yearly, monthly, daily, hourly)
Query folders and subfolders
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/976de3d9-6b9b-4233-b604-cbc684a7426e)

Query using File Metadata function. Specify the partition as detailed as possible to save cost 
- filename()
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/00c2623a-93c7-4873-a47c-e1dd71e9ad07)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/84dfaaea-8030-4409-88ca-4f67d6c8a161)

- filepath()
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/5a32b0ab-22c5-4d94-96b6-09172a871ca8)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/c1417023-4fb6-43db-9da6-c2b945859d08)

Serverless SQL Pool - Parquet file - Columnar format: Parquet file usually give a good data type, but watch out for VARCHAR type. It is the best to specify which column inside with() clause to improve performance and save cost.
Query single Parquet file
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/e849ebbb-054e-4adc-8ea1-d69b4e09f0df)

Query folders and subfolders (really similar to csv format)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/9b4c0020-5d25-4c62-9204-bce5d7577b00)

Query files from Delta Lake table (Delta files)
- Delta store the data in Parquet format. What differentiates Delta table is the transaction logs within a Delta Lake table. In the folder, there is a _delta_log folder (that keeps transaction log). The Delta Lake has to be coded from the main folder that contains _delta_log folder and data folders. One thing different in Delta file is when running the query, the furthest right of table will have the partitioned columns (in this case is column 'year' and 'month'). Cannot run without the partitioned columns. Always be specific with filter and be detail as much as possible to save time and cost.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/db348eca-7ffe-43dd-b3dd-98c461e8d504)














































