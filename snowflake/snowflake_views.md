# Snowflake Views

View is a virtual table Which is created on top of the tables buy using standerd sql queries. These views allows you to access the results without altering the underlying data directly.

Snowflake Supports four different views 

- Standard Views
- Secure Views
- Materialized Views 
- Semantic Views

##### Standard Views 

A Standard View is a named, non-materialized virtual table representing a saved query. It does not store actual data physically. When a standard view is queried, Snowflake dynamically runs the underlying SQL query.

**Pros: ** Simplifies complex joins or queries and ensures logic consistency across multiple analytical workloads.

**Cons: ** Provides no performance boost because it compiles and executes raw logic every time.


```sql
CREATE OR REPLACE TABLE DWC.NSCC.ACCOUNT 
(
ID NUMBER NOT NULL,
CUS_ID NUMBER  NOT NULL,
AMOUNT NUMBER NOT NULL,
SAL_IN VARCHAR 
);

INSERT INTO  DWC.NSCC.ACCOUNT
(ID,CUS_ID,AMOUNT,SAL_IN)
VALUES
(1,123,20000,'N'),
(2,345,20000,'Y'),
(3,456,10000,'N'),
(4,678,50000,'y');

SELECT * FROM DWC.NSCC.ACCOUNT;

CREATE OR REPLACE VIEW  DWC.NSCC.ACCOUNT_SLA
AS
SELECT * FROM  DWC.NSCC.ACCOUNT
WHERE SAL_IN='Y';

SELECT * FROM DWC.NSCC.ACCOUNT_SLA;

```

##### Secure Views 

A Secure View is specifically engineered for data privacy. In standard views, internal query optimizations allow users to infer underlying data patterns through error messages or query execution paths. Secure Views bypass these optimizations, completely hiding the internal query text, table structures, and conditional logic from unauthorized users.

**Pros:** Essential for multi-tenant architectures, regulatory compliance, and Secure Data Sharing.

**Cons:** Disables certain query optimizer techniques, which can slightly slow down query performance.

```sql
CREATE OR REPLACE  SECURE VIEW  DWC.NSCC.ACCOUNT_SLA_N
AS
SELECT * FROM  DWC.NSCC.ACCOUNT
WHERE SAL_IN='N';

SELECT * FROM DWC.NSCC.ACCOUNT_SLA_N;
```

##### Materialized View 

Unlike standard views, a Materialized View physically computes and stores the query result set. Snowflake manages the maintenance background processes, automatically updating the pre-computed results when any underlying base table changes.

**Pros:** Dramatically speeds up repetitive, heavy analytical queries, filtering, and aggregations on large datasets.

**Cons:** Consumes storage resources and incurs background compute costs for automatic data refreshes. It also features architectural restrictions (e.g., cannot utilize joins across multiple tables directly inside the view definition).

```sql
CREATE OR REPLACE  MATERIALIZED VIEW  DWC.NSCC.ACCOUNT_DATA
AS
SELECT
ID
,CUS_ID 
,SUM(AMOUNT) AS AMT
FROM  DWC.NSCC.ACCOUNT
WHERE SAL_IN in ('Y','y')
GROUP BY ID,CUS_ID;
```
##### Semantic View 

A semantic view is a business-friendly layer on top of raw data where we define joins, calculations, and business metrics once, so that all users consume the same trusted definitions repeatedly.