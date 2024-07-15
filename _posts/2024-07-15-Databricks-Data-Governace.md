##### Programmatically grant, deny, and revoke access to data objects
```SQL
GRANT privilege ON object_name TO user_info
GRANT SELECT ON my_table TO nico@user.com
```

Object types that can be managed:
- Catalog: The entire data catalog, which is a centralized repo that allows users to organize, manage, and access data assets across various databases and tables within the databricks environment
	- Includes: metadata about DBs, tables, views, and files stored in Databricks -> best way to manage all assets of one or more projects all at once
- Schema: An entire database
- Table: external or managed tables
- View: SQL views
- Functions: named functions
- ANY FILE: the underlying filesystem

Privileges Allowed:
- SELECT: Read privilege
- MODIFY: add, modify, delete privilege
- CREATE: create objects
- READ_METADATA: view an object and its metadata
- USAGE: the baseline privilege to do anything within a workspace
- ALL PRIVILEGES: full privilege

Granting Privileges by Role:
- Databricks Admin: All objects in a catalog and underlying filesystem
- Catalog Owner: All objects in catalog
- DB Owner: All objects in a database
- Table/View/Func: only to the table, view, or function

More Operations:
- DENY
- REVOKE
- SHOW GRANTS

(2) Unity Catalog 
Centralized governance solution across all your workspaces on any cloud filesystem
- Unify governance for all data and AD assets
	- Files, tables, ML models, and dashboards
	- Based on SQL
- Account Console
	- Workspaces are abstracted out of the Account Console, multiple workspaces can access the same metastore and access control list

(remebmer to paste img)

```SQL 
SELECT * FROM catalog.schema.table
```

(3) UC Hierarchy
1. UC Metastore
2. Catalog, storage credential, External location, shares, recipients
3. Schema
4. Table/View/Function

(4) Identities
- Users: identified by email
	- Account Admin (for example)
- Service Principles: identified by application ID
	- Service Principles with admin privileges
- Groups: grouping Users and and SP 
	- nested groups
Identity Federation:
- Can exist at (1) Account (2) Workspace level
- IDs are created once at Account level, then assigned to workspaces => not redundant 

Privileges:
- CREATE
- USAGE
- SELECT
- MODIFY
- READ FILES
- WRITE FILES
- EXECUTE (for functions)

Security Model
GRANT PRIV ON securable_obj TO principle
- securable objects are anything within the 3-level hierarchy excluding the actual UC metastore 
- You still have access to a workspace's legacy hive metastore when it is assigned to a UC metastore
Features:
- Built in data search + discovery
- Automated Lineage -> data origin and all references
- No hard migrations -> unifies UC and Legacy Catalog

Remember, all jobs are orchestrated in `Workflows`

