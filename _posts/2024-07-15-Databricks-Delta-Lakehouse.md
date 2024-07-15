##### (1) Databricks Delta Lake house
A `data lake` is simply a centralized storage repository that can hold structured and unstructured data in its raw form, from various sources. Can hold data at any scale (massive amounts)
`Delta Lake`: An open source storage framework that brings reliability to data lakes. 
- Delta Lake deployed on the cluster and stored as files with transaction logs
- Transaction logs are:
	- (1) Ordered records of every transaction performed on a dataset (e.g. table)
	- (2) Single Source of Truth
	- (3) JSON file contains commit info: operation performed and predicates used (SQL cmds) and data files that were affected
- Files that are written at the same time are recorded in a log file when they're read. An update to a file is recorded to a new log file, done by cloning the file then updating the clone and finally removing the original file. 

##### (2) Advanced Delta Lake Features
`Time Travel`: Allows for working with different versions of files 
- Audit data changes using `DESCRIBE HISTORY`
- Query older versions of a file by using the older version's timestamp or version number (everything is versioned).
- Rollback old versions of a file using `RESTORE TABLE`
- Compress small files using `OPTIMIZE`
- Clean up unused files like uncommitted files or files that are not in the latest dataset
	- The `VACUUM` cmd allows you to retain files for a certain period, then deletes them once their lifetime expires

##### (3) Relational Entities
Databases in Databricks are schemas in `Hive Metastore`
- Hive is a central repository that holds metadata
- When tables are created in the default database or in a custom database, they're saved as folders in the *'hive/warehouse'* directory of the Hive metastore
- Databases can be saved to custom paths, but retain their definition in the hive metastore of *YOUR WORKSPACE*

Managed Tables: 
- Created under the database directory 
- Dropping Tables deletes their underlying data files
External Tables
- Created under a custom path using `CREATE TABLE table_name LOCATION '/path/'`
- Dropping Tables will *NOT* delete underlying datafiles

Delta Tables can be created using `CTAS` cmds
- CTAS: Populate newly created tables with the results of a SELECT statement
	- `CREATE TABLE _ AS SELECT * FROM _` 
	- Automatically infer schema information from the query results 
	- Does *NOT* support manual schema declarations
	- You can filter and rename columns within a CTAS cmd such as `CREATE TABLE _ AS SELECT col1, col3 as new_col_3 FROM table` where col3 gets renamed
	- Can also specify more information in the cmd such as
		- `COMMENT "some comment"`
		- `PARTITIONED_BY(col1, col2)`
		- `LOCATION '/path/'`
- Regular CREATE TABLE cmds have to manually declare the fields of the table, and then must use inserts to fill the table
- Table Constraints: Databricks supports the (1) NOT NULL  and (2) CHECK constraints
	- `ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_details` such as `ALTER TABLE table_name ADD CONSTRAINT valid_date CHECK(date > '2020-11-11')`

Delta Lakes can be cloned:
- Deep Clone: fully copies data and metadata from a source table as a target
	- `CREATE TABLE table_name DEEPCLONE source_table` 
-Shallow Clone: Only copies over delta transaction logs from source table
	- `CREATE TABLE table_name SHALLOW COPY source table` 
- Changes made to either kind of clone are *NOT* reflected in source table

Views in Databricks
- Stored views: persisted objects using `CREATE VIEW view AS query`
- Temp Views: Spark session-scoped views using `CREATE TEMP VIEW view AS query`
	- `Spark Sessions` are created when (1) A new notebook is opened, (2) Detaching and reattaching to a cluster, (3) Installing a pypackage, and (4) Restarting a cluster
- Global Temp Views: Cluster-scoped views that persist as long as the cluster is active using `CREATE GLOBAL TEMP VIEW view`
	- They can only be accessed using `global_temp.view` since global temps are always added to a temporary database called global_temp that persists in that cluster
