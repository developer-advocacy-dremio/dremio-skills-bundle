---
name: Dremio SQL Reference
description: Teaches an AI agent the Dremio SQL dialect — unique syntax, Iceberg DML, reflections DDL, versioned queries, RBAC commands, and links to the full SQL reference docs.
---

# Dremio SQL Skill

This skill equips you to write correct SQL for Dremio. Dremio uses a SQL dialect built on Apache Calcite with significant extensions for **Apache Iceberg tables**, **versioned catalogs**, **reflections**, **RBAC**, and **PIPE ingestion**. Many of these features have syntax that differs from standard ANSI SQL or other engines like Snowflake, BigQuery, or PostgreSQL.

**Always read the linked documentation page** for exact syntax before writing a query.

---

## Key Dialect Differences

Things that catch agents off guard when writing SQL for Dremio:

| Feature | Dremio Syntax | Common Mistake |
|---|---|---|
| Table paths use dots | `"my-catalog".schema.table` | Forgetting double quotes around names with hyphens/special chars |
| Versioned queries | `SELECT * FROM t AT BRANCH main` | Using non-existent `FOR SYSTEM_TIME` instead of `AT` |
| Time travel | `AT SNAPSHOT '...'` or `AT TIMESTAMP '...'` | Using Snowflake-style `BEFORE(TIMESTAMP => ...)` |
| Iceberg DML | `INSERT`, `UPDATE`, `DELETE`, `MERGE` on Iceberg tables | Trying DML on non-Iceberg sources |
| COPY INTO | `COPY INTO table FROM '@source/path'` | Incorrect path format or file format spec |
| Reflections | `ALTER TABLE ... CREATE ... REFLECTION` | Using non-existent `CREATE REFLECTION` command |
| UDFs | `CREATE FUNCTION name(x INT) RETURNS INT RETURN SELECT x + 1` | Using `AS $$ ... $$` syntax from PostgreSQL |
| Column masking | `ALTER TABLE ... SET MASKING POLICY ...` | Column-level security has unique Dremio syntax |
| Path quoting | Double quotes: `"my-source"."my.schema"."table"` | Using backticks (MySQL) or square brackets (SQL Server) |

---

## SQL Command Quick Reference

### DDL — Tables & Views

```sql
-- Create an Iceberg table
CREATE TABLE catalog.schema.my_table (
  id INT,
  name VARCHAR,
  created_at TIMESTAMP
) PARTITION BY (MONTH(created_at));

-- Create table from query
CREATE TABLE catalog.schema.summary AS
SELECT region, SUM(amount) AS total FROM sales GROUP BY region;

-- Create a view
CREATE VIEW space.my_view AS
SELECT * FROM catalog.schema.my_table WHERE active = true;

-- Alter table (add/drop columns, change partition)
ALTER TABLE catalog.schema.my_table ADD COLUMNS (email VARCHAR);
ALTER TABLE catalog.schema.my_table DROP COLUMN email;

-- Drop
DROP TABLE catalog.schema.my_table;
DROP VIEW space.my_view;
```

### DML — Iceberg Tables

```sql
-- Insert
INSERT INTO catalog.schema.my_table VALUES (1, 'Alice', CURRENT_TIMESTAMP);
INSERT INTO catalog.schema.my_table SELECT * FROM staging_table;

-- Update
UPDATE catalog.schema.my_table SET name = 'Bob' WHERE id = 1;

-- Delete
DELETE FROM catalog.schema.my_table WHERE id = 1;

-- Merge (upsert)
MERGE INTO catalog.schema.target AS t
USING catalog.schema.source AS s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET t.name = s.name
WHEN NOT MATCHED THEN INSERT (id, name) VALUES (s.id, s.name);

-- Truncate
TRUNCATE TABLE catalog.schema.my_table;
```

### COPY INTO — Bulk Ingestion

```sql
-- Load from object storage into an Iceberg table
COPY INTO catalog.schema.my_table
FROM '@my_s3_source/data/files/'
FILE_FORMAT 'parquet';

-- CSV with options
COPY INTO catalog.schema.my_table
FROM '@my_s3_source/data/'
FILE_FORMAT 'csv'
(RECORD_DELIMITER '\n', FIELD_DELIMITER ',', SKIP_FIRST_LINE);
```

### Table Maintenance

```sql
-- Compact small files (Iceberg)
OPTIMIZE TABLE catalog.schema.my_table;

-- Remove old snapshots and orphaned files
VACUUM TABLE catalog.schema.my_table EXPIRE SNAPSHOTS OLDER_THAN '2024-01-01 00:00:00';

-- Rollback to prior snapshot
ROLLBACK TABLE catalog.schema.my_table TO SNAPSHOT 'snapshot_id_here';

-- Analyze table statistics
ANALYZE TABLE catalog.schema.my_table COMPUTE STATISTICS;
```

### Versioned Queries (Nessie/Arctic)

```sql
-- Query at a specific branch
SELECT * FROM catalog.schema.my_table AT BRANCH main;

-- Query at a tag
SELECT * FROM catalog.schema.my_table AT TAG v1_release;

-- Query at a snapshot ID
SELECT * FROM catalog.schema.my_table AT SNAPSHOT '1234567890';

-- Time travel
SELECT * FROM catalog.schema.my_table AT TIMESTAMP '2024-06-15 12:00:00';
```

### Reflections (Query Acceleration)

```sql
-- Create a raw reflection
ALTER TABLE catalog.schema.my_table
  CREATE RAW REFLECTION my_raw_ref
  USING DISPLAY (col1, col2, col3);

-- Create an aggregation reflection
ALTER TABLE catalog.schema.my_table
  CREATE AGGREGATE REFLECTION my_agg_ref
  USING DIMENSIONS (region) MEASURES (amount (SUM, COUNT));

-- Drop a reflection
ALTER TABLE catalog.schema.my_table DROP REFLECTION my_raw_ref;
```

### RBAC — Grants & Privileges

```sql
-- Grant SELECT on a dataset
GRANT SELECT ON catalog.schema.my_table TO ROLE analysts;

-- Grant on a space/folder
GRANT ALL ON FOLDER "Analytics" TO ROLE data_engineers;

-- Revoke
REVOKE SELECT ON catalog.schema.my_table FROM ROLE analysts;

-- Create a role
CREATE ROLE data_viewers;

-- Grant role to user
GRANT ROLE data_viewers TO USER "john@example.com";
```

### User-Defined Functions (UDFs)

```sql
-- Create a scalar UDF
CREATE FUNCTION double_it(x INT)
  RETURNS INT
  RETURN SELECT x * 2;

-- Create a tabular UDF
CREATE FUNCTION recent_orders(days INT)
  RETURNS TABLE(id INT, amount DOUBLE)
  RETURN SELECT id, amount FROM orders
    WHERE order_date > CURRENT_DATE - CAST(days AS INTERVAL DAY);

-- Drop
DROP FUNCTION double_it;
```

### Row-Access & Column-Masking Policies

```sql
-- Create a row-access policy function
CREATE FUNCTION region_filter(region_col VARCHAR)
  RETURNS BOOLEAN
  RETURN SELECT region_col = QUERY_USER();

-- Apply row-access policy
ALTER TABLE catalog.schema.my_table
  SET ROW ACCESS POLICY region_filter(region);

-- Create a masking policy function
CREATE FUNCTION mask_email(email_col VARCHAR)
  RETURNS VARCHAR
  RETURN SELECT CASE WHEN QUERY_USER() = 'admin' THEN email_col ELSE '***' END;

-- Apply masking policy
ALTER TABLE catalog.schema.my_table
  SET COLUMN MASKING POLICY mask_email(email);
```

### PIPE (Continuous Ingestion)

```sql
-- Create a pipe for continuous ingestion
CREATE PIPE my_pipe AS
  COPY INTO catalog.schema.target
  FROM '@my_s3_source/streaming/'
  FILE_FORMAT 'json';

-- Manage pipes
ALTER PIPE my_pipe DEDUPE_LOOKBACK_PERIOD 7;
DESCRIBE PIPE my_pipe;
DROP PIPE my_pipe;
```

### Utility

```sql
-- Set context (default schema)
USE catalog.schema;

-- Set query queue
SET QUEUE "high_priority";

-- Common table expressions
WITH recent AS (
  SELECT * FROM orders WHERE order_date > '2024-01-01'
)
SELECT customer_id, SUM(amount) FROM recent GROUP BY customer_id;

-- Show table properties (Iceberg metadata)
SHOW TBLPROPERTIES catalog.schema.my_table;
```

---

## SQL Documentation Index

When you need the exact syntax, parameters, or examples for a specific command or function, **read the relevant page** using your URL-reading tools.

### SQL Commands — General

| Command | URL |
|---|---|
| SQL Commands Overview | https://docs.dremio.com/current/reference/sql/commands/ |
| ALTER FOLDER | https://docs.dremio.com/current/reference/sql/commands/alter-folder |
| ALTER Reflections | https://docs.dremio.com/current/reference/sql/commands/acceleration |
| ALTER PIPE | https://docs.dremio.com/current/reference/sql/commands/alter-pipe |
| ALTER SPACE | https://docs.dremio.com/current/reference/sql/commands/alter-space |
| ALTER SOURCE | https://docs.dremio.com/current/reference/sql/commands/alter-source |
| ALTER TABLE | https://docs.dremio.com/current/reference/sql/commands/alter-table |
| ALTER VIEW | https://docs.dremio.com/current/reference/sql/commands/alter-view |
| ANALYZE TABLE | https://docs.dremio.com/current/reference/sql/commands/analyze-table |
| CREATE TABLE | https://docs.dremio.com/current/reference/sql/commands/create-table |
| CREATE TABLE AS | https://docs.dremio.com/current/reference/sql/commands/create-table-as |
| CREATE PIPE | https://docs.dremio.com/current/reference/sql/commands/create-pipe |
| CREATE VIEW | https://docs.dremio.com/current/reference/sql/commands/create-view |
| CREATE/ALTER/DROP Users | https://docs.dremio.com/current/reference/sql/commands/users |
| DESCRIBE PIPE | https://docs.dremio.com/current/reference/sql/commands/describe-pipe |
| DROP PIPE | https://docs.dremio.com/current/reference/sql/commands/drop-pipe |
| DROP VIEW | https://docs.dremio.com/current/reference/sql/commands/drop-view |
| DROP TABLE | https://docs.dremio.com/current/reference/sql/commands/tables |
| GRANT / REVOKE | https://docs.dremio.com/current/reference/sql/commands/rbac |
| Row-Access & Column-Masking | https://docs.dremio.com/current/reference/sql/commands/functions |
| SELECT | https://docs.dremio.com/current/reference/sql/commands/SELECT-statements |
| SET QUEUE / RESET QUEUE | https://docs.dremio.com/current/reference/sql/commands/set-queue |
| SET TAG / RESET TAG | https://docs.dremio.com/current/reference/sql/commands/set-tag |
| SHOW TBLPROPERTIES | https://docs.dremio.com/current/reference/sql/commands/show-table-properties |
| USE | https://docs.dremio.com/current/reference/sql/commands/use |
| WITH | https://docs.dremio.com/current/reference/sql/commands/with |

### SQL Commands — Iceberg Tables

| Command | URL |
|---|---|
| Iceberg Overview | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/ |
| COPY INTO | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/copy-into-table/ |
| DELETE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-delete/ |
| DROP (Iceberg) | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-drop/ |
| INSERT | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-insert/ |
| MERGE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-merge/ |
| OPTIMIZE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/optimize-table/ |
| ROLLBACK | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/rollback-table/ |
| TRUNCATE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-truncate/ |
| UPDATE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-update/ |
| VACUUM | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/vacuum-table/ |

### SQL Functions

| Category | URL |
|---|---|
| All Functions | https://docs.dremio.com/current/reference/sql/sql-functions/ALL_FUNCTIONS |
| Aggregate | https://docs.dremio.com/current/reference/sql/sql-functions/AGGREGATE |
| AI | https://docs.dremio.com/current/reference/sql/sql-functions/AI |
| Binary | https://docs.dremio.com/current/reference/sql/sql-functions/BINARY |
| Bitwise | https://docs.dremio.com/current/reference/sql/sql-functions/BITWISE |
| Boolean | https://docs.dremio.com/current/reference/sql/sql-functions/BOOLEAN |
| Conditional | https://docs.dremio.com/current/reference/sql/sql-functions/CONDITIONAL |
| Conversion | https://docs.dremio.com/current/reference/sql/sql-functions/CONVERSION |
| Cryptography | https://docs.dremio.com/current/reference/sql/sql-functions/CRYPTOGRAPHY |
| Datatype | https://docs.dremio.com/current/reference/sql/sql-functions/DATATYPE |
| Date/Time | https://docs.dremio.com/current/reference/sql/sql-functions/DATE_TIME |
| Directory | https://docs.dremio.com/current/reference/sql/sql-functions/DIRECTORY |
| Geospatial | https://docs.dremio.com/current/reference/sql/sql-functions/GEOSPATIAL |
| Math | https://docs.dremio.com/current/reference/sql/sql-functions/MATH |
| Percentile | https://docs.dremio.com/current/reference/sql/sql-functions/PERCENTILE |
| Regular Expressions | https://docs.dremio.com/current/reference/sql/sql-functions/REGULAR_EXPRESSIONS |
| Semi-Structured Data | https://docs.dremio.com/current/reference/sql/sql-functions/SEMI-STRUCTURED_DATA |
| String | https://docs.dremio.com/current/reference/sql/sql-functions/STRING |
| System | https://docs.dremio.com/current/reference/sql/sql-functions/SYSTEM |
| Window | https://docs.dremio.com/current/reference/sql/sql-functions/WINDOW |

### Other SQL Reference

| Topic | URL |
|---|---|
| Data Types | https://docs.dremio.com/current/reference/sql/data-types/ |
| Reserved Words | https://docs.dremio.com/current/reference/sql/reserved-keywords |
| System Tables | https://docs.dremio.com/current/reference/sql/system-tables |
| Table Functions | https://docs.dremio.com/current/reference/sql/table-functions |
| Information Schema | https://docs.dremio.com/current/reference/sql/information-schema |

---

## Tips

- Always double-quote identifiers with hyphens, dots, or special characters: `"my-catalog"."my.schema"`
- DML (INSERT, UPDATE, DELETE, MERGE) only works on **Iceberg tables**, not views or non-Iceberg sources.
- `COPY INTO` is the preferred way to bulk-load files (Parquet, CSV, JSON) from object storage into Iceberg tables.
- `OPTIMIZE` compacts small files; `VACUUM` removes expired snapshots. Both are essential for Iceberg table maintenance.
- Versioned queries (`AT BRANCH`, `AT TAG`, `AT SNAPSHOT`) only work with Nessie/Arctic catalogs.
- Reflections are created via `ALTER TABLE ... CREATE ... REFLECTION`, not a standalone `CREATE REFLECTION`.
- `QUERY_USER()` is a special function that returns the current user — useful in row-access and masking policies.
- When unsure about exact syntax, always read the specific doc page from the index above.
