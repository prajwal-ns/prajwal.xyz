---
title: Account usage-information schema
date: 2024-04-07
tags:
  - technical
  - snowflake
---  

- **TABLE_STORAGE_METRICS View**
    - table-level storage utilization information, which is used to calculate the storage billing for each table in the account, including tables that have been dropped, but are still incurring storage costs
    - it contains active bytes, deleted bytes — bytes in time travel, fail-safe, bytes related to clones
- MATERIALIZED_VIEW_REFRESH_HISTORY — This Account Usage view can be used to query the [materialized views](https://docs.snowflake.com/en/user-guide/views-materialized) refresh history. The information returned by the view includes the view name and credits consumed each time a materialized view is refreshed.
- **COPY_HISTORY —** The view displays load activity for both [COPY INTO <table>](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table) statements and continuous data loading using [Snowpipe](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro). The view avoids the 10,000 row limitation of the [LOAD_HISTORY View](https://docs.snowflake.com/en/sql-reference/info-schema/load_history).
 - **LOAD_HISTORY -- ** This Information Schema view enables you to retrieve the history of data loaded into tables using the [COPY INTO ](https://docs.snowflake.com/en/sql-reference/sql/copy-into-table) command — this doesn’t include history about snowpipe
- **ACCESS_HISTORY** — Access History in Snowflake refers to when the user query reads data and when the SQL statement performs a data write operation along with variations of the COPY command, from the source data object to the target data object.
- **LOGIN_HISTORY , LOGIN_HISTORY_BY_USER —**
    - The LOGIN_HISTORY family of table functions can be used to query login attempts by Snowflake users along various dimensions:
    - LOGIN_HISTORY returns login events within a specified time range.
    - LOGIN_HISTORY_BY_USER returns login events of a specified user within a specified time range.
- **WAREHOUSE_LOAD_HISTORY** — This table function can be used to query the activity history (defined as the “query load”) for a single warehouse within a specified date range. columns — AVG_RUNNING, AVG_QUEUED_LOAD, AVG_QUEUED_PROVISIONING, AVG_BLOCKED
- **WAREHOUSE_METERING_HISTORY** — This table function can be used in queries to return the hourly credit usage for a single warehouse (or all the warehouses in your account) within a specified date range. columns — CREDITS_USED, CREDITS_USED_COMPUTE, CREDITS_USED_CLOUD_SERVICES.
- **REPLICATION_USAGE_HISTORY — can be used to query the replication history for a specified database within a specified date range.** The information returned by the function includes the database name, credits consumed and bytes transferred for replication.
- **AUTO_REFRESH_REGISTRATION_HISTORY** — can be used to query the history of data files registered in the metadata of specified objects and the credits billed for these operations.
- **STAGE_DIRECTORY_FILE_REGISTRATION_HISTORY**— can be used to query information about the metadata history for a directory table, including — Files added or removed automatically as part of a metadata refresh, any errors found when refreshing the metadata.