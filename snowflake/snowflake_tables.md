# Snowflake Tables 
A table in Snowflake is a fundamental database object that stores structured or semi-structured data in rows and columns. When you load data into a table, Snowflake automatically translates it into a highly compressed, optimized, columnar format and stores it in cloud storage.

There are mainly three types of tables in snowflake 


 ##### Temporary Tables
- These are the Tables which are created for temporary use with the Session.
- Time Travel - With in the Session 
- No fail safe 

1. Creating Temporary Table in snowflake 
```SQL
CREATE OR REPLACE TEMPORARY TABLE DWC.NSCC.MAHESH 
(
    ID  NUMBER NOT NULL
);
```
2. Alter the table in snowflake 

```sql
ALTER TABLE DWC.NSCC.MAHESH RENAME TO DWC.NSCC.STUDENTS ;

ALTER TABLE DWC.NSCC.STUDENTS 
ADD COLUMN NAME VARCHAR NOT NULL
;

ALTER TABLE DWC.NSCC.STUDENTS 
ADD 
(
COLUMN CITY VARCHAR(10),
COLUMN PIN VARCHAR(10),
COLUMN BUDDY NUMBER NOT NULL
)
;

ALTER TABLE DWC.NSCC.STUDENTS 
DROP COLUMN BUDDY; 


ALTER TABLE DWC.NSCC.STUDENTS 
RENAME COLUMN PIN TO ADDRESS;

ALTER TABLE DWC.NSCC.STUDENTS 
MODIFY COLUMN ADDRESS SET NOT NULL; 

ALTER TABLE DWC.NSCC.STUDENTS 
MODIFY COLUMN ADDRESS DROP NOT NULL; 

```

##### Transient table 
- Table will create and  exist till we drop.
- Time travel - 0-1 days  ( default 1)
- No fail-safe 

```sql
CREATE OR REPLACE TRANSIENT TABLE DWC.NSCC.MAHESH
(
ID NUMBER NOT NULL
);

ALTER TABLE  DWC.NSCC.MAHESH RENAME TO  DWC.NSCC.STUDENT;

ALTER TABLE  DWC.NSCC.STUDENT
ADD COLUMN NAME VARCHAR NOT NULL;


ALTER TABLE  DWC.NSCC.STUDENT
ADD
(
COLUMN MAHESH VARCHAR,
COLUMN AMOUNT NUMBER ,
COLUMN CITY VARCHAR

);

ALTER TABLE  DWC.NSCC.STUDENT
DROP COLUMN MAHESH;

ALTER TABLE DWC.NSCC.STUDENT
MODIFY COLUMN AMOUNT SET NOT NULL;

ALTER TABLE DWC.NSCC.STUDENT
MODIFY COLUMN CITY SET NOT NULL;

ALTER TABLE DWC.NSCC.STUDENT
MODIFY COLUMN CITY DROP NOT NULL;

ALTER TABLE DWC.NSCC.STUDENT
SET DATA_RETENTION_TIME_IN_DAYS = 1 | 0;

```


##### Permanent  table 
- Table will create and  exist till we drop.
- Time travel - 1-90 days (Enterprise)  (Standerd-0-1days) (default 1)
- Fail-safe   - 7 days (All)


```sql
CREATE OR REPLACE TABLE DWC.NSCC.MAHESH
(
ID NUMBER NOT NULL
);


ALTER TABLE  DWC.NSCC.MAHESH RENAME TO  DWC.NSCC.STUDENT;

ALTER TABLE  DWC.NSCC.STUDENT
ADD COLUMN NAME VARCHAR NOT NULL;


ALTER TABLE  DWC.NSCC.STUDENT
ADD
(
COLUMN MAHESH VARCHAR,
COLUMN AMOUNT NUMBER ,
COLUMN CITY VARCHAR

);

ALTER TABLE  DWC.NSCC.STUDENT
DROP COLUMN MAHESH;

ALTER TABLE DWC.NSCC.STUDENT
MODIFY COLUMN AMOUNT SET NOT NULL;

ALTER TABLE DWC.NSCC.STUDENT
MODIFY COLUMN CITY SET NOT NULL;


ALTER TABLE DWC.NSCC.STUDENT
SET DATA_RETENTION_TIME_IN_DAYS = 1;
ALTER TABLE DWC.NSCC.STUDENT
MODIFY COLUMN CITY DROP NOT NULL;

```


#### Other Tables which are avialble in snowflake 

1. External Tables 

External tables in Snowflake let you query data stored in cloud storage (like Amazon S3, Azure Blob, or Google Cloud Storage) without loading it into Snowflake first. 

External tables are not Physically stored in snowflake storage, Only meta data of the files and structure of the tables  will be stored in snowflake.By using external stage and table structure snowflake will query the data which is avilable in external data platforms. 


- **External Stages:** Snowflake objects that point securely to your cloud storage bucket or container.
- **File Formats:** Definitions that tell Snowflake how to read your data (e.g., CSV, JSON, Parquet)

When you query an external table, Snowflake doesn't scan the whole file every time. It caches file-level metadata (like file names, paths, and update times) within Snowflake. The base data remains in your external cloud storage.

**Read-Only:** External tables are strictly used for querying and joining data. You cannot perform DML operations (like INSERT, UPDATE, or DELETE) on them.

**Variant Column by Default:** When you query an external table, the data initially lands in a single VARIANT column representing the file contents. You can create explicitly defined columns or views on top of this VARIANT column to map fields to proper datatypes.

**Metadata Columns:** Every external table automatically includes built-in metadata columns, specifically METADATA$FILENAME and METADATA$FILE_ROW_NUMBER.

**Auto-Refresh:** You can configure external tables to automatically refresh their metadata whenever a new file is added to your cloud storage using event notifications (e.g., AWS S3 EventBridge)

### Need To Practice this 

```sql
CREATE OR REPLACE EXTERNAL TABLE my_external_table
  WITH LOCATION = @my_external_stage/logs/
  FILE_FORMAT = (TYPE = JSON);
```


2. Iceberg Tables
 
Iceberg tables in Snowflake are a special table type that stores data and metadata in open-source formats (Apache Parquet and Apache Iceberg) on external cloud storage rather than inside Snowflake's proprietary internal storage.This allows you to retain full ownership of your data files while leveraging Snowflake's platform to query and manage them with native performance and security.

Apache Iceberg is an open-source, high-performance table format originally created by Netflix to handle massive analytical datasets in data lakes. Snowflake's integration relies on three main components:

**Parquet Files:** The raw data is stored in the highly optimized, columnar Apache Parquet format.

**Iceberg Metadata:** A robust metadata layer tracks table schemas, file statistics, partitions, and snapshots. This prevents the query engine from having to scan every single file, drastically improving performance.

**External Volumes:** An object inside Snowflake that securely connects to your cloud bucket (Amazon S3, Google Cloud Storage, or Azure Storage) using cloud identity and access management

When you create an Iceberg table in Snowflake, you must choose who manages the table's lifecycle and metadata updates: <br>

**Snowflake as the Catalog (Snowflake-Managed):**
Snowflake has read and write (DML) permissions.Snowflake handles automated maintenance like file compaction, garbage collection, and data snapshots.Other external engines (like Apache Spark or Flink) can still read this data directly from your storage.

**External Catalog (Externally-Managed):**
An external service like AWS Glue, Apache Polaris, or object storage maintains the metadata.Snowflake treats this table primarily as read-only (acting like a highly optimized external table).
 
 ```sql
 CREATE OR REPLACE ICEBERG TABLE my_iceberg_table (
    id INT,
    user_name STRING,
    created_at TIMESTAMP
)
CATALOG = 'SNOWFLAKE'
EXTERNAL_VOLUME = 'my_s3_storage_volume'
BASE_LOCATION = 'analytics/users/';
 ```

 3. Dynamic Tables 

 Dynamic tables in Snowflake are a powerful feature that simplifies building data pipelines. Instead of manually writing code to transform, schedule, and orchestrate data using Streams and Tasks, you simply declare the final SQL query you want, and Snowflake automatically handles the physical materialization and incremental refreshes to keep your data up to date.

 ``` sql
 CREATE OR REPLACE DYNAMIC TABLE regional_sales_summary
  TARGET_LAG = '15 minutes'
  WAREHOUSE = my_compute_warehouse
AS
  SELECT 
    region,
    SUM(sale_amount) AS total_sales,
    COUNT(DISTINCT transaction_id) AS total_transactions
  FROM sales_raw
  GROUP BY region;
 ```

 4. Hybrid Tables 

 Snowflake Hybrid Tables are a specialized table type designed to handle fast, high-concurrency transactional and operational workloads. Unlike standard tables that rely on columnar micro-partitions for analytics, Hybrid Tables utilize a row-based storage engine to deliver low-latency reads and writes, enforce primary and foreign keys, and allow for row-level locking.

 **Row-Level Locking:** Standard Snowflake tables use table-level or partition-level locking, which can cause bottlenecks during heavy updates. Hybrid Tables use row-level locking, allowing multiple concurrent operations on different records.
 
**Enforced Constraints:** Unlike standard tables where some constraints (like primary keys) are purely informational, Hybrid Tables strictly enforce PRIMARY KEY, UNIQUE, and FOREIGN KEY 
constraints.

**Indexed Lookups:** Hybrid Tables automatically create indexes for keys and support user-defined secondary indexes to drastically speed up point-lookups (fetching a single or small number of rows).

**Unistore Capabilities:** They act as the backbone for Snowflake Unistore, a feature allowing developers to build transactional applications directly on top of Snowflake while seamlessly running analytical queries against the same data.


```sql
CREATE OR REPLACE HYBRID TABLE customer_orders (
    order_id VARCHAR(50) PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL,
    order_date TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    status VARCHAR(20)
);
```

5. Event Tables 

Event tables in Snowflake are specialized database tables designed to capture, store, and analyze telemetry data—such as logs, events, and traces—generated by your application code and Snowflake processes (like Stored Procedures, UDFs, and Native Apps).

```sql
-- Set your active Event Table:
ALTER ACCOUNT SET EVENT_TABLE = your_database.your_schema.your_event_table;
-- Create a new Event Table:
CREATE EVENT TABLE your_event_table;
```