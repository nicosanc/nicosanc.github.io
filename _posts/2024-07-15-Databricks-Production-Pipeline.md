---
title: "Databricks Associate DE Notes: Production Pipeline"
---

### Delta Live Tables
A framework for building reliable and maintainable data processing pipelines
- They function using multi hop

AN EXAMPLE USING ORDERS, CUSTOMERS, AND BOOKS TABLES FROM AN EXAMPLE DATABASE

#### BRONZE

```SQL
CREATE OR REFRESH STREAMING LIVE TABLE orders_raw
COMMENT "SOME COMMENT AB THE DATA"
AS SELECT * FROM cloudFiles(path, source_format, map("schema", "val_1 data_type, val_2 data_type,..."))
```

here the map data structure helps us manually declare the scheme for our delta live table

```SQL
CREATE OR REFRESH LIVE TABLE customers
COMMENT "COMMENT AB DATA"
AS SELECT * FROM json.'${path}/customer_json'
```

Table 2 is a batch based table that does not use real-time data which is useful for historical/large data + resource intensive operations

#### SILVER

```SQL
CREATE OR REFRESH STREAMING LIVE TABLE orders_cleaned
(CONSTRAINT valid_order_number EXPECT(order_id IS NOT NULL)
ON VIOLATION DROP ROW)
COMMENT "COMMENT AB DATA"
AS SELECT order_id, ..., c.profile:last_name as l_name,
CAST(from_unixtime(order_timestamp, "<datetime_format>") AS timestamp) order_timestamp,...
FROM STREAM(LIVE.orders_raw) o
LEFT JOIN LIVE.customers c
ON o.customer_id = c.customer_id
```

`date_trunc("DD", order_timestamp)` will shrink a timestamp to only show the day

* An important note is that using these cmds does *NOT* explicitly create the live tables, but rather defines them 
* The live tables must be manually created in Databricks, and they'll use the multi-hop architecture defined by these cmds (bronze -> silver -> gold)
* Event logs for a live table are stored as delta tables

### Change Data Capture (CDC)
- Process of identifying changes made to data in the data source and applying those changes to a target 
- ROW level changes like record inserts, updates, deletes 

PROS:
- Orders late arriving records by the sequence key (the time of change)
- Default assumption that rows will contain inserts and updates
- Can optionally apply deletes -> `APPLY AS DELETE WHEN`
- Specify one or more fields as primary keys
- Specify columns to ignore using `EXCEPT`
- Supports applying changes as SCD type 1 (default) or type 2

CONS:
- Breaks the append-only requirement for streaming table resources, increasing complexity of state management and the pipeline, and also potentially invalidates the table as a streaming source later down the pipeline due to inconsistencies in data processing

Any Notebook in a pipeline can reference tables and views from other notebooks as long as LIVE prefix is used (treated as Delta Live Table pipeline)
- Jobs: Run a sequence of tasks defined by notebooks registered to the job
- SQL Warehouses: 
	- Purpose: to provide a managed compute env optimized for SQL queries
	- use: used to run interactive SQL queries, generating reports, performing data analyses, and executing SQL jobs
- SQL Editor
	- Purpose: The UI where you write and execute SQL queries that leverage the optimized efficiency of the SQL warehouse
	- Use: write queries, view results, and create visualizations for the dashboard
