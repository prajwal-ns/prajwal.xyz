---
title: TO_JSON and PARSE_JSON
date: 2024-04-07
tags:
  - technical
  - snowflake
---
The functions `TO_JSON` and `PARSE_JSON` in Snowflake are used for handling JSON data, but they serve different purposes.

### `TO_JSON`

- **Purpose**: Converts a Snowflake SQL data type into its JSON representation.
- **Usage**: Typically used when you want to serialize data from a Snowflake table into a JSON string format.
- **Example**:  
    This would output:  
    
    ```SQL
    SELECT TO_JSON(OBJECT_CONSTRUCT('key1', 'value1', 'key2', 'value2')) AS json_output;
    ```
    
    ```JSON
    {"key1":"value1","key2":"value2"}
    ```
    

### `PARSE_JSON`

- **Purpose**: Converts a string containing JSON data into a Snowflake `VARIANT` data type, which allows for querying JSON data.
- **Usage**: Used when you want to deserialize a JSON string into a format that can be queried using SQL.
- **Example**:  
    This converts the JSON string into a  
    `VARIANT` type, allowing you to query it like:
    
    ```SQL
    SELECT PARSE_JSON('{"key1":"value1","key2":"value2"}') AS variant_output;
    ```
    
    ```SQL
    SELECT variant_output:key1 FROM (SELECT PARSE_JSON('{"key1":"value1","key2":"value2"}') AS variant_output);
    ```
    

### Key Differences:

1. **Functionality**:
    - `TO_JSON`: Converts structured data (like objects or arrays) to JSON string format.
    - `PARSE_JSON`: Converts JSON string data to Snowflake's `VARIANT` data type, enabling JSON data querying.
2. **Direction**:
    - `TO_JSON`: Serialization (Structured data to JSON string).
    - `PARSE_JSON`: Deserialization (JSON string to `VARIANT`).
3. **Use Cases**:
    - `TO_JSON`: When you need to generate a JSON string from table data for storage, export, or API interactions.
    - `PARSE_JSON`: When you need to load and query JSON data from strings stored in tables.

### Practical Example:

Suppose you have a table `users` with columns `id`, `name`, and `attributes` (where `attributes` is a JSON string).

- **Using** `**TO_JSON**`:
    
    ```SQL
    SELECT id, name, TO_JSON(attributes) AS json_attributes
    FROM users;
    ```
    
    This converts the `attributes` column to a JSON string.
    
- **Using** `**PARSE_JSON**`:
    
    ```SQL
    SELECT id, name, PARSE_JSON(attributes) AS parsed_attributes
    FROM users;
    ```
    
    This converts the JSON string in the `attributes` column to a `VARIANT` type, allowing JSON querying:
    
    ```SQL
    SELECT id, name, parsed_attributes:key1
    FROM (SELECT id, name, PARSE_JSON(attributes) AS parsed_attributes FROM users);
    ```
    

In summary, use `TO_JSON` to convert data to JSON strings and `PARSE_JSON` to convert JSON strings to a queryable `VARIANT` type in Snowflake.