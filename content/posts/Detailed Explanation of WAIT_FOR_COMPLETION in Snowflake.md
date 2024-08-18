---
title: Detailed Explanation of WAIT_FOR_COMPLETION in Snowflake
date: 2024-04-07
tags:
  - technical
  - snowflake
--- 
The `WAIT_FOR_COMPLETION` parameter in Snowflake is used with the `ALTER WAREHOUSE` command to control whether the command waits for the warehouse resizing operation to complete before returning control to the user. This parameter is particularly useful when you need to ensure that the warehouse has fully provisioned the new compute resources before continuing with further operations.

### Valid Values

- **TRUE**: The `ALTER WAREHOUSE` command will block (i.e., it will wait) until the resizing operation has fully completed. This ensures that the warehouse is ready with all the new resources before you proceed with executing any queries that require the resized capacity.
- **FALSE**: The `ALTER WAREHOUSE` command returns immediately, without waiting for the resizing operation to complete. This is useful if you don't need to wait for the warehouse to be fully resized and you want to continue with other operations immediately.

### Default Value

- The default value for `WAIT_FOR_COMPLETION` is `FALSE`, meaning the `ALTER WAREHOUSE` command does not wait for the resizing to complete by default.

### Usage

When you resize a warehouse, you use the `ALTER WAREHOUSE` command along with the `WAREHOUSE_SIZE` parameter to specify the new size. The `WAIT_FOR_COMPLETION` parameter can be included in this command to control the blocking behavior.

```SQL
ALTER WAREHOUSE my_warehouse SET WAREHOUSE_SIZE = 'LARGE' WAIT_FOR_COMPLETION = TRUE;
```

In this example, the command will block until the warehouse has fully resized to the 'LARGE' size.

### Notes and Considerations

1. **Persistence**: The value of `WAIT_FOR_COMPLETION` is not persisted. This means you must specify `WAIT_FOR_COMPLETION = TRUE` every time you want the command to wait for the resizing to complete.
2. **Abort Behavior**: If you set `WAIT_FOR_COMPLETION = TRUE` and then abort the `ALTER WAREHOUSE` command (e.g., by canceling the query), only the waiting is aborted. The resizing operation itself will continue. To revert the warehouse to its original size, you need to execute another `ALTER WAREHOUSE` command.
3. **Required Parameter**: `WAIT_FOR_COMPLETION` must be used in conjunction with the `WAREHOUSE_SIZE` parameter. If you try to use `WAIT_FOR_COMPLETION` without `WAREHOUSE_SIZE`, Snowflake will throw an exception.

### Practical Scenario

Consider a scenario where you need to resize a warehouse before running a resource-intensive query. You want to ensure that the warehouse has all the required resources before starting the query to avoid any performance issues.

1. **Resize with Waiting**:
    
    ```SQL
    ALTER WAREHOUSE my_warehouse SET WAREHOUSE_SIZE = 'XLARGE' WAIT_FOR_COMPLETION = TRUE;
    ```
    
    This command ensures that you know exactly when the warehouse is fully resized to 'XLARGE' and ready to handle the intensive query.
    
2. **Continue Without Waiting**:
    
    ```SQL
    ALTER WAREHOUSE my_warehouse SET WAREHOUSE_SIZE = 'XLARGE' WAIT_FOR_COMPLETION = FALSE;
    ```
    
    This command returns immediately, and you can proceed with other tasks while the warehouse is resizing in the background. You might choose this option if the resizing is not critical to your next operations.
    

### Summary

The `WAIT_FOR_COMPLETION` parameter is a useful feature in Snowflake for controlling the blocking behavior of the `ALTER WAREHOUSE` command during resizing. By understanding and correctly using this parameter, you can better manage warehouse resources and ensure that your operations are executed with the appropriate compute capacity.

Would you like to know more about any specific aspects of this feature or see additional examples?