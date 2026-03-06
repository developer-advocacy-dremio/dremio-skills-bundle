---
name: Dremio Python Libraries
description: Enables an AI agent to use dremio-simple-query (lightweight SQL/Arrow Flight) and DremioFrame (full dataframe builder with ingestion, modeling, and admin) to interact with Dremio from Python.
---

# Dremio Python Skill

This skill covers two Python libraries for working with Dremio. **Choose the right tool for the job:**

| Need | Use |
|---|---|
| Run SQL and get results as Arrow, Pandas, Polars, or DuckDB | **dremio-simple-query** |
| Data ingestion, CRUD, Iceberg management, admin, modeling, charting, orchestration | **DremioFrame** |

---

# Part 1: dremio-simple-query (Lightweight SQL)

A lightweight library that uses **Apache Arrow Flight** for high-performance SQL queries against Dremio.

- **GitHub**: https://github.com/developer-advocacy-dremio/dremio_simple_query
- **Full Docs**: https://github.com/developer-advocacy-dremio/dremio_simple_query/blob/main/docs/dremio_simple_query_docs.md

## Installation

```bash
pip install dremio-simple-query
```

## Credential Configuration

### Option 1: Profiles (Recommended)

Both `dremio-simple-query` and DremioFrame share the same profile file at `~/.dremio/profiles.yaml`. Create it with the following structure:

```yaml
profiles:
  # Dremio Cloud with PAT
  my_cloud:
    type: cloud
    base_url: https://api.dremio.cloud
    auth:
      type: pat
      token: MY_PAT_TOKEN

  # Software with PAT
  my_software_pat:
    type: software
    base_url: https://dremio.company.com
    auth:
      type: pat
      token: MY_PAT_TOKEN

  # Software with Username/Password
  my_software_basic:
    type: software
    base_url: https://dremio.company.com
    auth:
      type: username_password
      username: my_user
      password: my_password

  # Software with OAuth Client Credentials
  my_software_oauth:
    type: software
    base_url: https://dremio.company.com
    auth:
      type: oauth
      client_id: MY_CLIENT_ID
      client_secret: MY_CLIENT_SECRET
```

Then connect using a profile name:

```python
from dremio_simple_query.connectv2 import DremioConnection

dremio = DremioConnection(profile="my_cloud")
```

### Option 2: Environment Variables / Direct Parameters

```python
from dremio_simple_query.connectv2 import DremioConnection
from os import getenv
from dotenv import load_dotenv

load_dotenv()

# Dremio Cloud — PAT auth
dremio = DremioConnection(
    location=getenv("ARROW_ENDPOINT"),   # e.g. grpc+tls://data.dremio.cloud:443
    token=getenv("DREMIO_TOKEN"),
    project_id=getenv("DREMIO_PROJECT_ID")  # Optional for Cloud
)

# Dremio Software — Username/Password auth
dremio = DremioConnection(
    location="grpc+tls://dremio.company.com:32010",
    username="my_user",
    password="my_password"
)
```

> **Agent prompt:** "I need to connect to Dremio from Python. Are you using **Dremio Cloud** or **Dremio Software**? Do you have a PAT (Personal Access Token), or should we use username/password? Do you already have a `~/.dremio/profiles.yaml` configured?"

## Query & Output Methods

Always use the **V2 client** (`dremio_simple_query.connectv2`).

```python
from dremio_simple_query.connectv2 import DremioConnection

dremio = DremioConnection(profile="my_profile")

# Arrow FlightStreamReader (raw, most performant)
stream = dremio.toArrow("SELECT * FROM my_table")
arrow_table = stream.read_all()        # Arrow Table
batch_reader = stream.to_reader()      # RecordBatchReader

# Pandas DataFrame
df = dremio.toPandas("SELECT * FROM my_table")

# Polars DataFrame
df = dremio.toPolars("SELECT * FROM my_table")

# DuckDB Relation
duck_rel = dremio.toDuckDB("SELECT * FROM my_table")
result = duck_rel.query("my_table", "SELECT * FROM my_table").fetchall()
```

### Querying Arrow results with DuckDB (zero-copy)

```python
import duckdb

stream = dremio.toArrow("SELECT * FROM my_table")
my_table = stream.read_all()

con = duckdb.connection()
results = con.execute("SELECT * FROM my_table").fetchall()
```

---

# Part 2: DremioFrame (Full-Featured Dataframe Builder)

A comprehensive Python library providing a **dataframe builder interface** for Dremio with CRUD, ingestion, admin, modeling, charting, orchestration, and AI features. Currently in **alpha**.

- **GitHub**: https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe

## Installation

```bash
pip install dremioframe

# With optional dependencies (e.g., for chart image export)
pip install "dremioframe[image_export]"
```

Optional dependency groups are documented at:
https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/dependencies.md

## Credential Configuration

### Option 1: Profiles (Recommended — shared with dremio-simple-query)

Create `~/.dremio/profiles.yaml` (same format as above), then:

```python
from dremioframe.client import DremioClient

# Uses the default profile
client = DremioClient()

# Or specify a profile
client = DremioClient(profile="my_profile")
```

> You can generate the profiles file using `dremio-cli` (from the dremio-cli-skill) or create it manually.

### Option 2: Environment Variables

Create a `.env` file in your project:

```bash
# Dremio Cloud
DREMIO_PAT=your_dremio_cloud_pat_here
DREMIO_PROJECT_ID=your_dremio_project_id_here
# DREMIO_URL=data.dremio.cloud  # Optional, defaults to data.dremio.cloud

# Dremio Software (v26+)
# DREMIO_SOFTWARE_PAT=your_software_pat_here
# DREMIO_SOFTWARE_HOST=dremio.example.com
# DREMIO_SOFTWARE_PORT=32010
# DREMIO_SOFTWARE_TLS=false

# Dremio Software (v25 / username-password)
# DREMIO_SOFTWARE_USER=your_username
# DREMIO_SOFTWARE_PASSWORD=your_password
```

### Option 3: Direct Parameters

```python
# Dremio Cloud (assumes env vars DREMIO_PAT and DREMIO_PROJECT_ID are set)
client = DremioClient()

# Dremio Software v26+
client = DremioClient(
    hostname="dremio.example.com",
    pat="your_pat_here",
    tls=True,
    mode="v26"
)

# Dremio Software v25
client = DremioClient(
    hostname="localhost",
    username="admin",
    password="password123",
    tls=False,
    mode="v25"
)
```

## Core Usage

### Querying Data (Dataframe Builder)

```python
from dremioframe.client import DremioClient

client = DremioClient()

# Fluent builder pattern
df = (client.table('finance.bronze.transactions')
      .select("transaction_id", "amount", "customer_id")
      .filter("amount > 1000")
      .limit(5)
      .collect())

# Raw SQL
df = client.query("SELECT * FROM finance.silver.customers")

# Aggregation
(client.table('finance.bronze.transactions')
 .group_by("customer_id")
 .agg(total_spent="SUM(amount)")
 .show())

# Joins
customers = client.table('finance.silver.customers')
(client.table('finance.bronze.transactions')
 .join(customers, on="transactions.customer_id = customers.customer_id")
 .show())

# Calculated columns
df.mutate(amount_with_tax="amount * 1.08").show()

# Iceberg Time Travel
df.at_snapshot("123456789").show()
```

### SQL Functions

```python
from dremioframe import F

(client.table("finance.silver.sales")
 .select(
     F.col("dept"),
     F.sum("amount").alias("total_sales"),
     F.rank().over(F.Window.order_by("amount")).alias("rank")
 )
 .show())
```

### Data Ingestion

```python
# API Ingestion
client.ingest_api(
    url="https://api.example.com/users",
    table_name="finance.bronze.users",
    mode="merge",
    pk="id"
)

# Insert from Pandas DataFrame
import pandas as pd
data = pd.DataFrame({"id": [1, 2], "name": ["A", "B"]})
client.table("finance.bronze.raw_data").insert(
    "finance.bronze.raw_data", data=data, batch_size=1000
)

# Merge (Upsert)
client.table("finance.silver.customers").merge(
    target_table="finance.silver.customers",
    on="customer_id",
    matched_update={"name": "source.name", "updated_at": "source.updated_at"},
    not_matched_insert={"customer_id": "source.customer_id", "name": "source.name"},
    data=data
)
```

### Export

```python
df.to_csv("transactions.csv")
df.to_parquet("transactions.parquet")
```

### Charting

```python
(client.table('finance.gold.sales_summary')
 .chart(kind="bar", x="category", y="total_sales", save_to="sales.png"))
```

### Admin & Catalog

```python
# List catalog
print(client.catalog.list_catalog())

# Reflections
client.admin.create_reflection(
    dataset_id="...", name="my_ref", type="RAW", display_fields=["col1"]
)

# Explain query
print(df.explain())
```

### Data Quality

```python
df.quality.expect_not_null("customer_id")
df.quality.expect_row_count("amount > 10000", 5, "ge")
```

---

## Documentation Reference

When you need more detail on a specific feature, **read the relevant doc page** from the repos below.

### dremio-simple-query Docs

| Topic | URL |
|---|---|
| Full Documentation | https://github.com/developer-advocacy-dremio/dremio_simple_query/blob/main/docs/dremio_simple_query_docs.md |

### DremioFrame Docs — Getting Started

| Topic | URL |
|---|---|
| Configuration | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/configuration.md |
| Connection Guide | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/connection.md |
| Optional Dependencies | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/dependencies.md |
| S3 Integration | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/s3_integration.md |
| ETL Pipeline Tutorial | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/tutorial_etl.md |
| Cookbook / Recipes | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/cookbook.md |
| Troubleshooting | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/getting_started/troubleshooting.md |
| API Compatibility | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/api_compatibility.md |

### DremioFrame Docs — Data Engineering

| Topic | URL |
|---|---|
| Dataframe Builder API | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/builder.md |
| Querying Data | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/querying.md |
| Joins & Transformations | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/joins.md |
| Aggregation | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/aggregation.md |
| Sorting & Filtering | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/sorting.md |
| Creating Tables | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/creating_tables.md |
| API Ingestion | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/ingestion.md |
| Ingestion Patterns | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/ingestion_patterns.md |
| Database Ingestion | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/database_ingestion.md |
| File System Ingestion | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/file_system_ingestion.md |
| File Upload | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/file_upload.md |
| dlt Integration | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/dlt_integration.md |
| Exporting Data | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/export.md |
| Export Formats | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/export_formats.md |
| Caching | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/caching.md |
| Pydantic Integration | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/pydantic_integration.md |
| Iceberg Tables | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/iceberg.md |
| Iceberg Lakehouse Mgmt | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/guide_iceberg_management.md |
| Schema Evolution | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/schema_evolution.md |
| Incremental Processing | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/incremental_processing.md |
| Query Templates | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/query_templates.md |
| SQL Linting | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_engineering/sql_linting.md |

### DremioFrame Docs — Analysis & Visualization

| Topic | URL |
|---|---|
| Charting & Plotting | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/analysis/charting.md |
| Interactive Plotting | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/analysis/plotting.md |
| Query Profiling | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/analysis/profiling.md |

### DremioFrame Docs — Data Modeling

| Topic | URL |
|---|---|
| Medallion Architecture | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/modeling/medallion.md |
| Dimensional Modeling | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/modeling/dimensional.md |
| Slowly Changing Dimensions | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/modeling/scd.md |
| Semantic Views | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/modeling/views.md |
| Documenting Datasets | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/modeling/documentation.md |

### DremioFrame Docs — AI Capabilities

| Topic | URL |
|---|---|
| AI Overview | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/ai/overview.md |
| DremioAgent Class | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/ai/agent.md |
| MCP Server | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/ai/mcp_server.md |
| MCP Client Integration | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/ai/mcp_client.md |
| SQL Generation | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/ai/sql.md |
| Script Generation | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/ai/generation.md |
| API Call Generation | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/ai/api.md |

### DremioFrame Docs — Orchestration

| Topic | URL |
|---|---|
| Overview | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/orchestration/overview.md |
| Tasks & Sensors | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/orchestration/tasks.md |
| Scheduling | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/orchestration/scheduling.md |
| Iceberg Tasks | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/orchestration/iceberg.md |
| Reflection Tasks | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/orchestration/reflections.md |
| Deployment | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/orchestration/deployment.md |

### DremioFrame Docs — Admin & Governance

| Topic | URL |
|---|---|
| Administration | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/admin.md |
| Catalog Management | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/catalog.md |
| Reflections | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/reflections.md |
| UDFs | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/udf.md |
| Security | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/security.md |
| Masking & Row Access | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/masking_and_row_access.md |
| Privileges | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/privileges.md |
| Tags | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/tags.md |
| Lineage | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/admin_governance/lineage.md |

### DremioFrame Docs — Reference & Performance

| Topic | URL |
|---|---|
| Function Reference | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/reference/function_reference.md |
| SQL Functions Guide | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/reference/functions_guide.md |
| API Reference | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/reference/client.md |
| Async Client | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/reference/async_client.md |
| Performance Tuning | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/performance/tuning.md |
| Bulk Loading | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/performance/bulk_loading.md |
| Connection Pooling | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/performance/connection_pooling.md |
| Data Quality Framework | https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe/blob/main/docs/data_quality.md |

---

## Agent Workflow

1. **Assess the use case:**
   - **Simple SQL queries** → use `dremio-simple-query`
   - **Ingestion, CRUD, admin, modeling, charting** → use `dremioframe`
2. **Check credentials** — Ask the user if they have `~/.dremio/profiles.yaml` or env vars set up. Guide them through configuration.
3. **Install the library** — `pip install dremio-simple-query` or `pip install dremioframe`.
4. **Write Python code** using the patterns above.
5. **Look up docs** — If you need details on a specific feature, read the relevant documentation URL from the tables above using your URL-reading tools.

## Tips

- Both libraries share the same `~/.dremio/profiles.yaml` format — configure once, use everywhere.
- For `dremio-simple-query`, always use the **V2 client** (`dremio_simple_query.connectv2`), not the legacy V1.
- DremioFrame is in **alpha** — features are rapidly evolving. Check the docs for the latest API.
- For Dremio Cloud, the Arrow Flight endpoint is typically `grpc+tls://data.dremio.cloud:443`.
- For Dremio Software, the Arrow Flight port is typically `32010` (e.g., `grpc+tls://dremio.company.com:32010`).
- DremioFrame's `mode` parameter matters: use `"v26"` for Software v26+ with PAT, `"v25"` for older username/password auth.
