---
name: Dremio Lakehouse Modeling
description: Teaches an AI agent data modeling best practices in a Dremio lakehouse — medallion architecture, views vs tables, reflections strategy, partitioning, dimensional modeling, and semantic layer design.
---

# Dremio Lakehouse Modeling Skill

This skill provides opinionated guidance on **how to model data** in a Dremio lakehouse. It covers architecture patterns, when to use views vs tables, how to design reflections, partitioning strategies, and semantic layer design. Use this when helping users design or refactor their data pipelines and models.

---

## Medallion Architecture (Bronze → Silver → Gold)

The most common pattern for organizing a Dremio lakehouse. Each layer has a specific purpose and is implemented using different Dremio objects.

### Bronze — Raw Ingestion Layer

**What:** Raw data landed from sources with minimal transformation. Data types should match the source.

**Dremio implementation:** Iceberg tables in a lakehouse catalog (Nessie, Arctic, Glue).

```sql
-- Ingest raw data from object storage
COPY INTO nessie.bronze.raw_transactions
FROM '@s3source/raw/transactions/'
FILE_FORMAT 'parquet';

-- Or ingest via API/database replication into Iceberg tables
INSERT INTO nessie.bronze.raw_customers
SELECT * FROM postgres_source.public.customers;
```

**Conventions:**
- Prefix tables with the source system name: `raw_stripe_payments`, `raw_salesforce_accounts`
- Include an `_ingested_at` timestamp column for lineage
- Partition by ingestion month: `PARTITION BY (MONTH(_ingested_at))`
- Don't filter, dedupe, or transform — the purpose is a faithful copy of source data

### Silver — Cleaned & Conformed Layer

**What:** Deduplicated, typed, validated data. Business logic starts here: column renames, type casts, null handling, deduplication.

**Dremio implementation:** Iceberg tables (materialized from Bronze) **or** Views (for lightweight transformations).

```sql
-- Materialized as an Iceberg table (best for large, reused datasets)
CREATE TABLE nessie.silver.customers AS
SELECT DISTINCT
  CAST(id AS INT) AS customer_id,
  TRIM(name) AS customer_name,
  LOWER(email) AS email,
  CAST(created_at AS TIMESTAMP) AS created_at
FROM nessie.bronze.raw_customers
WHERE id IS NOT NULL;

-- Or lightweight as a view (good for simple transformations)
CREATE VIEW analytics.silver.clean_transactions AS
SELECT
  transaction_id,
  amount,
  CASE WHEN status = 'completed' THEN true ELSE false END AS is_completed,
  event_date
FROM nessie.bronze.raw_transactions;
```

**Conventions:**
- Use business-friendly column names (not source system names)
- Enforce NOT NULL semantics by filtering in the transformation
- Log transformation rules in Dremio Wiki for documentation

### Gold — Business / Consumption Layer

**What:** Aggregated, joined, and ready-for-consumption datasets. Optimized for specific business questions, dashboards, or ML features.

**Dremio implementation:** Views (preferred) — allows the Dremio query engine and reflections to optimize automatically.

```sql
-- Gold view: revenue by region by month
CREATE VIEW analytics.gold.monthly_revenue AS
SELECT
  c.region,
  DATE_TRUNC('MONTH', t.event_date) AS month,
  SUM(t.amount) AS total_revenue,
  COUNT(DISTINCT t.customer_id) AS unique_customers
FROM analytics.silver.clean_transactions t
JOIN nessie.silver.customers c ON t.customer_id = c.customer_id
WHERE t.is_completed = true
GROUP BY c.region, DATE_TRUNC('MONTH', t.event_date);
```

**Conventions:**
- Gold views are the **primary interface** for BI tools and analysts
- Keep them simple — complex logic should live in Silver
- Add reflections on Gold views for performance (see below)

---

## Views vs Tables — Decision Framework

| Criteria | Use a **View** | Use an **Iceberg Table** |
|---|---|---|
| Data changes frequently | ✅ Always shows latest source data | Must re-run INSERT/MERGE to refresh |
| Needs DML (UPDATE/DELETE) | ❌ Views are read-only | ✅ Full DML support |
| Large aggregation/join cost | Add a reflection on the view | Materialize once, query fast |
| Needs time travel | ❌ Views don't have snapshots | ✅ AT SNAPSHOT / AT TIMESTAMP |
| Used by many downstream consumers | ✅ Views are cheap to create | Table + OPTIMIZE for large reads |
| Needs partitioning/sorting control | ❌ Views inherit source layout | ✅ Full control over data layout |
| Transformation is simple (rename, cast) | ✅ Near-zero cost | Overhead of table creation |
| Data pipeline with incremental loads | Tables with MERGE for upserts | Views can't receive data |

**Rule of thumb:**
- **Gold layer → Views** (with reflections for performance)
- **Silver layer → Mix** (tables for large datasets, views for simple transforms)
- **Bronze layer → Iceberg tables** (they receive ingested data)

---

## Reflections Strategy

Reflections are Dremio's query acceleration layer. They're materialized subsets of your data that the query engine uses automatically — users don't need to know they exist.

### When to Create Reflections

| Scenario | Reflection Type |
|---|---|
| Dashboard with aggregations (SUM, COUNT, AVG) | **Aggregation** reflection |
| Frequently queried subset of columns | **Raw** reflection with selected columns |
| Join between large tables | **Raw** reflection on the fact table with join columns |
| Filter on high-cardinality column | **Raw** reflection sorted by that column |

### Aggregation Reflections

Best for dashboards and reports that repeatedly aggregate along the same dimensions.

```sql
ALTER TABLE nessie.silver.transactions
  CREATE AGGREGATE REFLECTION txn_agg
  USING DIMENSIONS (region, DATE_TRUNC('MONTH', event_date))
  MEASURES (amount (SUM, COUNT, AVG), customer_id (COUNT));
```

### Raw Reflections

Best for speeding up point lookups, filtered scans, or joins.

```sql
ALTER TABLE nessie.silver.transactions
  CREATE RAW REFLECTION txn_raw
  USING DISPLAY (transaction_id, customer_id, amount, event_date)
  DISTRIBUTE BY (customer_id)
  LOCALSORT BY (event_date);
```

### Anti-Patterns

- **Don't create reflections on everything** — they consume storage and compute to maintain
- **Don't duplicate the entire table** — use column selection to reduce reflection size
- **Monitor usage** — check `sys.reflections` to see which reflections are being matched

---

## Partitioning Strategy

### General Guidelines

| Table Size | Recommended Partitioning |
|---|---|
| < 100 GB | No partitioning needed |
| 100 GB – 1 TB | Single partition column (e.g., `MONTH(date)`) |
| 1 TB – 10 TB | Two-level (e.g., `event_type` + `MONTH(date)`) |
| > 10 TB | Consider `BUCKET(n, id)` for high-cardinality or three-level |

### Rules

1. **Avoid over-partitioning.** Each partition should contain at least ~100 MB of data. If partitions are too small, you get many tiny files → poor performance.
2. **Use transform functions.** `MONTH(date)` is almost always better than `DAY(date)` for event data.
3. **Partition by query patterns.** If users always filter by `region`, partition by `region`.
4. **Use LOCALSORT for secondary sort.** If queries filter by date but also often filter by `user_id`, partition by date and LOCALSORT by `user_id`.

```sql
CREATE TABLE nessie.silver.events (
  event_id BIGINT,
  event_type VARCHAR,
  user_id INT,
  event_date DATE,
  payload VARCHAR
) PARTITION BY (MONTH(event_date))
  LOCALSORT BY (user_id);
```

---

## Dimensional Modeling in Dremio

### Star Schema Pattern

Build fact and dimension tables as Iceberg tables in Silver, then create Gold views that join them.

```sql
-- Fact table (Iceberg, partitioned)
CREATE TABLE nessie.silver.fact_sales (
  sale_id BIGINT,
  product_id INT,
  customer_id INT,
  store_id INT,
  sale_date DATE,
  quantity INT,
  amount DOUBLE
) PARTITION BY (MONTH(sale_date));

-- Dimension tables (Iceberg, small, no partitioning needed)
CREATE TABLE nessie.silver.dim_products (
  product_id INT, name VARCHAR, category VARCHAR, price DOUBLE
);

CREATE TABLE nessie.silver.dim_customers (
  customer_id INT, name VARCHAR, email VARCHAR, region VARCHAR
);

-- Gold star-schema view
CREATE VIEW analytics.gold.sales_analysis AS
SELECT
  f.sale_date,
  p.category AS product_category,
  c.region AS customer_region,
  SUM(f.quantity) AS total_quantity,
  SUM(f.amount) AS total_revenue
FROM nessie.silver.fact_sales f
JOIN nessie.silver.dim_products p ON f.product_id = p.product_id
JOIN nessie.silver.dim_customers c ON f.customer_id = c.customer_id
GROUP BY f.sale_date, p.category, c.region;
```

### Slowly Changing Dimensions (SCD)

For Type 2 SCD (maintain history), use MERGE with effective dating:

```sql
MERGE INTO nessie.silver.dim_customers AS target
USING staging.customer_updates AS source
ON target.customer_id = source.customer_id AND target.is_current = true
WHEN MATCHED AND (target.name <> source.name OR target.region <> source.region) THEN
  UPDATE SET is_current = false, effective_end = CURRENT_DATE
WHEN NOT MATCHED THEN
  INSERT (customer_id, name, email, region, effective_start, effective_end, is_current)
  VALUES (source.customer_id, source.name, source.email, source.region, CURRENT_DATE, NULL, true);
```

---

## Semantic Layer Design

### Organizing Spaces and Folders

```
Analytics (Space)
├── gold/
│   ├── sales_analysis (View)
│   ├── customer_360 (View)
│   └── monthly_revenue (View)
├── silver/
│   ├── clean_transactions (View)
│   └── clean_customers (View)
└── reports/
    ├── executive_dashboard (View)
    └── ops_metrics (View)
```

**Guidelines:**
- Use **Spaces** for organizational boundaries (per team or domain)
- Use **Folders** within Spaces for the medallion layers
- Use **Wiki** on views to document business definitions
- Use **Tags** for classification (e.g., `pii`, `certified`, `draft`)

```sql
-- Create space and folder structure
-- (typically done via UI, CLI, or REST API)

-- Document a Gold view
-- Use Wiki API or CLI: dremio wiki set <view-id> --file docs/monthly_revenue.md

-- Tag a view
-- Use Tags API or CLI: dremio tag set <view-id> --tags "certified,finance,gold"
```

---

## Maintenance Schedule Recommendations

| Task | Frequency | SQL |
|---|---|---|
| OPTIMIZE (compact files) | Daily or after large loads | `OPTIMIZE TABLE catalog.schema.table` |
| VACUUM (expire snapshots) | Weekly | `VACUUM TABLE ... EXPIRE SNAPSHOTS OLDER_THAN '...'` |
| Refresh Silver tables | Per pipeline schedule | `MERGE INTO` or `INSERT INTO ... SELECT` |
| Reflection refresh | Automatic (Dremio handles) | Monitor via `sys.reflections` |

---

## Documentation Reference

| Topic | URL |
|---|---|
| CREATE TABLE (partitioning, sorting) | https://docs.dremio.com/current/reference/sql/commands/create-table |
| CREATE VIEW | https://docs.dremio.com/current/reference/sql/commands/create-view |
| Reflections | https://docs.dremio.com/current/reference/sql/commands/acceleration |
| MERGE (upserts, SCD) | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/apache-iceberg-merge/ |
| OPTIMIZE | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/optimize-table/ |
| VACUUM | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/vacuum-table/ |
| COPY INTO | https://docs.dremio.com/current/reference/sql/commands/apache-iceberg-tables/copy-into-table/ |
| ALTER TABLE (schema evolution) | https://docs.dremio.com/current/reference/sql/commands/alter-table/ |

---

## Tips

- **Default to views for Gold.** They're cheap, always up-to-date, and reflections handle performance.
- **Materialize Silver only when needed.** Simple type casts and renames work great as views.
- **Design reflections around queries, not tables.** Look at what your dashboards and analysts actually query.
- **Use MERGE for incremental pipelines.** It handles inserts, updates, and deletes in one atomic statement.
- **Partition by how data is queried, not how it arrives.** If users filter by month, partition by month — even if data arrives daily.
- **Document everything.** Dremio Wiki and Tags are designed for this. A well-documented Gold layer is far more valuable than a fast one.
