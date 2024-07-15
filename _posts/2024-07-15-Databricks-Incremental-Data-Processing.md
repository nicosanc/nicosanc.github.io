### Data Streams
Any source of data that grows over time
- New files landing in a cloud 'db'
- Updates to a database captured in a CDC feed
	- CDC feeds are mechanisms that monitor db's listening for changes to data like 'deletes' or 'updates'
- Events queued in a pub/sub messaging log
	- Pub/Sub stands for publish/subscribe. 
	- Publishers generate data and send them out w/o a specific destination.
	- Subscribers listen for certain events and consume the messages
	- messaging logs help different components in a system to communicate
	- Decoupling pubs and subs makes it more scalable and flexible
   
### Processing Data Streams
1. Reprocess the whole source data after each update => not that efficient
2. Only process the newly added data => Spark Structured Streaming is the better option
Spark streaming is a scalable data stream processing engine that allows for an infinite data source, processes the data, then passes relevant updates into a data sink (like a table or file)
Spark treats incoming data as a table so every new piece of data gets appended as a row. These tables are unbounded tables

READ
```python
streamDF = spark.readStream.table("Input_Table")
```

WRITE
```python
streamDF.trigger(processingTime="2 minutes")
	.outputMode("append")
	.option("CheckpointLocation", "/path/")
	.table("Output_Table)
```

OTHER TRIGGERS AND OUTPUT MODES 
```Python
.trigger(once=True) # processes all available data in a single batch
.trigger(availableNow=True) # Processes all data in multiple micro batches
.outputMode("complete") # Target table is overwritten by each batch
```

- Checkpointing is crucial for saving stream states and tracking your stream process
	- Checkpoint locations *MUST* be unique to each stream
- Structured Streaming guarantees
	- Fault tolerance through checkpoints and write ahead logs
	- they record the offset range of data being processed during each trigger interval
	- Exactly-once guarantee which means no duplicates saved 
- Stream DFs do not support sorting or reduplication

### Incremental Data Ingestion
Loading new data files encountered since the last ingestion which reduces redundant preprocessing
There are 2 mechanisms for processing files
1. `COPY INTO`: a SQL cmd that idempotently and incrementally loads new data files and files that already have been loaded are skipped.

```SQL
COPY INTO my_table
FROM '/path/'
FILEFORMAT = CSV
FORMAT_OPTIONS('delimiter'='|', header='true')
COPY_OPTIONS('mergeSchema'='true')
```

2. Autoloader: 
- Structured streaming
- Can process billions of files
- Support near real-time ingestion of millions of files per hour
- Autoloader checkpointing
	- Store metadata of the discovered files
	- exactly-once guarantees
	- fault tolerance
 
```python
spark.readStream.format('cloudFiles') # specifies Autoloader as format
	.option("cloudFiles.format", source_format)
	.option("cloudFiles.schemaLocation", schema_directory) # You can use a stored schema to properly read in files
	.load('path') # the path to the data source being read in 
	.writeStream.option("checkpointLocation", checkpoint_directory)
	.option("mergeSchema", "true")
	.table(table_name)
```

COPY INTO: Less scalable, Thousands of files
Autoloader: Millions of files, highly scalable

### Multi-Hop architecture
- AKA medallion Architecture
- Organize data in a multi-layered approach
- Incrementally improving the structure and quality of the data as it flows through each layer


| Input                 | BRONZE            | SILVER                            | GOLD            | OUTPUT                |
| --------------------- | ----------------- | --------------------------------- | --------------- | --------------------- |
| JSON, DB, input files | Ingested RAW data | Filtered, cleaned, augmented data | Aggregated Data | BI, ML, analysts, etc |

Benefits:
- Simple data model
- Enables incremental ETL
- Combine streaming and Batch workloads in unified pipeline
- Can recreate tables from Raw data at anytime

