# AWS Copy 

## Creating AWS account 

[AWS ACCOUNT](https://eu-north-1.console.aws.amazon.com/console/services?nc2=h_uta_mc&region=eu-north-1)



####  Load Data from S3 to snowfalke .

1. Create a S3 bucket .
s3://amz-snowpipe-csv/studentcsv/
2. Create a User or role for s3 bucket .

3. Create a table in snowflake .
```sql
CREATE OR REPLACE DATABASE  SCHOOL;

CREATE OR REPLACE SCHEMA  CLASS9;

CREATE OR REPLACE TABLE SCHOOL.CLASS9.MARKS (
ID NUMBER,
NAME VARCHAR(100),
DATE DATE,
TELUGU NUMBER,
ENGLISH NUMBER,
SCIENCE NUMBER,
MATH NUMBER,
SOCIAL NUMBER
);

```
4. Create a file format in snowflake.

```sql
CREATE OR REPLACE FILE FORMAT SCHOOL.CLASS9.STUDENTCSV
    TYPE='CSV'
    FIELD_DELIMITER = ','
    RECORD_DELIMITER = '\n'
    SKIP_HEADER = 1
    NULL_IF = ('NULL', 'null', '');
```

## Through credential 
5. Create a external user  and credential and then Stage 

```sql
CREATE OR REPLACE  STAGE EXTERNAL_S3
FILE_FORMAT= SCHOOL.CLASS9.STUDENTCSV
URL='s3://amz-snowpipe-csv/studentcsv/';

LIST @EXTERNAL_S3;

SHOW STAGES;
```
6. loading data through copy 

```sql
COPY INTO SCHOOL.CLASS9.MARKS
FROM @EXTERNAL_S3
credentials=(aws_key_id='AKIASMJI5MH3P3GI43NS' aws_secret_key='');
```

## Through Storage Integration  

7. create a Storage Integration 

```sql
CREATE OR REPLACE STORAGE INTEGRATION student_int
type=external_stage 
storage_provider=s3
enabled=true
storage_aws_role_arn='arn:aws:iam::163831439862:role/sagar'
storage_allowed_locations=('s3://amz-new-bucket-pipe/raw/');
```
8. Crate a Stage

```sql 
CREATE OR REPLACE  STAGE EXTERNAL_S3_INT
FILE_FORMAT= SCHOOL.CLASS9.STUDENTCSV
Storage_integration=student_int
URL='s3://amz-new-bucket-pipe/raw/';

COPY INTO SCHOOL.CLASS9.MARKS
FROM @EXTERNAL_S3_INT;
```


