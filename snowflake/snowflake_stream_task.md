# Snowflake Streams

Snowflake Streams are Change Data Capture (CDC) objects that record row-level DML changes (inserts, updates, deletes) made to a source table, view, or dynamic table. They enable incremental data pipelines by allowing you to query only the new or modified data since the last read.


- **METADATA$ACTION**: The operation performed (INSERT or DELETE).
- **METADATA$ISUPDATE**: TRUE if the row is part of an update operation (a delete of the old row and insert of the new row).
- **METADATA$ROW_ID**: A unique, immutable ID for tracking the specific row.

#### Consume Stream

To use the stream, you query it much like a regular table. Reading a stream also consumes/advances its offset—meaning the next time you query it, you will only get the new changes.

##### Standard Stream 

```sql
CREATE OR REPLACE STREAM emp_standard_stream ON TABLE employees;
```

##### Append Only Stream 

```sql
CREATE OR REPLACE STREAM emp_standard_stream 
ON TABLE employees
APPEND_ONLY = TRUE ;
```


# Task 

A Snowflake Task is a built-in scheduling and orchestration object used to automate SQL statements or stored procedures. It eliminates the need for external cron jobs, allowing you to run individual queries at set intervals or chain multiple interdependent tasks together into directed acyclic graphs (DAGs).

##### Basic Task 

```sql
CREATE OR REPLACE TASK my_hourly_task
  WAREHOUSE = my_compute_wh
  SCHEDULE = '60 MINUTE'
  AS
  INSERT INTO sales_summary 
  SELECT * FROM raw_sales WHERE created_at >= DATEADD(hour, -1, CURRENT_TIMESTAMP());


  CREATE OR REPLACE TASK clean_logs_task
  WAREHOUSE = my_compute_wh
  SCHEDULE = 'USING CRON 0 0 * * * UTC'
AS
  DELETE FROM app_logs WHERE log_date < DATEADD(day, -30, CURRENT_DATE());

```
###### Task Graphs (Chained/Dependent Tasks)

```sql
-- Root Task
CREATE OR REPLACE TASK extract_task
  WAREHOUSE = my_compute_wh
  SCHEDULE = '60 MINUTE'
  AS
  INSERT INTO extracted_data SELECT * FROM source_api;

-- Child Task (runs after the root task)
CREATE OR REPLACE TASK transform_task
  WAREHOUSE = my_compute_wh
  AFTER extract_task
  AS
  CALL run_transformations();

```
##### Server Less task 
Instead of managing and paying for a dedicated virtual warehouse, you can use serverless tasks. Snowflake dynamically sizes and scales the resources to match your workload.

```sql
CREATE OR REPLACE TASK serverless_import_task
  SCHEDULE = '15 MINUTE'
  -- Omit WAREHOUSE and use Snowflake-managed compute
  AS
  COPY INTO my_table FROM @my_stage;
```


###### Managing Tasks (Resume & Execute):
Newly created tasks are suspended by default. You must resume them, or they will never run.
- Start a Task Graph: ALTER TASK my_hourly_task RESUME;
- Suspend a Task Graph: ALTER TASK my_hourly_task SUSPEND;
- Manually Trigger a Task: EXECUTE TASK my_hourly_task