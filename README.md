# HandsonProject:
Prodect description: Implementing a data engineering solution using all services available in Azure Synapse Analytics (Synapse Pipelines, Synapse Data Flows, Dedicated SQL Pool, Serverless SQL Pool, Azure Data Lake Gen2, Meta Store, Synapse Link, Power BI). 

Using dataset from Taxi Trip in New York State (website: https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

Solution architecture:
1. Import taxi-data manually into Data Lake raw container (bronze layer)
2. Discover data requirement on this data, use the OPENROWSET() function offered by Serverless SQL Pool. Also look at creating external tables and views to make the job of the analyst easier
3. Take data from bronze layer and process using Serverless SQL Pool to ingest into the silver layer (ingestion). Data here will have the schema, format parquet applied. Then create partitions and create external tables/views
4. Data from silver layer go through transformation where I create the aggregations required to satisfy business requirements and output go to gold layer. In gold layer, create external tables/views so I can use T-SQL to query
5. Use Synapse Pipelines to schedule, monitor, send alerts to keep the pipeline run on regular basis without interuption. Convert pipeline to a dynamic pipeline. 

Use PowerBI to report from the gold layer. Power BI connect to Severless SQL Pool to query from the gold layer.
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

Transforming duration
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/b67b628e-a2b2-45b2-b83d-a1b04a68cca5)

Serverless SQL Pool - Data Virtualization 
- It is a logical data layer that allows to combine data from multiple sources at query time without having to write complex ETL pipelines to load the database.
- Create External table which I can create on the data in the data lake. As part of the external table creation, I can define things such as array of storage location, file formats, column names and data types. Once the external tables is created, it can be accessed like any tables in a relational database.
- View in here  is similar to any relational database. 2 ways to created: (1) select data using OPENROWSET() and create a view on that selection of data. (2) create view on the data selected from the external tables. It is a combination of external table and OPENROWSET() function.
- Serverless SQL Pool doesnt have any storage option available within it. Data always resides externally (for ex: azure data lake gen 2)
- Create external tables on the data lake storage or blob storage. Before creating External Table, have to create 2 database objects - External Data Source and External File Format (delimited, parquet, delta - not available in production yet). User/application can use data from external table.
- 2 types of external tables:
    - (1) CREATE EXTERNAL TABLE on the data that already present in th storage. Not moving data anywhere, just change the metadata. No data in the files or folders will be affected byt this process.
    - (2) CREATE EXTERNAL TABLE AS SELECT (CETAS): selected data will copied from the storage to another location in the storage. Make a table available so it can be coded. Metadata change.

Create External Table - DELIMITED CSV file manually. NOTE: Create External Table can be generated by UI (Data hub -> linked -> look for data file -> Create SQL script -> Create External Table -> input info -> using SQL -> OK)
- Objects that need to be created in an order: new database for the logical data warehouse, database schema (bronze level - raw), external data source, external file format, and finally external table.
- new database for the logical data warehouse has to be created from the master database. LOCATION is the URL (Data -> Linked -> ADLS2 -> data folder -> properties -> URL)

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/3966710a-cf9c-4f10-912c-9e55846a15d5)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/efba5d67-421c-4ba8-912a-9dc302dff4c2)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/e0423338-dfa0-4438-8aa2-e4728f0f4f98)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/1791c162-a663-4b7d-b2aa-1478d0030bc9)

Handling rejections/errors: add REJECT options inside CREATE EXTERNAL TABLE. Make sure to update external file format (add parser v1) and external table (update file format name and add reject options). Can view the rejection folder in Azure Storage Explorer. 

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/9ecd74c1-a7db-4a47-abfe-6b4691519da2)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/7efb4691-203a-4f7f-9f62-88d607e9f456)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/662c7837-870d-4e2c-b003-f3f112eea92e)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/cd31b482-25fe-4e37-8a3c-e6cf42135254)

Create External Table - DELIMITED TSV file. Make sure to update external file format (field_terminator = '\t' and parser v1) and external table (update file format name)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/eab71340-7be2-4049-9044-bfa04080ad16)

Create External Table - Partitioned CSV files. Because it is a large amount of data, I want to take off the reject option in external table and use file format with parser v2. 
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/a2d44ef7-6d48-4303-939a-e67fb9d8cadd)

Create External Table - Parquet file. Need to create external file format for parquet.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/9e1fa79e-ced2-4044-91fa-de5b46254fdf)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/9514a803-6d51-4401-8a79-cfde8fb22014)

Create External Table - Delta file. Need to create external file format for Delta.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/4507fbfd-7b3c-44de-ad7c-86cbd5a9d7bb)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/657906f2-ccf8-4962-b19f-beb42b7dc75d)

Views: virtual tables with rows and columns. Data in a view is defined by a Select statement. 2 ways to create views: (1) use OPENROWSET() function in a Select statement. (2) create view based on top of external table. 
Create View for JSON files. JSON file format is not supported in table, but is supported in view.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/7aa2263c-f542-421d-b812-9d371a29ad86)

Partition Pruning: 
- it's important to prune partitions to get the optimum performance.
- External tables in Serverless SQL Pool cannot prune partitions (not support).
- Combination of Views and OPENROWSET() function can be used to prune partitions (Microsoft recommended)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/8d9221cb-9f2b-454a-8135-d30bf674d1c8)

3. Data Ingestion in Severless SQL Pool:
- Use CETAS to convert the delimited text file (csv) to parquet

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/905256fe-63d9-4ffc-aab1-c9d818f44d87)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/415f6c1d-f689-4db1-9ab7-28b57056ab33)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/65ac94f8-585a-470d-9bdb-4b4aed190be6)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/6635a98d-5164-48ed-8841-9e9fcbf48bb9)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/cde4df42-e8d3-4c3d-9505-2a196afdf219)

- Transform the JSON file to parquet. Cant create external table for JSON format, instead use OPENROWSET() or the view.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/49c12be9-1a37-4ae4-a95b-60f4a76858ad)
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/2a8367fc-0197-4336-b96f-2ec413252cab)

- Transform the Partitioned Data (csv file) to parquet file
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/a5e35078-a592-46d5-95d0-dd11f6bd95de)

- Transform all partitioned data to parquet and also write output into partitioned folders. Each folder will have its own parquet file using stored procedure. It is different method compared to the one above which transformed all partitioned data to one parquet file.
    - stored proc - CETAS as input to transform data and stored proc - DROP external tables
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/002c9d59-babe-4b23-b290-adbf8a6300a5)
    - EXEC stored proc for each partition in silver data layer
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/61b2e8aa-0b50-49fe-b260-f36b6f894db8)
    - create a view on these transformed data so it can be queried easily (expose the partition to columns year and month so I can prune the partitions). Data is in silver folder.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/4a16730d-ab1d-467e-9e1d-d64dc3c71bb5)

4. Data Transformation in Severless SQL Pool: (gold table)
Business requirements 1:
- Trips made using credit card/cash payments
- Payment behavior during days of the week/weekend
- Payment behavior between boroughs
Non-functional requirements:
- Reporting data to be pre-aggregated for better performance
- Pre-aggregate data for each year/month partition in isolation
- Able to read data efficiently for specific months from aggregated data
- Minimize the number of aggregated tables created
Move to schema gold (cleanest data layer - reporting data layer))
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/6ef48467-d321-4ff8-ae81-78f5b096bfbd)

Stored proc in gold data layer

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/195a8559-8dcc-4c5a-853c-0e9b286a90a4)

![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/4349f689-0c06-43fc-8b90-2c13f14c8748)

Create a view on gold layer to query all of these data (because external table doesnt allow to prune partition, so it is best to have a view). Analyst and BI can use data on this view.
![image](https://github.com/britneydang/HandsonProject-Taxi/assets/110323703/e4eca038-3377-4daa-8583-805fa3296579)

5. Synapse Pipelines: It is a fully managed, serverless data integration and orchestration service within Synapse studio. It can orchestrate data through synapse data flows, dedicated SQL Pool scripts, serverless SQL pool scripts, synapse Spark notebooks, Azure Databricks Notebooks, Azure HDInsight scrripts, Azure Machine Learning Pipelines. Once pipelines are built, they need to be scheduled. Schedule based on clock, on tumbling window, and on events. 

Components to create an integration pipeline:
- Storage (ADLS, SQL Database)
- Compute (Synapse Pool, Azure Databricks)
- Linked service: is the place where specify the URL of the resource that I am trying to access and the credential that provides the access. It holds the connection information about a storage or a compute resource
- Dataset: provide metadata about data in files or tables
- Activity: there are many types of activity. Activities perform tasks but they are not executable. Some activities dont require a compute connection (copy, validation, etc.)
- Pipeline: in order to execute one or more activities, need a pipeline. We dont want to run pipeline manually. Pipelines should be scheduled to run at regular intervals or when an event happens.
- Trigger: invoke pipelines so they will run at regular intervals or when an event happens.

Now I will design a pipeline to transform a csv file to parquet file: If I do manually it will involve 2 steps: delete physical file from folder in Data Hub and rerun SQL script that contain Drop table if exists + CETAS. If I create a pipeline, it will include Delete Activity (delete activity > dataset > linked service > storage account (ADLS gen2) and Script Activity (script activity > linked service > serverless SQL Pool). And finally there is a trigger.























































































