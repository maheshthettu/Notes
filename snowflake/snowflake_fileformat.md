# File Format

In Snowflake, a File Format is a reusable database object that describes the structure, layout, and properties of data files so Snowflake knows exactly how to read them during loading or write them during unloading. 

Instead of defining how to parse a file every single time you execute a command, you encapsulate settings like column delimiters, header rows, and compression types into a single, named schema object.

#### Supported File Types

##### CSV FORMAT 

The CSV format is used for plain text delimited data files. You can customize field delimiters, headers, and string encodings.

```sql
CREATE OR REPLACE FILE FORMAT my_csv_format
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    RECORD_DELIMITER = '\n'
    SKIP_HEADER = 1
    NULL_IF = ('NULL', 'null', '')
    EMPTY_FIELD_AS_NULL = TRUE;
```

##### JSON FORMAT

The JSON format handles semi-structured data structures. A critical option is stripping null values to prevent wasting storage on explicit JSON "null" strings.

```sql
CREATE OR REPLACE FILE FORMAT prod_json_format
  TYPE = 'JSON'
  COMPRESSION = AUTO
  STRIP_OUTER_ARRAY = TRUE
  ENABLE_OCTAL = FALSE
  ALLOW_DUPLICATE_KEYS = FALSE
  STRIP_NULL_VALUES = FALSE
  IGNORE_UTF8_ERRORS = FALSE ;
```

##### PARQUET 

PARQUET is a popular columnar file storage format optimized for analytical queries. Snowflake natively extracts binary and logical types.


```sql
CREATE OR REPLACE FILE FORMAT my_parquet_format
  TYPE = 'PARQUET'
  COMPRESSION = 'AUTO'
  BINARY_AS_TEXT = TRUE
  USE_LOGICAL_TYPE = TRUE
  TRIM_SPACE = FALSE;
```

##### AVRO

AVRO is a row-based data serialization framework frequently used in Apache Kafka pipelines.

```sql
CREATE OR REPLACE FILE FORMAT my_avro_format
  TYPE = 'AVRO'
  COMPRESSION = 'AUTO'
  TRIM_SPACE = TRUE
  REPLACE_INVALID_CHARACTERS = TRUE;
```

##### ORC
ORC provides a highly efficient way to store Hive data, reducing storage footprints and speeding up data processing.

```sql
CREATE OR REPLACE FILE FORMAT my_orc_format
  TYPE = 'ORC'
  TRIM_SPACE = TRUE
  NULL_IF = ('\\N')
  REPLACE_INVALID_CHARACTERS = TRUE;

```
##### XML
The XML format reads structured document markup hierarchies into Snowflake's variant fields.

```sql
CREATE OR REPLACE FILE FORMAT my_xml_format
  TYPE = 'XML'
  COMPRESSION = 'AUTO'
  PRESERVE_SPACE = FALSE
  STRIP_OUTER_ELEMENT = TRUE
  DISABLE_AUTO_CONVERT = FALSE
  SKIP_BYTE_ORDER_MARK = TRUE;
```