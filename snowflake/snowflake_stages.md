# Snowflake Stage 

Snowflake stage is a temparory storage location where the raw file will be stored before loading into database tables and after unloading from them. It acts as an intermediary bridge to move data into Snowflake's structured format.

Stages fall into two main categories:
- Internal Stage 
- External Stage 

##### Internal Stages

These stages are completly hosted and managed with in the snowflake.

Internal stages are again three types 
- Named Internal Stage
- User Internal Stage 
- Table Internal stage 

###### Named Internal stage 

This is a custom internal stage which can be created by any user too hold the data files .

```sql
CREATE OR REPLACE STAGE my_named_stage;

LIST @my_named_stage; -- list the files in the stage 
PUT file:///tmp/customers.csv @my_named_stage;
```

###### User Internal stage 

This is private storage location which is automatically assinged to every  individual snowflake user when it created.

``` sql
LIST @~; -- list the files in user stage 
PUT file:///tmp/customers.csv @~;
```

###### Table Internal stage 

This is private storage location which is automatically assinged to every  individual snowflake table when it created.

``` sql
LIST @%; -- list the files in Table stage 

PUT file:///tmp/customers.csv @%CUSTOMERS;

```

##### External Stages

A Snowflake external stage is a virtual bridge that lets Snowflake read or write files in your own external cloud storage (like AWS S3, Google Cloud Storage, or Azure Blob Storage) without actually moving those files into Snowflake's own internal storage

```sql
CREATE OR REPLACE STAGE my_s3_external_stage
  URL='s3://my-company-data-bucket/sales-data/'
  STORAGE_INTEGRATION = my_aws_integration
  FILE_FORMAT = (FILE_FORMAT_NAME);

```

