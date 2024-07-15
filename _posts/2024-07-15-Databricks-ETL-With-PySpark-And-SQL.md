##### (1) Querying Files
- SELECT * FROM <font color='cyan'>file_format.`path`
	- Uses backticks to name the file path
	- Best supports self describing file formats like JSON or parquet files
	- Can retrieve a file, many files using the asterisk, or a directory
- Raw Data
	- Text based files can be extracted as raw strings using text.`path`
	- Unstructured data like images can be extracted as raw bytes using binaryFile.`path`
- CTAS: create table as select query
	- CREATE TABLE AS SELECT * FROM file_format.`path` 
	- CTAS always infer the schema automatically based on the query results
	- Best supports well defined schemas like parquets and tables
	- Do not support file options
- Registering tables on an external data sourec
	- `CREATE TABLE table (datatype col1, ...) USING external_data_source`
	- Always creates an external table meaning it is just a reference to the data files so no actual data is ever moved
	- this is a *NON-Delta table* that keeps the files' data in their original format
	- `CREATE TABLE table_name USING CSV OPTIONS (header ='true', delimiter=';') LOCATION '/path/'`

##### (2) Querying files using Spark 
- All files in directory have the same format and schema for querying an entire directory
- When querying multiple files, use pyspark cmd`SELECT *, input_file_name() source_file FROM ... `
- This adds a column named source_file that is populated by the pyspark method which returns the filename the data was retrieved from
- text.path will store all the data's info in one column named 'value'.
	- For example, JSON files' info gets stored as one long JSON string
- binaryFile.path stores path, modification time, length, and raw byte content

SPARK API for extracting table data
```python
spark.read_table.write.mode("append)
	.format("csv")
	.option("header", "true")
	.option("delimiter", ";")
	.save("/path")

```

##### (3) Writing To Tables
- CRAS: `CREATE OR REPLACE TABLE ...` cmd can overwrite and create new table
- `DESCRIBE HISTORY table_name` provides metadata on table versions
- `INSERT OVERWRITE` can only overwrite existing tables and can only overwrite new records that match the table's schema
	- This means the overwrite does not effect the table's schema so no new columns can be added and no columns can be deleted
- `INSERT INTO` can directly append data to a table but it has no guarantees of avoiding duplicates so 2 identical "insert into as select" statements will populate the same data twice to a table
To work around duplicates we can use the `MERGE INTO` cmd
```SQL
CREATE OR REPLACE TEMPVIEW customer_updates AS SELECT * FROM json.`path`;

MERGE INTO customers c
USING customer_updates u
ON c.customer_id = u.customer_id
WHEN MATCHED AND c.email IS NULL AND u.email IS NOT NULL THEN
UPDATE SET email = u.email, updated = u.updated
WHEN NOT MATCHED THEN INSERT *
```
This will create a tempview that is merged into a table customers where all records in u that match a record in c (id) just updates their email, if not it inserts the entire record. Their schemas should match.

##### (4) Advanced Transformations
- Nested Data Structures like JSONS that are stored as singular values in a cell, can be accessed in select cmds
	- `SELECT profile:firstname, profile:address:Country` 
- Structs are native Spark types with native attributes
	- `SELECT from_json(col_name, schema_from_json('{"Firstname":"thomas", ...}')) AS profile_struct FROM customer` 
	- Schema must be specified in the from_json() call or else it will fail
	- can be stored as a temp view
	- once stored, different member variables can be accessed using only '.' like `col_struct.field` 
	- To separate each subfield into its own column, `SELECT col_struct.*` 
- `Explode()` method turns an array of structs into separate rows of structs for each array index
- `collect_set()` method collects unique values from fields 
	- `array_distinct(flatten(collect_set(field)))` will flatten an N-D array to 1-D and preserves only unique values
- Spark supports standard `JOIN` cmds and `SET` cmds like `UNION, INTERSECT, MINUS` etc.
- Pivot Clause: used to turn values from rows into separate columns.
	- Like separating multiple rows of a certain product into multiple columns of one row, with the product being the identifier and something like 'month' or 'region' dictating the columns
	- useful when dealing with large datasets that contain many different categorical variables with unique values
- Higher Order Functions + SQL UDFs
	- Useful for parsing complex data entries like Arrays and Maps
	- `Filter()` filters data
	- `Transform(table_name, t -> CAST(t.value * 0.8 AS INT)) AS new_col_name)` will apply the transform to all matching entries
- UDFS: 
	- `CREATE OR REPLACE FUNCTION func_name() RETURNS data_type RETURN {code}`
- `CASE WHEN HOFs` work like "if", "else if", and "else" statements
