---
name: Basic Dremio REST API
description: Enables an AI agent to authenticate with and make curl requests to the Dremio REST API for both Dremio Software and Dremio Cloud.
---

# Basic Dremio REST API Skill

This skill equips you to interact with Dremio's REST API via `curl`. It covers authentication, environment variable configuration, and provides direct links to the official API reference so you can look up exact request/response formats.

---

## Environment Variables

Before making any API calls, **ask the user** to provide the following environment variables. Explain what each one is for, and confirm whether they are using **Dremio Software** (self-hosted) or **Dremio Cloud**.

### Dremio Software (Self-Hosted)

| Variable | Description |
|---|---|
| `DREMIO_URL` | Base URL of the Dremio instance (e.g. `http://localhost:9047` or `https://dremio.example.com`) |
| `DREMIO_PAT` | A Personal Access Token (PAT) created in the Dremio UI or via the API |
| `DREMIO_USERNAME` | *(Optional — only if using username/password auth instead of a PAT)* |
| `DREMIO_PASSWORD` | *(Optional — only if using username/password auth instead of a PAT)* |

### Dremio Cloud

| Variable | Description |
|---|---|
| `DREMIO_CLOUD_PROJECT_ID` | The Project ID (UUID) from the Dremio Cloud console |
| `DREMIO_PAT` | A Personal Access Token (PAT) created in the Dremio Cloud UI |
| `DREMIO_CLOUD_REGION` | *(Optional)* Set to `EU` if using the EU control plane. Defaults to US. |

> **Prompt the user:** "To interact with Dremio's REST API, I'll need some credentials set as environment variables. Are you using **Dremio Software** (self-hosted) or **Dremio Cloud**? And can you provide a Personal Access Token (PAT)?"

---

## Authentication

### Dremio Software — Using a PAT (Recommended)

All API v3 requests use `Bearer` token authentication:

```bash
curl -X GET "${DREMIO_URL}/api/v3/catalog" \
  --header "Authorization: Bearer ${DREMIO_PAT}" \
  --header "Content-Type: application/json"
```

### Dremio Software — Using Username/Password (Fallback)

If the user cannot provide a PAT, you can generate a short-lived authentication token (valid for 30 hours) via the v2 login endpoint:

```bash
# Step 1: Obtain a token
TOKEN=$(curl -s -X POST "${DREMIO_URL}/apiv2/login" \
  --header "Content-Type: application/json" \
  --data-raw "{\"userName\": \"${DREMIO_USERNAME}\", \"password\": \"${DREMIO_PASSWORD}\"}" \
  | jq -r '.token')

# Step 2: Use the token (prepend _dremio prefix)
curl -X GET "${DREMIO_URL}/api/v3/catalog" \
  --header "Authorization: _dremio${TOKEN}" \
  --header "Content-Type: application/json"
```

> **Note:** The `_dremio` prefix is required when using authentication tokens obtained from the login endpoint. PATs use the standard `Bearer` prefix instead.

### Dremio Cloud — Using a PAT

Dremio Cloud uses `https://api.dremio.cloud` (US) or `https://api.eu.dremio.cloud` (EU) as the base URL, and API version `v0`. Most endpoints require the Project ID in the path:

```bash
# Determine base URL
if [ "${DREMIO_CLOUD_REGION}" = "EU" ]; then
  BASE_URL="https://api.eu.dremio.cloud"
else
  BASE_URL="https://api.dremio.cloud"
fi

# Example: List catalog
curl -X GET "${BASE_URL}/v0/projects/${DREMIO_CLOUD_PROJECT_ID}/catalog" \
  --header "Authorization: Bearer ${DREMIO_PAT}" \
  --header "Content-Type: application/json"
```

---

## Base URLs and API Versions

| Edition | Base URL | API Version |
|---|---|---|
| **Dremio Software** | `{DREMIO_URL}/api/v3` | v3 |
| **Dremio Cloud (US)** | `https://api.dremio.cloud/v0` | v0 |
| **Dremio Cloud (EU)** | `https://api.eu.dremio.cloud/v0` | v0 |

> For Dremio Cloud, most resource endpoints are scoped under `/v0/projects/{projectId}/`.

---

## Common API Operations (Quick Reference)

### Submit a SQL Query

```bash
# Dremio Software
curl -X POST "${DREMIO_URL}/api/v3/sql" \
  --header "Authorization: Bearer ${DREMIO_PAT}" \
  --header "Content-Type: application/json" \
  --data '{"sql": "SELECT * FROM my_table LIMIT 10"}'
```

The response returns a `jobId` — use it with the Job API to poll status and retrieve results.

### Get Job Status

```bash
curl -X GET "${DREMIO_URL}/api/v3/job/${JOB_ID}" \
  --header "Authorization: Bearer ${DREMIO_PAT}" \
  --header "Content-Type: application/json"
```

### Get Job Results

```bash
curl -X GET "${DREMIO_URL}/api/v3/job/${JOB_ID}/results" \
  --header "Authorization: Bearer ${DREMIO_PAT}" \
  --header "Content-Type: application/json"
```

### List Catalog

```bash
curl -X GET "${DREMIO_URL}/api/v3/catalog" \
  --header "Authorization: Bearer ${DREMIO_PAT}" \
  --header "Content-Type: application/json"
```

---

## API Reference Links

When you need the exact request/response schema for a specific operation, **read the relevant documentation page** using your URL reading capabilities. Here are the key sections:

### Dremio Software API Reference

| Resource | Documentation URL |
|---|---|
| **API Overview & Auth** | https://docs.dremio.com/current/reference/api/ |
| **Catalog** | https://docs.dremio.com/current/reference/api/catalog/ |
| **SQL (Submit Queries)** | https://docs.dremio.com/current/reference/api/sql/ |
| **Job** | https://docs.dremio.com/current/reference/api/job/ |
| **Reflections** | https://docs.dremio.com/current/reference/api/reflections/ |
| **Source** | https://docs.dremio.com/current/reference/api/source/ |
| **User** | https://docs.dremio.com/current/reference/api/user/ |
| **Role** | https://docs.dremio.com/current/reference/api/role/ |
| **Votes (Wiki/Labels)** | https://docs.dremio.com/current/reference/api/votes/ |
| **Personal Access Token** | https://docs.dremio.com/current/reference/api/personal-access-token/ |
| **OAuth Token** | https://docs.dremio.com/current/reference/api/oauth-token/ |

### Dremio Cloud API Reference

| Resource | Documentation URL |
|---|---|
| **API Overview & Auth** | https://docs.dremio.com/cloud/reference/api/ |
| **Catalog** | https://docs.dremio.com/cloud/reference/api/catalog/ |
| **SQL (Submit Queries)** | https://docs.dremio.com/cloud/reference/api/sql/ |
| **Job** | https://docs.dremio.com/cloud/reference/api/job/ |
| **Reflections** | https://docs.dremio.com/cloud/reference/api/reflections/ |
| **Source** | https://docs.dremio.com/cloud/reference/api/source/ |
| **Project** | https://docs.dremio.com/cloud/reference/api/project/ |
| **Cloud Roles (RBAC)** | https://docs.dremio.com/cloud/reference/api/role/ |
| **Personal Access Token** | https://docs.dremio.com/cloud/reference/api/personal-access-tokens/ |

---

## Agent Workflow

When the user asks you to interact with Dremio, follow this sequence:

1. **Determine edition** — Ask if they are using Dremio Software or Dremio Cloud.
2. **Collect credentials** — Request the appropriate environment variables (see table above). Guide the user to set them: `export DREMIO_PAT="..."` etc.
3. **Verify connectivity** — Run a simple catalog listing curl to confirm authentication works.
4. **Look up API docs** — If you need the exact schema for an endpoint, read the relevant documentation URL from the tables above using your URL-reading tools.
5. **Execute requests** — Build and run curl commands using the environment variables.
6. **Parse responses** — Use `jq` to extract fields from JSON responses when needed.

---

## Tips

- Always use `jq` for parsing JSON responses (e.g., extracting `jobId` from SQL submission responses).
- For Dremio Software, SQL queries are **asynchronous**: submit → poll job status → fetch results.
- Dremio Cloud uses API version `v0`, not `v3`. Don't mix them up.
- PATs are the recommended auth method for both editions. Username/password login is a fallback only for Dremio Software.
- When a user provides a PAT, use `Bearer ${DREMIO_PAT}` as the auth header value. When using a login token on Software, use `_dremio${TOKEN}` (no space, no "Bearer" prefix).
