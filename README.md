# Dremio AI Agent Skills

A curated collection of markdown-based skill sets designed to supercharge AI coding agents (e.g., Google Antigravity, Claude Code, Cursor, Windsurf) when working with [Dremio](https://www.dremio.com/) — the unified lakehouse platform.

## What Are "Skills"?

Skills are structured folders of markdown files that provide AI agents with domain-specific knowledge and instructions. When an agent loads a skill, it gains contextual expertise — enabling it to write better code, follow best practices, and avoid common pitfalls specific to a technology or workflow.

Think of each skill as a **portable knowledge pack** that turns a general-purpose AI assistant into a Dremio specialist.

---

## Available Skills

### [`basic-dremio-skill`](basic-dremio-skill/SKILL.md) — REST API & curl

The foundational skill for interacting with Dremio's REST API directly via `curl`. Use this when you need raw API access without additional libraries.

- **Authentication** — PAT (recommended), OAuth, and username/password for both Software and Cloud
- **Environment variables** — `DREMIO_URL`, `DREMIO_PAT`, `DREMIO_CLOUD_PROJECT_ID`, etc.
- **Common curl patterns** — SQL submission, job polling, catalog listing
- **API reference index** — Direct links to every resource page on `docs.dremio.com`

---

### [`dremio-cli-skill`](dremio-cli-skill/SKILL.md) — Python CLI Tool

Teaches the agent to use the [`dremio-cli`](https://github.com/developer-advocacy-dremio/dremio-python-cli) Python package for managing Dremio from the command line.

- **Installation** — `pip install dremio-cli`
- **Profile management** — Create and switch between Software/Cloud profiles
- **Full command reference** — Catalog, SQL, source, view, job, space/folder, grants, users/roles, tags, wiki
- **Dremio-as-Code (GitOps)** — `dremio sync pull/push` for managing views, folders, and governance as local YAML files
- **Documentation links** — 15+ deep-link URLs to individual command docs

---

### [`dremio-python-skill`](dremio-python-skill/SKILL.md) — Python Libraries

Covers two Python libraries for different levels of complexity:

**[dremio-simple-query](https://github.com/developer-advocacy-dremio/dremio_simple_query)** — For simple SQL use cases via Apache Arrow Flight:
- Query Dremio and get results as Arrow, Pandas, Polars, or DuckDB
- Lightweight, high-performance, zero-copy data transfer

**[DremioFrame](https://github.com/developer-advocacy-dremio/dremio-cloud-dremioframe)** — For complex use cases:
- Fluent dataframe builder API (select, filter, join, aggregate)
- Data ingestion (API, database, file system, merge/upsert)
- Iceberg table management, charting, data quality, orchestration
- Admin, governance, AI capabilities

Both share `~/.dremio/profiles.yaml` for credential management. The skill includes **60+ documentation deep-links** organized by category.

---

### [`dremio-connector-skill`](dremio-connector-skill/SKILL.md) — Source Connector Guide

An interactive skill that guides users through adding data sources to Dremio and troubleshooting connection issues.

- **Interactive question flow** — Step-by-step decision tree: edition → category → connector-specific
- **Info-gathering tables** — What to ask for each of 30+ connector types (ports, auth, credentials)
- **Full documentation index** — 45+ direct URLs to connector docs for both Software and Cloud
- **Troubleshooting guide** — Network, authentication, parameter validation, source-specific fixes
- **Availability matrix** — Which connectors are Software-only vs available in Cloud

---

### [`dremio-sql-skill`](dremio-sql-skill/SKILL.md) — SQL Dialect Reference

Teaches the agent Dremio's SQL dialect — the unique syntax and features that differ from standard ANSI SQL.

- **Dialect differences** — Path quoting, versioned queries, reflection DDL, UDF syntax, column masking
- **Quick-reference SQL** — Examples for every command category (DDL, DML, COPY INTO, maintenance, RBAC, UDFs, PIPE)
- **SQL commands index** — 25+ commands with direct doc links
- **Iceberg DML index** — COPY INTO, INSERT, UPDATE, DELETE, MERGE, OPTIMIZE, VACUUM, ROLLBACK, TRUNCATE
- **SQL functions index** — 20 function categories (Aggregate, AI, Geospatial, Window, etc.)

---

### [`dremio-iceberg-skill`](dremio-iceberg-skill/SKILL.md) — Apache Iceberg Operations

A deep guide for working with Apache Iceberg tables in Dremio — covering the full lifecycle.

- **Table creation** — Partitioning strategies, sort orders, CTAS
- **Full DML** — INSERT, UPDATE, DELETE, MERGE (upsert), TRUNCATE
- **COPY INTO** — Bulk ingestion from S3/Azure/GCS (Parquet, CSV, JSON)
- **Schema & partition evolution** — Add/drop columns and change partitions without data rewrites
- **Time travel** — AT SNAPSHOT, AT TIMESTAMP, ROLLBACK
- **Versioned catalog operations** — AT BRANCH, AT TAG (Nessie/Arctic)
- **Table maintenance** — OPTIMIZE (compaction), VACUUM (snapshot expiry)

---

### [`dremio-lakehouse-modeling-skill`](dremio-lakehouse-modeling-skill/SKILL.md) — Data Modeling Best Practices

Opinionated guidance on how to model data in a Dremio lakehouse.

- **Medallion architecture** — Bronze (raw) → Silver (cleaned) → Gold (consumption) with Dremio-specific implementation patterns
- **Views vs tables** — Decision framework for when to materialize vs virtualize
- **Reflections strategy** — When and how to create raw vs aggregation reflections
- **Partitioning guidelines** — Size-based recommendations and anti-patterns
- **Dimensional modeling** — Star schema, SCD Type 2 with MERGE
- **Semantic layer design** — Organizing Spaces, Folders, Wiki, and Tags
- **Maintenance schedules** — OPTIMIZE, VACUUM, refresh cadences

---

## Repository Structure

```
dremio-skills-repo/
├── README.md
├── basic-dremio-skill/
│   └── SKILL.md
├── dremio-cli-skill/
│   └── SKILL.md
├── dremio-connector-skill/
│   └── SKILL.md
├── dremio-iceberg-skill/
│   └── SKILL.md
├── dremio-lakehouse-modeling-skill/
│   └── SKILL.md
├── dremio-python-skill/
│   └── SKILL.md
└── dremio-sql-skill/
    └── SKILL.md
```

## How to Use

### 1. Clone the Repository

```bash
git clone https://github.com/<your-org>/dremio-skills-repo.git
```

### 2. Add Skills to Your Project

Copy (or symlink) skill folders into your project's agent configuration directory:

```bash
# Add a single skill
cp -r dremio-skills-repo/dremio-sql-skill /path/to/your-project/.agent/skills/

# Or add all skills at once
cp -r dremio-skills-repo/*-skill /path/to/your-project/.agent/skills/
```

### 3. Let the Agent Discover It

Most AI coding agents automatically detect skill folders placed in their expected directory (e.g., `.agent/skills/`). Once discovered, the agent reads the `SKILL.md` and follows its instructions when the skill is relevant to the current task.

## Which Skill Should I Use?

| I want to... | Use this skill |
|---|---|
| Make raw API calls to Dremio with curl | **basic-dremio-skill** |
| Manage Dremio from the command line | **dremio-cli-skill** |
| Run SQL queries from Python and get DataFrames | **dremio-python-skill** |
| Ingest data, manage Iceberg tables, or do admin tasks from Python | **dremio-python-skill** |
| Add a new data source or troubleshoot connections | **dremio-connector-skill** |
| Write correct SQL for Dremio (dialect, functions, syntax) | **dremio-sql-skill** |
| Work with Apache Iceberg tables (DML, maintenance, time travel) | **dremio-iceberg-skill** |
| Design a lakehouse data model (medallion, reflections, partitioning) | **dremio-lakehouse-modeling-skill** |

## Contributing a New Skill

1. **Create a new folder** at the repository root with a descriptive, kebab-case name.
2. **Add a `SKILL.md`** with YAML frontmatter (`name`, `description`) and detailed markdown instructions.
3. **Optionally include** `scripts/`, `examples/`, or `resources/` subdirectories.
4. **Update this README** to index the new skill.
5. **Open a pull request** with a clear description.

## About Dremio

[Dremio](https://www.dremio.com/) is a unified lakehouse platform that enables high-performance analytics directly on data lake storage. It provides:

- **SQL query engine** powered by Apache Arrow
- **Data-as-Code** with Git-like semantics (Arctic / Nessie catalog)
- **Apache Iceberg** table management
- **Reflections** for automatic query acceleration
- **Unified access** across data lakes, databases, and cloud storage

## License

This repository is open source. See [LICENSE](LICENSE) for details.
