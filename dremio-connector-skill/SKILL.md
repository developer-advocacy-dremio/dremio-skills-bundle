---
name: Dremio Connector Guide
description: Guides an AI agent through adding data sources to Dremio by asking the user the right questions, recommending connection settings, and linking to the exact documentation for each connector.
---

# Dremio Connector Skill

This skill helps you **guide a user through adding a data source to Dremio** — whether they're using Dremio Software or Dremio Cloud. Your job is to:

1. Ask the right questions to identify what they're connecting to
2. Gather the credentials and connection details they'll need
3. Link them to the exact documentation page for their connector
4. Help troubleshoot connection failures

---

## Agent Workflow: Interactive Question Flow

When a user wants to add a source, follow this decision tree by asking questions one step at a time.

### Step 1: Determine Dremio Edition

> "Are you using **Dremio Software** (self-hosted) or **Dremio Cloud**?"

This matters because:
- Some connectors are only available in Software (e.g., HDFS, NAS, Elasticsearch, MongoDB, OpenSearch, Teradata)
- Cloud uses different URL patterns in the docs
- Cloud requires specific networking setup (VPC peering, PrivateLink, etc.)

### Step 2: Identify the Source Category

> "What type of data source do you want to connect? Choose a category:
> 1. **Lakehouse Catalog** — AWS Glue, Nessie, Iceberg REST, Snowflake Open Catalog, Unity Catalog, OneLake, Hive
> 2. **Object Storage** — Amazon S3, Azure Storage, Google Cloud Storage, HDFS, NAS
> 3. **Relational Database** — PostgreSQL, MySQL, SQL Server, Oracle, Snowflake, BigQuery, Redshift, etc.
> 4. **Other** — Dremio Cluster, Arctic, Elasticsearch, MongoDB, OpenSearch"

### Step 3: Ask Connector-Specific Questions

Once you know the source type, ask the user for the required connection parameters. Use the tables below to know what to ask. **Always read the connector's documentation page** (from the index below) for the full list of configuration options.

---

## What to Ask For Each Source Type

### Lakehouse Catalogs

| Source | Key Questions |
|---|---|
| **AWS Glue** | AWS region? AWS Access Key ID and Secret (or IAM role)? Default S3 bucket path? |
| **Nessie** | Nessie server URL? Auth type (Bearer token or None)? Storage backend (S3, ADLS, GCS)? Bucket/container name? Storage credentials? |
| **Iceberg REST Catalog** | Catalog URI? Auth type (Bearer, OAuth2, None)? Warehouse location? Storage credentials? |
| **Snowflake Open Catalog** | Account URL? Client ID and Secret? Warehouse? |
| **Unity Catalog** | Databricks workspace URL? Personal access token? Catalog name? Storage credentials for underlying S3/ADLS? |
| **Hive** | Hive Metastore host and port? HDFS or S3 storage backend? Kerberos auth? |
| **OneLake** | Azure Tenant ID? Client ID and Secret? Lakehouse name? |
| **Open Catalog (External)** | Catalog URI? OAuth client ID and Secret? Warehouse? |

### Object Storage

| Source | Key Questions |
|---|---|
| **Amazon S3** | Bucket name? AWS region? Auth method (Access Key, IAM Role, EC2 Metadata, None)? If Access Key: AWS Access Key ID and Secret? Root path (optional)? Enable Iceberg support? |
| **Azure Storage** | Container name? Account name? Auth method (Access Key, Shared Access Signature, Azure AD)? Credentials based on method? |
| **Google Cloud Storage** | Bucket name? Project ID? Service account JSON key file? Root path? |
| **HDFS** *(Software only)* | NameNode host and port? Impersonation enabled? Kerberos auth? Root path? |
| **NAS** *(Software only)* | Mount path on Dremio server? |

### Relational Databases

For all database connectors, always ask:

1. **Hostname** (or IP address)
2. **Port** (ask or use the default for that DB)
3. **Database name** (or schema/catalog depending on DB)
4. **Username and Password**
5. **TLS/SSL** — Is the connection encrypted?
6. **Network access** — Can Dremio reach this host? (firewall rules, VPC peering for Cloud)

| Source | Default Port | Extra Questions |
|---|---|---|
| **PostgreSQL** | 5432 | Schema name? |
| **MySQL** | 3306 | — |
| **Microsoft SQL Server** | 1433 | Windows auth or SQL auth? Instance name? |
| **Oracle** | 1521 | SID or Service Name? |
| **Snowflake** | 443 | Account identifier? Warehouse? Database? Schema? |
| **Google BigQuery** | — | Project ID? Service account JSON key? Dataset? |
| **Amazon Redshift** | 5439 | Cluster identifier or Serverless workgroup? |
| **IBM Db2** | 50000 | — |
| **SAP HANA** | 30015 | Instance number? |
| **Azure Synapse** | 1433 | Server name? Database? |
| **Apache Druid** | 8888 | Broker host? |
| **Teradata** *(Software only)* | 1025 | — |
| **Vertica** | 5433 | — |
| **MongoDB** *(Software only)* | 27017 | Authentication database? Replica set? |
| **Elasticsearch** *(Software only)* | 9200 | Cluster name? Use scripts? |
| **OpenSearch** *(Software only)* | 9200 | AWS region (for managed)? |
| **Azure Data Explorer** *(Software only)* | — | Cluster URL? Application ID? |

### Other Sources

| Source | Key Questions |
|---|---|
| **Dremio Cluster** | Dremio host URL? Auth token or username/password? |
| **Arctic** *(Cloud only)* | Arctic catalog name? (Created in Dremio Cloud UI) |

---

## Connector Documentation Index

When you need the exact configuration parameters, required fields, or troubleshooting steps for a specific connector, **read the relevant documentation page** using your URL-reading tools.

### Dremio Software — Source Connector Docs

#### Overview
| Topic | URL |
|---|---|
| Manage Sources (Overview) | https://docs.dremio.com/current/sonar/data-sources/ |

#### Open Catalog (Built-In)
| Topic | URL |
|---|---|
| Open Catalog | https://docs.dremio.com/current/data-sources/open-catalog/ |

#### Lakehouse Catalogs
| Source | URL |
|---|---|
| AWS Glue Data Catalog | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/aws-glue-catalog/ |
| Open Catalog (External) | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/open-catalog-external |
| Hive | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/hive/ |
| Iceberg REST Catalog | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/iceberg-rest-catalog/ |
| Nessie | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/nessie/ |
| Snowflake Open Catalog | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/snowflake-open/ |
| Unity Catalog | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/unity/ |
| Microsoft OneLake | https://docs.dremio.com/current/data-sources/lakehouse-catalogs/onelake |

#### Object Storage
| Source | URL |
|---|---|
| Amazon S3 | https://docs.dremio.com/current/data-sources/object/s3/ |
| Azure Storage | https://docs.dremio.com/current/data-sources/object/azure-storage/ |
| Google Cloud Storage | https://docs.dremio.com/current/data-sources/object/gcs/ |
| HDFS | https://docs.dremio.com/current/data-sources/object/hdfs/ |
| NAS | https://docs.dremio.com/current/data-sources/object/nas/ |

#### Databases
| Source | URL |
|---|---|
| Amazon OpenSearch | https://docs.dremio.com/current/data-sources/databases/opensearch |
| Amazon Redshift | https://docs.dremio.com/current/data-sources/databases/redshift |
| Apache Druid | https://docs.dremio.com/current/data-sources/databases/apache-druid |
| Dremio Cluster | https://docs.dremio.com/current/data-sources/databases/dremio |
| Google BigQuery | https://docs.dremio.com/current/data-sources/databases/google-bigquery |
| Elasticsearch | https://docs.dremio.com/current/data-sources/databases/elasticsearch |
| IBM Db2 | https://docs.dremio.com/current/data-sources/databases/ibm-db2 |
| Azure Data Explorer | https://docs.dremio.com/current/data-sources/databases/azure-data-explorer |
| Azure Synapse Analytics | https://docs.dremio.com/current/data-sources/databases/azure-synapse-analytics |
| Microsoft SQL Server | https://docs.dremio.com/current/data-sources/databases/sql-server |
| MongoDB | https://docs.dremio.com/current/data-sources/databases/mongo |
| MySQL | https://docs.dremio.com/current/data-sources/databases/mysql |
| Oracle | https://docs.dremio.com/current/data-sources/databases/oracle |
| PostgreSQL | https://docs.dremio.com/current/data-sources/databases/postgres |
| SAP HANA | https://docs.dremio.com/current/data-sources/databases/sap-hana |
| Snowflake | https://docs.dremio.com/current/data-sources/databases/snowflake |
| Teradata | https://docs.dremio.com/current/data-sources/databases/teradata |
| Vertica | https://docs.dremio.com/current/data-sources/databases/vertica |

#### Files & Other
| Topic | URL |
|---|---|
| Formatting Data to a Table | https://docs.dremio.com/current/developer/data-formats/table/ |
| Upload Files | https://docs.dremio.com/current/data-sources/file-upload |
| External Queries | https://docs.dremio.com/current/help-support/advanced-topics/external-queries/ |
| Runtime Filtering | https://docs.dremio.com/current/help-support/advanced-topics/runtime-filtering/ |

---

### Dremio Cloud — Source Connector Docs

#### Overview
| Topic | URL |
|---|---|
| Connecting to Your Data (Overview) | https://docs.dremio.com/cloud/sonar/data-sources/ |

#### Data-as-Code / Catalogs
| Source | URL |
|---|---|
| Arctic Catalog | https://docs.dremio.com/cloud/sonar/data-sources/arctic/ |

#### Metastores
| Source | URL |
|---|---|
| AWS Glue Catalog | https://docs.dremio.com/cloud/sonar/data-sources/metastores/aws-glue |
| Snowflake Open Catalog | https://docs.dremio.com/cloud/sonar/data-sources/metastores/snowflake-open |
| Unity Catalog | https://docs.dremio.com/cloud/sonar/data-sources/metastores/unity |

#### Object Storage
| Source | URL |
|---|---|
| Amazon S3 | https://docs.dremio.com/cloud/sonar/data-sources/amazon-s3 |
| Azure Storage | https://docs.dremio.com/cloud/sonar/data-sources/azure-storage |

#### Relational Databases (External Sources)
| Source | URL |
|---|---|
| Amazon Redshift | https://docs.dremio.com/cloud/sonar/data-sources/external/amazon-redshift |
| Apache Druid | https://docs.dremio.com/cloud/sonar/data-sources/external/apache-druid |
| Google BigQuery | https://docs.dremio.com/cloud/sonar/data-sources/external/google-bigquery |
| IBM Db2 | https://docs.dremio.com/cloud/sonar/data-sources/external/ibm-db2 |
| Azure Synapse Analytics | https://docs.dremio.com/cloud/sonar/data-sources/external/azure-synapse-analytics |
| Microsoft SQL Server | https://docs.dremio.com/cloud/sonar/data-sources/external/microsoft-sql-server |
| MySQL | https://docs.dremio.com/cloud/sonar/data-sources/external/mysql |
| Oracle | https://docs.dremio.com/cloud/sonar/data-sources/external/oracle |
| PostgreSQL | https://docs.dremio.com/cloud/sonar/data-sources/external/postgres |
| SAP HANA | https://docs.dremio.com/cloud/sonar/data-sources/external/sap-hana |
| Snowflake | https://docs.dremio.com/cloud/sonar/data-sources/external/snowflake |
| Vertica | https://docs.dremio.com/cloud/sonar/data-sources/external/vertica |

#### Other Cloud Sources
| Source | URL |
|---|---|
| Dremio Enterprise Cluster | https://docs.dremio.com/cloud/sonar/data-sources/dremio |

---

## Troubleshooting Guide

When a connection fails, walk the user through this checklist:

### 1. Network Connectivity

> "Can the Dremio instance reach the data source host?"

- **Dremio Software**: The Dremio server must be able to resolve and reach the hostname/IP on the specified port. Ask the user to test with `telnet <host> <port>` or `nc -zv <host> <port>` from the Dremio server.
- **Dremio Cloud**: The data source must be reachable from Dremio Cloud's infrastructure. This often requires:
  - Public internet access with IP allowlisting
  - VPC peering or AWS PrivateLink
  - Azure Private Endpoint
  - Check the [Cloud networking docs](https://docs.dremio.com/cloud/cloud-infrastructure/) for setup guidance.

### 2. Authentication Failures

> "What error message are you seeing?"

Common patterns:
- **"Authentication failed"** → Wrong username/password, expired PAT, or wrong auth method selected
- **"Access denied"** → The user account exists but lacks permission to the database/schema/bucket
- **"Certificate error" / "SSL handshake failed"** → TLS is required but not configured, or the certificate is self-signed
  - For self-signed certs on Software, the CA must be added to the Dremio server's Java truststore
- **AWS credential errors** → Check IAM role trust policy, Access Key validity, or EC2 instance profile configuration

### 3. Missing or Wrong Parameters

> "Let me verify the connection parameters step by step."

Walk through each parameter:
- **Host** — Is it a hostname, IP, or full URL? Some connectors want just the hostname, not `https://...`
- **Port** — Is it the default, or has it been customized?
- **Database/Schema/Catalog** — The terminology differs per source. Make sure the user is putting the right value in the right field.
- **SSL/TLS** — Does the source require encrypted connections? Is the Dremio server configured for it?

### 4. Source-Specific Issues

| Issue | Resolution |
|---|---|
| **S3 "Access Denied"** | Check IAM permissions: `s3:GetObject`, `s3:ListBucket`, `s3:GetBucketLocation`. For Iceberg, also need `s3:PutObject`, `s3:DeleteObject`. |
| **Azure "AuthorizationFailed"** | Verify storage account access keys, SAS token expiry, or Azure AD app registration. |
| **GCS "Forbidden"** | Verify service account has `storage.objects.get` and `storage.objects.list` permissions. |
| **Snowflake "Warehouse suspended"** | The Snowflake warehouse may need to be resumed, or auto-suspend is too aggressive. |
| **BigQuery "Not found"** | Verify the project ID and dataset name are correct. The service account needs `bigquery.datasets.get` at minimum. |
| **PostgreSQL/MySQL "Connection refused"** | Check `pg_hba.conf` (Postgres) or `bind-address` (MySQL) allows connections from the Dremio server's IP. |
| **Oracle "ORA-12541: TNS: no listener"** | Wrong host, port, or the Oracle listener service is not running. |
| **MongoDB "Authentication failed"** | Check the `authSource` database (usually `admin`). Ensure the user has `readAnyDatabase` role. |
| **Metadata refresh very slow** | Large catalogs with many tables can cause slow refreshes. Use metadata filters, limit schemas, or increase refresh intervals. |

### 5. Dremio Cloud Networking

For Cloud-specific connectivity issues, refer to:
- **AWS**: https://docs.dremio.com/cloud/cloud-infrastructure/
- **Azure**: https://docs.dremio.com/cloud/cloud-infrastructure/

Ask: "Is your data source publicly accessible, or is it in a private VPC/subnet? If private, have you set up VPC peering or PrivateLink?"

---

## Software-Only vs Cloud Connectors

Some connectors are **only available in Dremio Software**:

| Connector | Available In |
|---|---|
| HDFS | Software only |
| NAS | Software only |
| MongoDB | Software only |
| Elasticsearch | Software only |
| Amazon OpenSearch | Software only |
| Azure Data Explorer | Software only |
| Teradata | Software only |
| Hive Metastore | Software only |
| Nessie | Software only |

If a user on Dremio Cloud asks for one of these, let them know it's not available in Cloud and suggest alternatives (e.g., "For MongoDB, consider using a federated database approach or data replication to an S3-based Iceberg table").

---

## Adding Sources via the REST API

Sources can also be added programmatically via the Dremio REST API. See the Source API docs:

| Edition | URL |
|---|---|
| Software | https://docs.dremio.com/current/reference/api/source/ |
| Cloud | https://docs.dremio.com/cloud/reference/api/source/ |

Read the relevant API doc to get the JSON payload structure for creating a source via `POST /api/v3/source`.

---

## Tips

- Always **read the specific connector doc page** before giving configuration advice. Each connector has unique required and optional fields.
- For database sources, the most common issues are **network connectivity** and **authentication** — always check those first.
- Dremio Cloud requires data sources to be network-accessible from Cloud infrastructure — this is the #1 issue for Cloud users.
- For object storage, **IAM permissions** are the most common stumbling block. Read the connector doc for the exact permissions list.
- Case-insensitive file naming is a known behavior in Dremio — warn users if they have files with names differing only in case.
- For relational databases, Dremio requires collation equivalent to `LATIN1_GENERAL_BIN2` for consistent pushdown results.
