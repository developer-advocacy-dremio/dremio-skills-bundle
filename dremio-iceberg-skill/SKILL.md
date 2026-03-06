---
name: Dremio Iceberg Operations
description: Teaches an AI agent how to create, manage, and maintain Apache Iceberg tables in Dremio — including DML, schema evolution, time travel, table maintenance, partitioning, and versioned catalog workflows.
---

# Dremio Iceberg Skill

This skill is a deep guide for working with **Apache Iceberg tables** in Dremio. Iceberg is the core table format in Dremio's lakehouse architecture — enabling ACID transactions, schema evolution, time travel, and partition evolution. This skill covers the full lifecycle: create → populate → query → evolve → maintain.

---

## When to Use Iceberg Tables in Dremio

- **You need DML** — INSERT, UPDATE, DELETE, and MERGE only work on Iceberg tables.
- **You need ACID** — Concurrent reads/writes with snapshot isolation.
- **You need time travel** — Query data as it existed at a prior point in time.
- **You need schema evolution** — Add, drop, or rename columns without rewriting data.
- **You need partition evolution** — Change partitioning strategy without rewriting existing data.

Iceberg tables are stored in a **lakehouse catalog source** (Nessie, Arctic, AWS Glue, Iceberg REST, or Open Catalog).

---

## Creating Tables

```sql
-- Basic table
CREATE TABLE catalog.schema.customers (
  id INT,
  name VARCHAR,
  email VARCHAR,
  created_at TIMESTAMP
);

-- With partitioning
CREATE TABLE catalog.schema.events (
  event_id BIGINT,
  event_type VARCHAR,
  user_id INT,
  event_date DATE,
  payload VARCHAR
) PARTITION BY (event_type, MONTH(event_date));

-- With sorting (for better data layout)
CREATE TABLE catalog.schema.orders (
  order_id INT,
  customer_id INT,
  amount DOUBLE,
  order_date DATE
) PARTITION BY (YEAR(order_date))
  LOCALSORT BY (customer_id);

-- CTAS — create from a query
CREATE TABLE catalog.schema.summary AS
SELECT region, COUNT(*) AS cnt, SUM(amount) AS total
FROM catalog.schema.orders
GROUP BY region;
```

**Partition transforms available:** `YEAR()`, `MONTH()`, `DAY()`, `HOUR()`, `BUCKET(n, col)`, `TRUNCATE(n, col)`, or identity (just the column name).

For exact syntax: https://docs.dremio.com/current/reference/sql/commands/create-table

---

## Inserting Data

```sql
-- Insert values
INSERT INTO catalog.schema.customers
VALUES (1, 'Alice', 'alice@example.com', CURRENT_TIMESTAMP);

-- Insert from query
INSERT INTO catalog.schema.summary
SELECT region, COUNT(*), SUM(amount) FROM catalog.schema.orders GROUP BY region;
```

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-insert/

---

## Updating Data

```sql
UPDATE catalog.schema.customers
SET email = 'alice.new@example.com'
WHERE id = 1;
```

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-update/

---

## Deleting Data

```sql
DELETE FROM catalog.schema.customers WHERE id = 1;
```

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-delete/

---

## MERGE (Upsert)

The most powerful DML command — handles INSERT, UPDATE, and DELETE in a single atomic operation.

```sql
MERGE INTO catalog.schema.customers AS target
USING catalog.schema.staging_customers AS source
ON target.id = source.id
WHEN MATCHED AND source.is_deleted = true THEN
  DELETE
WHEN MATCHED THEN
  UPDATE SET target.name = source.name, target.email = source.email
WHEN NOT MATCHED THEN
  INSERT (id, name, email, created_at)
  VALUES (source.id, source.name, source.email, CURRENT_TIMESTAMP);
```

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-merge/

---

## COPY INTO — Bulk Ingestion from Files

Load data from object storage (S3, Azure, GCS) directly into Iceberg tables.

```sql
-- Parquet files
COPY INTO catalog.schema.events
FROM '@my_s3_source/raw/events/'
FILE_FORMAT 'parquet';

-- CSV with options
COPY INTO catalog.schema.events
FROM '@my_s3_source/raw/events.csv'
FILE_FORMAT 'csv'
(RECORD_DELIMITER '\n', FIELD_DELIMITER ',', SKIP_FIRST_LINE);

-- JSON
COPY INTO catalog.schema.events
FROM '@my_s3_source/raw/events/'
FILE_FORMAT 'json';
```

The `@source_name` prefix refers to an object storage source already configured in Dremio.

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/copy-into-table/

---

## Schema Evolution

Modify table schemas without rewriting data files.

```sql
-- Add a column
ALTER TABLE catalog.schema.customers ADD COLUMNS (phone VARCHAR);

-- Drop a column
ALTER TABLE catalog.schema.customers DROP COLUMN phone;

-- Rename a column
ALTER TABLE catalog.schema.customers CHANGE COLUMN name full_name VARCHAR;
```

Docs: https://docs.dremio.com/current/reference/sql/commands/alter-table/

---

## Partition Evolution

Change the partitioning strategy without rewriting existing data. New data uses the new scheme; old data keeps the old scheme. Iceberg handles this transparently.

```sql
-- Add a new partition field
ALTER TABLE catalog.schema.events ADD PARTITION FIELD DAY(event_date);

-- Drop a partition field
ALTER TABLE catalog.schema.events DROP PARTITION FIELD MONTH(event_date);
```

---

## Time Travel

Query data as it existed at a previous point in time.

```sql
-- At a specific snapshot ID
SELECT * FROM catalog.schema.customers AT SNAPSHOT '1234567890123456789';

-- At a specific timestamp
SELECT * FROM catalog.schema.customers AT TIMESTAMP '2024-06-15 12:00:00';

-- Show table properties to see available snapshots
SHOW TBLPROPERTIES catalog.schema.customers;
```

---

## Versioned Catalog Operations (Nessie / Arctic)

When using a Nessie or Arctic catalog, Iceberg tables support Git-like branching and tagging.

```sql
-- Query at a specific branch
SELECT * FROM catalog.schema.customers AT BRANCH main;
SELECT * FROM catalog.schema.customers AT BRANCH dev;

-- Query at a tag
SELECT * FROM catalog.schema.customers AT TAG v1_release;

-- DML operations on a branch
-- (set context to the branch first)
INSERT INTO catalog.schema.customers AT BRANCH dev
VALUES (99, 'Test User', 'test@example.com', CURRENT_TIMESTAMP);
```

---

## Table Maintenance

### OPTIMIZE — Compact Small Files

Small files degrade query performance. OPTIMIZE rewrites them into larger, more efficient files.

```sql
-- Compact all files
OPTIMIZE TABLE catalog.schema.events;

-- Compact with specific target file size (in bytes)
OPTIMIZE TABLE catalog.schema.events
FOR PARTITIONS event_type = 'click';
```

**When to run:** After many small INSERT operations, or on a regular schedule (daily/weekly).

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/optimize-table/

### VACUUM — Remove Expired Snapshots

Clean up old snapshots and orphaned data files to reclaim storage.

```sql
-- Remove snapshots older than a specific timestamp
VACUUM TABLE catalog.schema.events
EXPIRE SNAPSHOTS OLDER_THAN '2024-01-01 00:00:00';

-- Remove orphan files (files not referenced by any snapshot)
VACUUM TABLE catalog.schema.events
REMOVE ORPHAN FILES OLDER_THAN '2024-01-01 00:00:00';
```

**Warning:** After VACUUM, time travel to removed snapshots is no longer possible.

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/vacuum-table/

### ROLLBACK — Undo Changes

Revert a table to a prior snapshot.

```sql
ROLLBACK TABLE catalog.schema.events TO SNAPSHOT 'snapshot_id_here';
```

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/rollback-table/

---

## Truncate

Remove all rows from a table without dropping it.

```sql
TRUNCATE TABLE catalog.schema.events;
```

Docs: https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-truncate/

---

## Reflections on Iceberg Tables

Accelerate queries with materialized reflections.

```sql
-- Raw reflection (full copy with selected columns)
ALTER TABLE catalog.schema.events
  CREATE RAW REFLECTION events_raw
  USING DISPLAY (event_id, event_type, user_id, event_date);

-- Aggregation reflection
ALTER TABLE catalog.schema.events
  CREATE AGGREGATE REFLECTION events_agg
  USING DIMENSIONS (event_type, event_date)
  MEASURES (event_id (COUNT));
```

Docs: https://docs.dremio.com/current/reference/sql/commands/acceleration

---

## Complete Documentation Index

| Topic | URL |
|---|---|
| Iceberg SQL Commands Overview | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/ |
| CREATE TABLE | https://docs.dremio.com/current/reference/sql/commands/create-table |
| CREATE TABLE AS | https://docs.dremio.com/current/reference/sql/commands/create-table-as |
| ALTER TABLE | https://docs.dremio.com/current/reference/sql/commands/alter-table/ |
| COPY INTO | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/copy-into-table/ |
| INSERT | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-insert/ |
| UPDATE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-update/ |
| DELETE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-delete/ |
| MERGE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-merge/ |
| TRUNCATE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-truncate/ |
| OPTIMIZE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/optimize-table/ |
| VACUUM | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/vacuum-table/ |
| ROLLBACK | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/rollback-table/ |
| DROP TABLE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-drop/ |
| SHOW TBLPROPERTIES | https://docs.dremio.com/current/reference/sql/commands/show-table-properties |
| Reflections | https://docs.dremio.com/current/reference/sql/commands/acceleration |
| ANALYZE TABLE | https://docs.dremio.com/current/reference/sql/commands/analyze-table |

---

## Tips

- **DML is Iceberg-only.** You cannot INSERT/UPDATE/DELETE on views, non-Iceberg tables, or external database sources.
- **Use OPTIMIZE after bulk loads.** Many small INSERT operations create many small files — compact them for performance.
- **Schedule VACUUM.** Old snapshots consume storage. Vacuum regularly, but keep enough history for your time-travel needs.
- **Partition wisely.** Over-partitioning (too many distinct values) creates too many small files. Prefer coarse-grained partitions like `MONTH(date)` over `DAY(date)` for small tables.
- **LOCALSORT is powerful.** Sorting within partitions improves filter pushdown on sorted columns even without partitioning on those columns.
- **MERGE is atomic.** Use it for upsert/SCD patterns instead of separate DELETE + INSERT operations.
- **Branch before experimenting.** On Nessie/Arctic catalogs, create a branch, make changes there, and merge only when validated.
