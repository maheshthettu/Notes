# SnowPipe >> S3

1.Create a storage account on s3 and create a Folder in s3.
2.create a table 
3.stage 
4.integration
5.copy command 
6.pipe
7.sns notification 

```sql
-- DDL for SCHOOL database, CLASS9 schema, MARKS table, and CSV file format
-- Co-authored with CoCo
CREATE OR REPLACE DATABASE SCHOOL;
CREATE OR REPLACE SCHEMA CLASS9;
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

CREATE OR REPLACE FILE FORMAT SCHOOL.CLASS9.STUDENTCSV
    TYPE = CSV
    FIELD_DELIMITER = ','
    RECORD_DELIMITER = '\n'
    SKIP_HEADER = 1;

CREATE OR REPLACE STORAGE INTEGRATION student_int
type=external_stage
storage_provider=s3
enabled=true
storage_aws_role_arn='arn:aws:iam::163831439862:role/sagar'
storage_allowed_locations=('s3://amz-new-bucket-pipe/raw/');

desc integration student_int;

CREATE OR REPLACE STAGE EXTERNAL_S3_INT
FILE_FORMAT= SCHOOL.CLASS9.STUDENTCSV
Storage_integration=student_int
URL='s3://amz-new-bucket-pipe/raw/';

desc stage EXTERNAL_S3_INT;

list @EXTERNAL_S3_INT;

CREATE OR REPLACE PIPE SCHOOL.CLASS9.STUDENT_PIPE
AUTO_INGEST = TRUE
AS
COPY INTO SCHOOL.CLASS9.MARKS
FROM @EXTERNAL_S3_INT;

DESC PIPE SCHOOL.CLASS9.STUDENT_PIPE;

ALTER PIPE SCHOOL.CLASS9.STUDENT_PIPE 
REFRESH;

SELECT * FROM SCHOOL.CLASS9.MARKS;

SELECT SYSTEM$PIPE_STATUS('SCHOOL.CLASS9.STUDENT_PIPE');
```
