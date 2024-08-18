---
title: short notes
date: 2024-04-07
tags:
  - technical
  - snowflake
---
### QAS

- Examples of the types of workloads that might benefit from the query acceleration service include:
    - Ad hoc analytics.
    - Workloads with unpredictable data volume per query.
    - Queries with large scans and selective filters.
-  [QUERY_ACCELERATION_ELIGIBLE View](https://docs.snowflake.com/en/sql-reference/account-usage/query_acceleration_eligible) & [SYSTEM$ESTIMATE_QUERY_ACCELERATION](https://docs.snowflake.com/en/sql-reference/functions/system_estimate_query_acceleration) function to assess whether a specific query is eligible for acceleration.
- eligible queries:
    - Large scans with an ==aggregation or selective filter.==
    - Large scans that ==insert many new rows.==
-  **reasons that queries are ineligible**
    - There are not enough partitions in the scan.
    - Even if a query has a filter, the filters may not be selective enough
    - if the query has an aggregation with GROUP BY, the cardinality of the GROUP BY expression might be too high for eligibility.
    - The query includes a LIMIT clause but does not have an ORDER BY clause.
    - The query includes functions that return nondeterministic results (for example, [SEQ](https://docs.snowflake.com/en/sql-reference/functions/seq1) or [RANDOM](https://docs.snowflake.com/en/sql-reference/functions/random)).
- QAS eligibility can be checked only the ACCOUNTADMIN role (or a [role granted IMPORTED PRIVILEGES](https://docs.snowflake.com/en/sql-reference/account-usage.html#label-enabling-usage-for-other-roles) on the shared SNOWFLAKE database) is in use.
- The query acceleration service supports the following SQL commands:
    - SELECT
    - INSERT
    - CREATE TABLE AS SELECT (CTAS)
- If the scale factor is not explicitly set, the ==default value is 8==. Setting the scale factor to 0 eliminates the upper bound limit and allows queries to lease as many resources as necessary and as available to service the query.

```SQL
ALTER WAREHOUSE my_wh SET
ENABLE_QUERY_ACCELERATION = true
QUERY_ACCELERATION_MAX_SCALE_FACTOR = 0;
```

- [QUERY_ACCELERATION_HISTORY](https://docs.snowflake.com/en/sql-reference/functions/query_acceleration_history)  — view billing for QAS
- The scale factor is a _**cost control**_ mechanism that allows you to set an upper bound on the amount of compute resources a warehouse can lease for query acceleration. This value is used as a multiplier based on warehouse size and cost.
    
    For example, suppose that you set the scale factor to 5 for a medium warehouse. This means that:
    
    - The warehouse can lease compute resources up to 5 times the size of a medium warehouse.
    - Because a medium warehouse costs [4 credits per hour](https://docs.snowflake.com/en/user-guide/cost-understanding-compute.html#label-virtual-warehouse-credit-usage), leasing these resources can cost up to an additional 20 credits per hour (4 credits per warehouse x 5 times its size).

### Access control considerations

- The user administrator (USERADMIN) role includes the privileges to create and manage users and roles (assuming ownership of those roles or users has not been transferred to another role).
- The security administrator (i.e users with the SECURITYADMIN system role) role includes the global MANAGE GRANTS privilege to grant or revoke privileges on objects in the account. The USERADMIN role is a child of this role in the default access control hierarchy.
- The system administrator (SYSADMIN) role includes the privileges to create warehouses, databases, and all database objects (schemas, tables, etc.).
- objects must be assigned to SYSADMIN
- recommend using a role other than ACCOUNTADMIN for automated scripts
- By default, ==not== even the ACCOUNTADMIN role ==can modify or drop objects created by a custom role==. The custom role must be granted to the ACCOUNTADMIN role directly or, preferably, ==to another role in a hierarchy with the SYSADMIN role as the parent==. The SYSADMIN role is managed by the ACCOUNTADMIN role.
- A user cannot view the result set from a query that another user executed.
- Even a user with the ACCOUNTADMIN role cannot view the results for a query run by another user.
- A cloned object is considered a new object in Snowflake.
- Any privileges granted on the source object do not transfer to the cloned object.
- However, a cloned container object (a database or schema) retains any privileges granted on the objects contained in the source object. For example, a cloned schema retains any privileges granted on the tables, views, UDFs, and other objects in the source schema.

### external tables

- An external table is a Snowflake feature that allows you to query data stored in an [external stage](https://docs.snowflake.com/en/user-guide/data-load-overview.html#label-data-load-overview-external-stages) as if the data were inside a table in Snowflake. 
- External tables let you store (within Snowflake) certain file-level metadata, including filenames, version identifiers, and related properties.
- External tables are read-only.

> [!important]  
> If Snowflake encounters an error while scanning a file in cloud storage during a query operation, the file is skipped and scanning continues on the next file. A query can partially scan a file and return the rows scanned before the error was encountered  

- METADATA$FILENAME or METADATA$FILE_ROW_NUMBER pseudo columns.
- ==Delta Lake is a table format on your data lake that supports ACID (atomicity, consistency, isolation, durability) transactions among other features.==
- All data in Delta Lake is stored in Apache Parquet format.
- To create an external table that references a Delta Lake, set the `TABLE_FORMAT = DELTA` parameter in the CREATE EXTERNAL TABLE statement.
- The refresh operation synchronizes the metadata with the latest set of associated files in the external stage and path, i.e.:
    - New files in the path are added to the table metadata.
    - Changes to files in the path are updated in the table metadata.
    - Files no longer in the path are removed from the table metadata.
- An overhead to manage event notifications for the automatic refreshing of external table metadata is included in your charges.
- a small maintenance overhead is charged for manually refreshing the external table metadata (using ALTER EXTERNAL TABLE … REFRESH)
- manual refreshes of external tables enhanced with Delta Lake rely on user-managed compute resources (i.e. a virtual warehouse).
- Users with the ACCOUNTADMIN role, or a role with the global MONITOR USAGE privilege, can query the [AUTO_REFRESH_REGISTRATION_HISTORY](https://docs.snowflake.com/en/sql-reference/functions/auto_refresh_registration_history) table function to retrieve the history of data files registered in the metadata of specified objects and the credits billed for these operations
- A stored procedure can remove older staged files from the metadata in an external table using an ALTER EXTERNAL TABLE … REMOVE FILES statement.
- **[EXTERNAL_TABLES View](https://docs.snowflake.com/en/sql-reference/info-schema/external_tables)** **—** Displays information for external tables in the specified (or current) database.
- **[AUTO_REFRESH_REGISTRATION_HISTORY](https://docs.snowflake.com/en/sql-reference/functions/auto_refresh_registration_history)** **—** Retrieve the history of data files registered in the metadata of specified objects and the credits billed for these operations.
- **[EXTERNAL_TABLE_FILES](https://docs.snowflake.com/en/sql-reference/functions/external_table_files)** **—** Retrieve information about the staged data files included in the metadata for a specified external table.
- **[EXTERNAL_TABLE_FILE_REGISTRATION_HISTORY](https://docs.snowflake.com/en/sql-reference/functions/external_table_registration_history)** **—**Retrieve information about the metadata history for an external table, including any errors found when refreshing the metadata.

### kafka connector

- Apache Kafka software uses a publish and subscribe model to write and read streams of records, similar to a message queue or enterprise messaging system.
- Kafka allows processes to read and write messages asynchronously.
- subs need not be connected directly to publisher, a publisher can queue a message in Kafka for the subscriber to receive later.
- An application publishes messages to a _topic_, and an application subscribes to a topic to receive those messages.
- Topics can be divided into _partitions_ to increase scalability.
- The Kafka connector is designed to run in a Kafka Connect cluster to read data from Kafka topics and write the data into Snowflake tables.
- each Kafka message contains one row.
- Kafka, like many message publish/subscribe platforms, allows a many-to-many relationship between publishers and subscribers.
- A single application can publish to many topics, and a single application can subscribe to multiple topics.
- With Snowflake, the typical pattern is that one topic supplies messages (rows) for one Snowflake table.
- kafka supports 2 loading methods: Snowpipe and Snowpipe Streaming.
- The connector converts the topic name to a valid Snowflake table name using the following rules:
    - Lowercase topic names are converted to uppercase table names.
    - If the first character in the topic name is not a letter (`a-z`, or `A-Z`) or an underscore character (`_`), then the connector prepends an underscore to the table name.
    - If any character inside the topic name is not a legal character for a Snowflake table name, then that character is replaced with the underscore character.
- every Snowflake table loaded by the Kafka connector has a schema consisting of two VARIANT columns:
    - RECORD_CONTENT. This contains the Kafka message.
    - RECORD_METADATA. This contains metadata about the message, for example, the topic from which the message was read.
- Kafka message is passed to Snowflake in JSON format or Avro format.
- The connector creates the following objects for each topic:
    - One internal stage to temporarily store data files for each topic.
    - One pipe to ingest the data files for each topic partition.
    - One table for each topic. If the table specified for each topic does not exist, the connector creates it; otherwise, the connector creates the RECORD_CONTENT and RECORD_METADATA columns in the existing table and verifies that the other columns are nullable (and produces an error if they are not).
- The Kafka connector subscribes to one or more Kafka topics based on the configuration information provided via the Kafka configuration file or command line (or the Confluent Control Center; Confluent only).

### search optimization service

- Business users who need fast response times for critical dashboards with highly selective filters
- Data scientists to train large models
- Data applications retrieving a small set of results based on an extensive set of filtering predicates.
- Substring and regular expression searches (e.g. [ NOT ] LIKE, [ NOT ] ILIKE, [ NOT ] RLIKE, etc.)
- Queries on fields in VARIANT, OBJECT, and ARRAY (semi-structured) columns that use the following types of predicates:
    - Equality predicates.
    - IN predicates.
    - Predicates that use ARRAY_CONTAINS.
    - Predicates that use ARRAYS_OVERLAP.
    - Substring and regular expression predicates.
    - Predicates that check for NULL values.
- Queries that use selected geospatial functions with [GEOGRAPHY](https://docs.snowflake.com/en/sql-reference/data-types-geospatial) values.
- sos maintains a persistent data structure called a _search access path_. the search access path keeps track of which values of the table’s columns might be found in each of its micro-partitions, allowing some micro-partitions to be skipped when scanning the table
- Queries that involve other types of values (i.e., FLOAT, GEOMETRY) do not benefit from sos.
- Search optimization can also improve the performance of views and of queries that use JOIN.
- SOS doesn’t support - MV, external tables, columns defined with collate, column concatenation, analytical expressions and casting columns
- SOS doesn’t improve the queries of time travel
- improve performance when searching for substrings that are 5 or more characters long.
- SOS for AND — where x = constant and y = constant will work if there’s imbalance of returning of rows
- SOS for OR — where x = constant or y = constant ==will not== work if there’s imbalance of returning of rows
- If you clone a table, schema, or database, the SEARCH OPTIMIZATION property and search access paths of each table are also cloned.

### cloning

if the source object is a database or schema, ==the clone inherits== ==_**all**_== ==granted privileges on the clones of all child objects== contained in the source object:

- When tables, schemas, or databases are cloned, the cloning operation does not contribute to total storage until data manipulation language ==(DML) operations are performed on the source or target==, which modify or delete existing data or add additional data.
- For databases, contained objects include schemas, tables, views, etc.
- For schemas, contained objects include tables, views, etc.

> [!important]  
> Note that the clone of the container itself (database or schema) does not inherit the privileges granted on the source container.  

- clone won’t copy grants, but there’s an option to copy grant while cloning
- object params are cloned
- If the database or schema containing both the table and sequence is cloned, the cloned table references the cloned sequence.
- Otherwise, the cloned table references the source sequence.
- same goes with foreign key constraints
- when table with clustering key is cloned, ==it clones the table and clustering key but automatic clustering will be suspended in new table==, can be resumed manually
- external stages are cloned but ==internal stages are not cloned==
- tables associated with internal stage are cloned but ==the files are not copied==
- pipe referencing external stage can be cloned but not internal stage
- when a database or schema that contains source tables and streams is cloned, any unconsumed records in the streams (in the clone) are inaccessible. This behavior is consistent with [Time Travel](https://docs.snowflake.com/en/user-guide/data-time-travel) for tables. If a table is cloned, historical data for the table clone begins at the time/point when the clone was created.
- ==Cloning an individual policy object is not supported.==
- Cloning a schema results in the cloning of all policies within the schema.
- regarding policies it’s same as sequences
- refrain from executing DDL/DML statements while cloning
- not possible to clone child objects with no time travel. to ignore this use [IGNORE TABLES WITH INSUFFICIENT DATA RETENTION](https://docs.snowflake.com/en/sql-reference/sql/create-clone.html#label-ignore-tables-with-insufficient-data-retention)
- if child object has shorter data retention time compared to parent object then can’t clone the child object which is equal to parent’s retention time
- ==Cloning is achieved through metadata operation performed in the cloud services layer.== Data is not physically copied, nor are new micro-partitions created—instead, the cloned table points to the micro-partitions of the source table.

### Notes on Micro-Partitioning in Snowflake

**Overview:**

- **Micro-Partitions**: All data in Snowflake tables is automatically divided into micro-partitions, which are contiguous units of storage.
- **Size**: Each micro-partition contains between ==50 MB and 500 MB of uncompressed data==. However, data is stored compressed, so the actual size in Snowflake is smaller.
- **Structure**: Groups of rows in tables are mapped into individual micro-partitions, organized in a columnar fashion.

**Benefits:**

- **Granular Pruning**: The size and structure of micro-partitions allow for extremely granular pruning of very large tables, which can consist of millions or even hundreds of millions of micro-partitions. This improves query performance by reducing the amount of data scanned.

**Metadata Storage:**  
Snowflake stores metadata about all rows stored in each micro-partition, including:  

- ==**Range of Values**==: For each column in the micro-partition.
- ==**Number of Distinct Values**==: For each column.
- ==**Additional Properties**==: Used for both optimization and efficient query processing.

**Automatic Partitioning:**

- **Automatic Process**: Micro-partitioning is automatically performed on all Snowflake tables.
- **Ordering**: Tables are transparently partitioned based on the ordering of the data as it is inserted or loaded.

**Key Points:**

- Micro-partitioning improves query performance through efficient data storage and granular pruning.
- Metadata about the data in micro-partitions enhances query optimization and processing efficiency.
- The process is automatic and transparent, requiring no manual intervention from users.

---

  

### About EXPLAIN in snowflake — https://docs.snowflake.com/en/sql-reference/sql/explain

- EXPLAIN compiles the SQL statement, but does not execute it, so EXPLAIN does not require a running warehouse.
- Although EXPLAIN does not consume any compute credits, the compilation of the query does consume Cloud Service credits, just as other metadata operations do.
- To post-process the output of this command, you can:
    - Use the [RESULT_SCAN](https://docs.snowflake.com/en/sql-reference/functions/result_scan) function, which treats the output as a table that can be queried.
    - Generate the output in JSON format and insert the JSON-formatted output into a table for analysis later. If you store the output in JSON format, you can use the function [SYSTEM$EXPLAIN_JSON_TO_TEXT](https://docs.snowflake.com/en/sql-reference/functions/system_explain_json_to_text) or [EXPLAIN_JSON](https://docs.snowflake.com/en/sql-reference/functions/explain_json) to convert the JSON to a more human readable format (either tabular or formatted text).
- The assignedPartitions and assignedBytes values are upper bound estimates for query execution. Runtime optimizations such as join pruning can reduce the number of partitions and bytes scanned during query execution.
- The EXPLAIN plan is the “logical” explain plan. It shows the operations that will be performed, and their logical relationship to each other. The actual execution order of the operations in the plan does not necessarily match the logical order shown by the plan.
- If any of the database objects in the EXPLAIN statement are INFORMATION_SCHEMA objects, the statement fails with error `EXPLAIN command has insufficient privilege on object <objName>`.

### difference between CALLER and OWNER

In Snowflake, the `CALLER` and `OWNER` security contexts define how privileges are handled when executing stored procedures. These contexts determine whose privileges are checked when accessing database objects within the stored procedure. Here's a detailed explanation of each:

### `CALLER` Rights

When a stored procedure runs with `CALLER` rights, the procedure executes with the privileges of the user who calls the stored procedure. This means that the access permissions of the caller are used to determine what operations can be performed within the procedure.

**Key Points:**

- **Privileges**: The stored procedure can only perform actions that the caller has permission to execute.
- **Use Case**: This is useful when you want to ensure that the caller’s privileges are respected, such as when the actions performed by the procedure should be constrained by the caller’s access level.
- **Example**:
    
    ```SQL
    CREATE OR REPLACE PROCEDURE my_procedure()
    RETURNS STRING
    LANGUAGE SQL
    EXECUTE AS CALLER
    AS
    $$
    BEGIN
      -- Actions here are checked against the caller's privileges
    END;
    $$;
    ```
    

### `OWNER` Rights

When a stored procedure runs with `OWNER` rights, the procedure executes with the privileges of the owner (the role that created or owns the procedure). This means that the stored procedure has the privileges of the owner, regardless of who calls the procedure.

**Key Points:**

- **Privileges**: The stored procedure can perform actions that the owner has permission to execute, even if the caller does not have those permissions.
- **Use Case**: This is useful for procedures that need to perform privileged operations that the caller might not be allowed to perform, such as administrative tasks.
- **Example**:
    
    ```SQL
    CREATE OR REPLACE PROCEDURE my_procedure()
    RETURNS STRING
    LANGUAGE SQL
    EXECUTE AS OWNER
    AS
    $$
    BEGIN
      -- Actions here are checked against the owner's privileges
    END;
    $$;
    ```
    

### Differences in Detail:

1. **Privilege Scope**:
    - **CALLER**: Uses the permissions of the user executing the procedure.
    - **OWNER**: Uses the permissions of the procedure’s owner (the role that created the procedure).
2. **Access Control**:
    - **CALLER**: Restricts the procedure to only actions the caller is permitted to perform. If the caller does not have the required permissions for a particular action, the action will fail.
    - **OWNER**: Grants the procedure the ability to perform actions that the owner can perform, regardless of the caller’s permissions. This is useful for sensitive or administrative tasks that require elevated privileges.
3. **Security Implications**:
    - **CALLER**: Ensures that the caller's security context is respected, enhancing security by limiting the procedure's capabilities to the caller's permissions.
    - **OWNER**: Can perform more powerful actions as it uses the owner’s permissions, which can be broader than those of the caller. This can be a potential security risk if not managed properly, as it could allow callers to indirectly perform actions they are not authorized to do.

### Practical Examples:

**CALLER Rights Example**:

```SQL
-- Assume the procedure performs a SELECT from a sensitive table
CREATE OR REPLACE PROCEDURE get_sensitive_data()
RETURNS TABLE (col1 STRING)
LANGUAGE SQL
EXECUTE AS CALLER
AS
$$
  SELECT * FROM sensitive_table;
$$;
```

- This procedure will only execute successfully if the caller has `SELECT` permissions on `sensitive_table`.

**OWNER Rights Example**:

```SQL
-- Assume the procedure performs an administrative task
CREATE OR REPLACE PROCEDURE perform_admin_task()
RETURNS STRING
LANGUAGE SQL
EXECUTE AS OWNER
AS
$$
  INSERT INTO audit_log (action, performed_by) VALUES ('admin_task', CURRENT_USER());
  RETURN 'Task completed';
$$;
```

- This procedure can insert into the `audit_log` table even if the caller does not have `INSERT` permissions, as long as the owner of the procedure has those permissions.

### Choosing Between `CALLER` and `OWNER`:

- Use `CALLER` rights when you need to ensure that the actions within the procedure are limited to the permissions of the user executing the procedure.
- Use `OWNER` rights when the procedure needs to perform actions that require elevated privileges that the caller might not have.

Understanding the distinction between `CALLER` and `OWNER` rights helps in designing secure and effective stored procedures in Snowflake, ensuring that appropriate permissions are enforced based on the use case.

### Database roles and role hierarchies

The following limitations currently apply to database roles:

- If a database role is granted to a [share](https://docs.snowflake.com/en/user-guide/data-sharing-gs.html#label-data-sharing-provider-option2), then no other database roles can be granted to that database role. For example, if database role `d1.r1` is granted to a share, then attempting to grant database role `d1.r2` to `d1.r1` is blocked.
    
    In addition, if a database role is granted to _**another**_ database role, the grantee database role cannot be granted to a share.
    
    Database roles that are granted to a share can be granted to other database roles, as well as account roles.
    
- Account roles cannot be granted to database roles in a role hierarchy.

### releases

Each week, Snowflake deploys two planned/scheduled releases:

**Full release:** A full release may include any of the following:  
• New features  
• Feature enhancements or updates  
• Fixes  
• Behavior changes (see next section in this topic)  
In addition, a full release includes updated Snowflake release notes documentation per weekly cycle. See   
[What’s New](https://docs.snowflake.com/en/release-notes/new-features).  
Full releases may be deployed on any day of the week, except we typically do not plan full releases on Friday to mitigate against issues that may be encountered during off-hours.  

**Patch release:** A patch release includes fixes only. Note that the scheduled/planned patch release for a given week may be canceled if the full release for the week is sufficiently delayed or prolonged.  
Additionally,  
==patch releases are deployed (as needed) during or after the completion of the full release== to address any issues that are encountered.

**Behavior Changes (Monthly)**

- Each month except nov and dec, snowflake chooses one of the weekly release to release behavior changes
- A behavior change is defined as any change to existing behavior that returns different results from before and may impact customer code or workloads

**Bundle Lifecycle**

T**esting period (1st month):** ==The bundle is introduced “Disabled by Default”.== users can choose to enable the bundle

**Opt-out period (2nd month):** The bundle moves from “Disabled by Default” to “Enabled by Default”. During this period, you can choose to _disable_ the bundle in your accounts.  
  

During these two periods, Snowflake does not override the setting for a given bundle. For example, if you disable a bundle during the testing period, we do not enable it at the beginning of the opt-out period.

  

**Staged Release Process**

Once a full release has been deployed, Snowflake does not move all accounts to the release at the same time.

**Day 1** Stage 1 (_early access_) for designated Enterprise (or higher) accounts.

**Day 1 or 2** Stage 2 (_regular access_) for Standard accounts.

**Day 2** Stage 3 (_final_) for Enterprise (or higher) accounts.

> [!important]  
> This staged approach only applies to full releases. For patch releases, all accounts are moved on the same day.  

### replication

|   |   |   |   |
|---|---|---|---|
|**Object**|**Type or Feature**|**Replicated**|**Notes**|
|[Databases](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-database-replication)||✔|Replication of some databases is not supported or might fail the refresh operation. For more information, see [Current limitations of replication](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-replication-limitations).|
|[Integrations](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-account-replication-integrations)|Security, API, Notification, Storage, External Access [[1]](https://docs.snowflake.com/en/user-guide/account-replication-intro\#id2)|✔|For additional caveats and details on the supported types, see [Integration replication](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-account-replication-integrations).|
|[Network policies](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-network-policy-replication)||✔||
|[Parameters (account level)](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-parameter-replication)||✔||
|[Resource monitors](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-resource-monitor-replication)||✔|Resource monitor notifications for non-administrator users are replicated if you include `users` in the group, however account administrator notification settings are not replicated. For more information, see [Replication of resource monitor email notification settings](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-resource-monitor-notifications-replication).|
|[Roles](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-role-replication)||✔|• Includes [account and database roles](https://docs.snowflake.com/en/user-guide/security-access-control-overview.html#label-access-control-overview-role-types).  <br>• Includes privileges granted to roles, as well as roles granted to roles (i.e. hierarchies of roles).  <br>• If users and roles are replicated, roles granted to users are also replicated.  <br>• The REPLICATE and FAILOVER privileges are   <br>_not_ replicated.|
|[Shares](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-share-replication)||✔|Replication of [inbound shares](https://docs.snowflake.com/en/user-guide/data-share-consumers) (shares from providers) is _not_ supported.|
|[Users](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-user-replication)||✔||
|[Warehouses](https://docs.snowflake.com/en/user-guide/account-replication-intro#label-warehouse-replication)||✔||

- If `users` and `resource monitors` are included in the `object_types` list for the replication or failover group, notification settings for non-administrator users are replicated
- If `resource monitors` is included in the `object_types` list for the replication or failover group, but `users` is not included, the `notify_users` list for a secondary warehouse-level resource monitor is empty.
- Replication of [inbound shares](https://docs.snowflake.com/en/user-guide/data-share-consumers) (shares from providers) is _**not**_ supported.
- Warehouses are replicated in the suspended state to each target account and can be resumed in the target account
- If a role is dropped that has the OWNERSHIP privilege on an active pipe in the target account, the refresh operation fails.
- Privileges on replication groups and failover groups are _**not**_ replicated. If the REPLICATE or FAILOVER privilege has been granted on replication groups or failover groups, these privileges need to be granted in both the source _**and**_ target accounts
- future grants are replicated
- If new objects are created in a target account during a refresh from the source account, and roles are not replicated to the target account, the OWNERSHIP privilege for the new objects is granted to the ACCOUNTADMIN role. if roles are replicated while creating new object then ownership will be granted to the role

### User who refreshes objects in a target account

A user who executes the [ALTER FAILOVER GROUP … REFRESH](https://docs.snowflake.com/en/sql-reference/sql/alter-failover-group.html#label-alter-failover-group) command to refresh objects in a target account from the source account must use a role with the REPLICATE privilege on the failover group. Snowflake protects this user in the target account by failing in the following scenarios:

- If the user does not exist in the source account, the refresh operation fails.
- If the user exists in the source account, but a role with the REPLICATE privilege was not granted to the user, the refresh operation fails.
- automatic refresh are recommended
- A secondary failover group cannot be promoted to the primary group while a refresh is executing.
- Failover groups can only be created in a Business Critical Edition (or higher) account. Therefore failover groups can _**only**_ be replicated to an account that is a Business Critical Edition (or higher) account.
- **Current limitations of replication**
    - Databases created from shares cannot be replicated.
    - Refresh operations fail if the primary database includes a stream with an unsupported source object. The operation also fails if the source object for any stream has been dropped.
    - ==Append-only streams are not supported on replicated source objects.==
- Cloned objects are replicated physically rather than logically to secondary databases.
- the materialized view _**data**_ is not replicated
- Automatic Clustering is enabled for a materialized view in a primary database, automatic monitoring and re-clustering of the materialized view in the secondary database is also enabled.
- You can replicate an external stage. However, the files on an external stage are not replicated.
- You can replicate an internal stage. To replicate the files on an internal stage, you must enable a directory table on the stage. Snowflake replicates only the files that are mapped by the directory table.
- A refresh operation will fail if the directory table on an internal stage contains a file that is larger than 5GB.
- Files on user stages and table stages are not replicated.
- Replication of notification integrations is not supported. Snowflake only replicates load history after the latest table truncate.
- Historical usage data for activity in a primary database is not replicated to secondary databases.

  

### SHOW GRANTS

1. SHOW GRANTS TO

List all privileges granted to the `analyst` role:

```SQL
SHOW GRANTS TO ROLE analyst;

+---------------------------------+------------------+------------+------------+------------+--------------+------------+
| created_on                      | privilege        | granted_on | name       | granted_to | grant_option | granted_by |
|---------------------------------+------------------+------------+------------+------------+--------------+------------+
| Wed, 17 Dec 2014 18:19:37 -0800 | CREATE WAREHOUSE | ACCOUNT    | DEMOENV    | ANALYST    | false        | SYSADMIN   |
+---------------------------------+------------------+------------+------------+------------+--------------+------------+
```

  

List all the roles granted to the `demo` user:

```SQL
SHOW GRANTS TO USER demo;

+---------------------------------+------+------------+-------+---------------+
| created_on                      | role | granted_to | name  | granted_by    |
|---------------------------------+------+------------+-------+---------------+
| Wed, 31 Dec 1969 16:00:00 -0800 | DBA  | USER       | DEMO  | SECURITYADMIN |
+---------------------------------+------+------------+-------+---------------+
```

  

1. SHOW GRANTS OF

List all roles and users who have been granted the `analyst` role:

```SQL
SHOW GRANTS OF ROLE analyst;

+---------------------------------+---------+------------+--------------+---------------+
| created_on                      | role    | granted_to | grantee_name | granted_by    |
|---------------------------------+---------+------------+--------------+---------------|
| Tue, 05 Jul 2016 16:16:34 -0700 | ANALYST | ROLE       | ANALYST_US   | SECURITYADMIN |
| Tue, 05 Jul 2016 16:16:34 -0700 | ANALYST | ROLE       | DBA          | SECURITYADMIN |
| Fri, 08 Jul 2016 10:21:30 -0700 | ANALYST | USER       | JOESM        | SECURITYADMIN |
+---------------------------------+---------+------------+--------------+---------------+
```

  

1. `**SHOW GRANTS ON ...ACCOUNT**` Lists all the account-level (i.e. global) privileges that have been granted to roles.
2. **SHOW GRANTS TO ...**`**SHARE**` _`**share_name**`_ Lists all the privileges granted to the share.
3. **SHOW GRANTS OF...**`**SHARE**` _`**share_name**`_ Lists all the accounts for the share and indicates the accounts that are using the share.

  

### URLs

  

|   |   |
|---|---|
|**SQL Function**|**Description**|
|[GET_STAGE_LOCATION](https://docs.snowflake.com/en/sql-reference/functions/get_stage_location)|Returns the URL for an external or internal named stage using the stage name as the input.|
|[GET_RELATIVE_PATH](https://docs.snowflake.com/en/sql-reference/functions/get_relative_path)|Extracts the path of a staged file relative to its location in the stage using the stage name and absolute file path in cloud storage as inputs.|
|[GET_ABSOLUTE_PATH](https://docs.snowflake.com/en/sql-reference/functions/get_absolute_path)|Returns the absolute path of a staged file using the stage name and path of the file relative to its location in the stage as inputs.|
|[GET_PRESIGNED_URL](https://docs.snowflake.com/en/sql-reference/functions/get_presigned_url)|Generates the pre-signed URL to a staged file using the stage name and relative file path as inputs. Access files in an external stage using the function.|
|[BUILD_SCOPED_FILE_URL](https://docs.snowflake.com/en/sql-reference/functions/build_scoped_file_url)|Generates a scoped Snowflake file URL to a staged file using the stage name and relative file path as inputs.|
|[BUILD_STAGE_FILE_URL](https://docs.snowflake.com/en/sql-reference/functions/build_stage_file_url)|Generates a Snowflake file URL to a staged file using the stage name and relative file path as inputs.|

### Listing

Data exchange: [Data Exchange | Snowflake Documentation](https://docs.snowflake.com/en/user-guide/data-exchange)

  

  

  

  

### syntaxes

row access policy

```SQL
-- table
CREATE TABLE sales (
  customer   varchar,
  product    varchar,
  spend      decimal(20, 2),
  sale_date  date,
  region     varchar
)
WITH ROW ACCESS POLICY sales_policy ON (region);

-- view
CREATE VIEW sales_v WITH ROW ACCESS POLICY sales_policy ON (region)
AS SELECT * FROM sales;

-- table

ALTER TABLE t1 ADD ROW ACCESS POLICY rap_t1 ON (empl_id);

-- view

ALTER VIEW v1 ADD ROW ACCESS POLICY rap_v1 ON (empl_id);
```