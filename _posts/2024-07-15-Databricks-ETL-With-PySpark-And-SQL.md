##### (1) Querying Files
- SELECT * FROM <font color='cyan'>file_format.`path`</font> 
	- Uses backticks to name the file path
	- Best supports self describing file formats like JSON or parquet files
	- Can retrieve a file, many files using the asterisk, or a directory
- Raw Data
	- Text based files can be extracted as raw strings using <font color='cyan'>text.`path`</font> 
	- Unstructured data like images can be extracted as raw bytes using <font color='cyan'>binaryFile.`path`</font> 
- CTAS: create table as select query
	- <font color='cyan'>CREATE TABLE AS SELECT * FROM file_format.`path`</font> 
	- CTAS always infer the schema automatically based on the query results
	- Best supports well defined schemas like parquets and tables
	- Do not support file options
- Registering tables on an external data sourec
	- <font color='cyan'>CREATE TABLE table (datatype col1, ...) USING external_data_source</font> 
	- Always creates an external table meaning it is just a reference to the data files so no actual data is ever moved
	- this is a *NON-Delta table* that keeps the files' data in their original format
	- <font color='cyan'>CREATE TABLE table_name USING CSV OPTIONS (header ='true', delimiter=';') LOCATION '/path/'</font>

##### (2) Querying files using Spark 
- All files in directory have the same format and schema for querying an entire directory
- When querying multiple files, use pyspark cmd <font color='cyan'>SELECT *, input_file_name() source_file FROM ...</font> 
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
- CRAS: <font color='cyan'>CREATE OR REPLACE TABLE ...</font> cmd can overwrite and create new table
- <font color='cyan'>DESCRIBE HISTORY table_name</font> provides metadata on table versions
- <font color='cyan'>INSERT OVERWRITE</font> can only overwrite existing tables and can only overwrite new records that match the table's schema
	- This means the overwrite does not effect the table's schema so no new columns can be added and no columns can be deleted
- <font color='cyan'>INSERT INTO</font> can directly append data to a table but it has no guarantees of avoiding duplicates so 2 identical "insert into as select" statements will populate the same data twice to a table
To work around duplicates we can use the <font color='cyan'>MERGE INTO</font> cmd
```SQL
CREATE OR REPLACE TEMPVIEW customer_updates AS SELECT * FROM json.`path`;

MERGE INTO customers c
USING customer_updates u
ON c.customer_id = u.customer_id
WHEN MATCHED AND c.email IS NULL AND u.email IS NOT NULL THEN
UPDATE SET email = u.email, updated = u.updated
WHEN NOT MATCHED THEN INSERT *
```
This will create a tempview that is merged into a table customers where all records in u that match a record in c (id) just updates their email, if not it inserts the entire record. Their schemas should match

##### (4) Advanced Transformations
- Nested Data Structures like JSONS that are stored as singular values in a cell, can be accessed in select cmds
	- <font color='cyan'>SELECT profile:firstname, profile:address:Country</font> 
- Structs are native Spark types with native attributes
	- <font color='cyan'>SELECT from_json(col_name, schema_from_json('{"Firstname":"thomas", ...}')) AS profile_struct FROM customers</font> 
	- Schema must be specified in the from_json() call or else it will fail
	- can be stored as a temp view
	- once stored, different member variables can be accessed using only '.' like <font color='cyan'>col_struct.field</font> 
	- To separate each subfield into its own column, <font color='cyan'>SELECT col_struct.*</font> 
- <font color='cyan'>Explode()</font> method turns an array of structs into separate rows of structs for each array index
- <font color='cyan'>collect_set()</font> method collects unique values from fields 
	- <font color='cyan'>array_distinct(flatten(collect_set(field)))</font> will flatten an N-D array to 1-D and preserves only unique values
- Spark supports standard <font color='cyan'>JOIN</font> cmds and <font color='cyan'>SET</font> cmds like <font color='cyan'>UNION, INTERSECT, MINUS</font> etc.
- Pivot Clause: used to turn values from rows into separate columns.
	- Like separating multiple rows of a certain product into multiple columns of one row, with the product being the identifier and something like 'month' or 'region' dictating the columns
	- useful when dealing with large datasets that contain many different categorical variables with unique values
- Higher Order Functions + SQL UDFs
	- Useful for parsing complex data entries like Arrays and Maps
	- <font color='cyan'>Filter()</font> filters data
	- <font color='cyan'>Transform(table_name, t -> CAST(t.value * 0.8 AS INT)) AS new_col_name)</font> will apply the transform to all matching entries
- UDFS: 
	- <font color='cyan'>CREATE OR REPLACE FUNCTION func_name() RETURNS data_type RETURN {code}</font>
- <font color='cyan'>CASE WHEN HOFs</font> work like "if", "else if", and "else" statements
