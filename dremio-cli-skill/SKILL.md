---
name: Dremio CLI
description: Enables an AI agent to install, configure, and use the dremio-cli Python tool to manage Dremio Software and Cloud from the command line.
---

# Dremio CLI Skill

This skill equips you to use the `dremio-cli` — a Python CLI tool that provides 100% API coverage for both **Dremio Software** (self-hosted) and **Dremio Cloud**. Use it to manage catalogs, execute SQL, manage sources/views/jobs, control access, and perform GitOps workflows via Dremio-as-Code.

- **GitHub**: https://github.com/developer-advocacy-dremio/dremio-python-cli
- **Full Docs**: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/README.md

---

## Installation

Requires **Python 3.8+**.

```bash
# Standard install
pip install dremio-cli

# Or use pipx for an isolated environment
pipx install dremio-cli

# Verify
dremio --version
```

---

## Credential Configuration

The CLI supports **profile-based** configuration and **environment variables**. Ask the user which Dremio edition they use (Software or Cloud) and guide them through setup.

### Option 1: Profiles (Recommended)

Profiles are stored locally and can be switched between. A user can have multiple profiles for different environments.

#### Dremio Software Profile

```bash
dremio profile create --name myprofile --type software \
  --base-url https://dremio.company.com \
  --username admin --password secret
```

#### Dremio Cloud Profile

```bash
dremio profile create --name cloud-prod --type cloud \
  --base-url https://api.dremio.cloud \
  --project-id <project-id> \
  --token <pat-token>
```

> For the EU control plane, use `--base-url https://api.eu.dremio.cloud`.

#### Switching Profiles

```bash
# Use a specific profile for a single command
dremio --profile cloud-prod catalog list

# Set a default profile
dremio profile set-default myprofile
```

### Option 2: Environment Variables

The user can also set credentials via environment variables or a `.env` file:

```bash
export DREMIO_BASE_URL=https://dremio.company.com
export DREMIO_USERNAME=admin
export DREMIO_PASSWORD=secret
```

> **Agent prompt:** "To use the Dremio CLI, I'll need to configure credentials. Are you using **Dremio Software** or **Dremio Cloud**? Would you like to create a named profile or set environment variables?"

---

## Output Formats

The CLI supports three output formats. Use `--output json` when you need to parse results programmatically.

```bash
# Table (default, human-readable)
dremio catalog list

# JSON (best for scripting and parsing)
dremio --output json catalog list

# YAML
dremio --output yaml catalog list
```

Use `--verbose` for debugging.

---

## Command Reference

### Catalog Operations

```bash
dremio catalog list                    # List all catalog items
dremio catalog get <id>                # Get item details by ID
dremio catalog get-by-path <path>      # Get item details by path
```

### SQL Operations

```bash
dremio sql execute "<query>"           # Execute a SQL query
dremio sql explain "<query>"           # Show execution plan
dremio sql validate "<query>"          # Validate SQL syntax
```

SQL execution is **asynchronous** on Dremio Software — the CLI handles polling automatically and returns results.

Use `--async` for long-running queries to return the job ID immediately without waiting.

### Source Management

```bash
dremio source list                     # List all sources
dremio source create --name MyDB \
  --type POSTGRES --config-file db.json  # Create a source from config
dremio source refresh <id>             # Refresh source metadata
```

### View Management

```bash
dremio view list                       # List views
dremio view create --path "Analytics.sales_summary" \
  --sql "SELECT date, SUM(amount) FROM sales GROUP BY date"
dremio view update <id>                # Update a view
```

### Job Management

```bash
dremio job list                        # List jobs
dremio job list --max-results 10       # List recent jobs (limited)
dremio job get <job-id>                # Get job details
dremio job results <job-id>            # Get job results
dremio job cancel <job-id>             # Cancel a running job
dremio job profile <job-id> \
  --download profile.zip               # Download job profile
```

### Space & Folder Management

```bash
dremio space create --name Analytics   # Create a space
dremio folder create --path <path>     # Create a folder
```

### Access Control (Grants, Users, Roles)

```bash
dremio grant list <id>                 # List grants on an object
dremio grant add <id> \
  --grantee-type ROLE \
  --grantee-id analyst \
  --privileges SELECT                  # Grant access
dremio user list                       # List users
dremio role list                       # List roles
```

### Tags & Wiki

```bash
dremio wiki set <id> --file README.md  # Set wiki content from file
dremio tag set <id> --tags "production,sensitive,pii"  # Set tags
```

---

## Platform Support Matrix

| Feature | Software | Cloud |
|---|---|---|
| Catalog Operations | ✅ | ✅ |
| SQL Execution | ✅ | ⚠️ Limited |
| Job Management | ✅ | ✅ |
| View Management | ✅ | ✅ |
| Source Management | ✅ | ✅ |
| Space/Folder Mgmt | ✅ | ✅ |
| Tags & Wiki | ✅ | ✅ |
| Grant Management | ✅ | ✅ |
| User Management | ✅ | ⚠️ Via Console |
| Role Management | ✅ | ⚠️ Via Console |
| Table Operations | ✅ | ✅ |

---

## Dremio-as-Code (GitOps)

The CLI includes a `sync` feature for managing Dremio catalog objects (Spaces, Folders, Views) as local files — enabling GitOps workflows.

### Setup

Create a `dremio.yaml` in your project root:

```yaml
version: "1.0"
scope:
  path: "dremio-catalog.finance"   # Dremio path to sync
  type: "SPACE"                    # SPACE or ICEBERGCATALOG
ignore:
  - "*.tmp"
```

### Pull/Push Workflow

```bash
# Pull current state from Dremio into local files
dremio sync pull

# Edit views locally (SQL + YAML pairs)
# Then push changes back to Dremio
dremio sync push
```

Pulling creates a directory structure mirroring Dremio with `.sql` and `.yaml` file pairs for each view.

### View Definition Format

Each view is defined by a YAML file that can include SQL, tags, wiki, access control, governance policies, and reflections:

```yaml
name: revenue_report
type: VIRTUAL_DATASET
path: ["dremio-catalog", "finance", "reports", "revenue_report"]
sql: |
  SELECT region, sum(amount) as total 
  FROM "dremio-catalog".finance.stg_sales 
  GROUP BY region
dependencies:
  - "stg_sales"
tags: ["finance", "official"]
description: "docs/revenue_report.md"

access_control:
  roles:
    - name: "finance_managers"
      privileges: ["SELECT"]
  users:
    - name: "auditor@example.com"
      privileges: ["SELECT", "ALTER"]
```

> **Note:** Governance policies (RBAC, row access, masking) and reflections are NOT automatically pulled — they must be manually defined in YAML. Full DAC docs: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/dac.md

---

## Common Workflows

### Data Pipeline Setup

```bash
# 1. Create a source
dremio source create --name MyDB --type POSTGRES --config-file db.json

# 2. Create a space
dremio space create --name Analytics

# 3. Create a view
dremio view create --path "Analytics.sales_summary" \
  --sql "SELECT date, SUM(amount) FROM sales GROUP BY date"

# 4. Grant access
dremio grant add <view-id> --grantee-type ROLE \
  --grantee-id analyst --privileges SELECT
```

### Monitoring Jobs

```bash
dremio job list --max-results 10
dremio job get <job-id>
dremio job profile <job-id> --download profile.zip
```

### Documenting Datasets

```bash
dremio wiki set <id> --file README.md
dremio tag set <id> --tags "production,sensitive,pii"
```

---

## Agent Workflow

When the user asks you to manage Dremio using the CLI:

1. **Check installation** — Run `dremio --version`. If not installed, install with `pip install dremio-cli`.
2. **Check configuration** — Run `dremio profile list` to see existing profiles. If none exist, guide the user to create one (Software vs Cloud).
3. **Execute commands** — Use the command reference above. Prefer `--output json` when you need to parse results.
4. **Look up advanced docs** — If you need details on a specific command group, read the relevant doc page from the repo:
   - Catalog: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/catalog.md
   - SQL: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/sql.md
   - Sources: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/sources.md
   - Views: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/views.md
   - Jobs: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/jobs.md
   - Spaces & Folders: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/spaces.md
   - Grants: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/grants.md
   - Tags & Wiki: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/wiki.md
   - Tables: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/tables.md
   - Users & Roles: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/users.md
   - DAC (GitOps): https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/dac.md
   - DAC Sources: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/dac_sources.md
   - DAC Tables: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/dac_tables.md
   - DAC Governance: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/dac_governance.md
   - DAC Reflections: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/dac_reflections.md
   - DAC Validations: https://github.com/developer-advocacy-dremio/dremio-python-cli/blob/main/dremio-cli/docs/dac_validations.md

---

## Tips

- Use `--output json` when you need to extract IDs or specific fields from command output.
- Use `--verbose` to debug connectivity or authentication issues.
- Use `--async` for SQL execution when you only need to submit a query and get the job ID back.
- Store SQL queries in files and reference them rather than inlining long queries.
- Profiles are stored in the user's home directory — multiple profiles can coexist for different environments (dev, staging, prod).
