# Dremio AI Agent Skills

A curated collection of markdown-based skill sets designed to supercharge AI coding agents when working with [Dremio](https://www.dremio.com/) — the unified lakehouse platform.

## Table of Contents

- [What Are Skills?](#what-are-skills)
- [Available Skills](#available-skills)
  - [basic-dremio-skill](#basic-dremio-skill--rest-api--curl)
  - [dremio-cli-skill](#dremio-cli-skill--python-cli-tool)
  - [dremio-python-skill](#dremio-python-skill--python-libraries)
  - [dremio-connector-skill](#dremio-connector-skill--source-connector-guide)
  - [dremio-sql-skill](#dremio-sql-skill--sql-dialect-reference)
  - [dremio-iceberg-skill](#dremio-iceberg-skill--apache-iceberg-operations)
  - [dremio-lakehouse-modeling-skill](#dremio-lakehouse-modeling-skill--data-modeling-best-practices)
- [Which Skill Should I Use?](#which-skill-should-i-use)
- [Using These Skills with AI Tools](#using-these-skills-with-ai-tools)
  - [Google Antigravity](#google-antigravity)
  - [Claude Code](#claude-code)
  - [Claude CoWork](#claude-cowork)
  - [OpenAI Codex CLI & App](#openai-codex-cli--app)
  - [Gemini CLI](#gemini-cli)
  - [Cursor](#cursor)
  - [Windsurf](#windsurf)
  - [Zed](#zed)
- [Using Skills in Web-Based AI Chat Apps](#using-skills-in-web-based-ai-chat-apps)
- [Repository Structure](#repository-structure)
- [Contributing a New Skill](#contributing-a-new-skill)
- [About Dremio](#about-dremio)

---

## What Are Skills?

Skills are structured folders of markdown files that provide AI agents with domain-specific knowledge and instructions. When an agent loads a skill, it gains contextual expertise — enabling it to write better code, follow best practices, and avoid common pitfalls specific to a technology or workflow.

Each skill folder contains a `SKILL.md` file with:

- **YAML frontmatter** — `name` and `description` fields that help agents decide when the skill is relevant
- **Markdown body** — Detailed instructions, code examples, documentation links, and troubleshooting guidance

More complex skills may also include `scripts/`, `examples/`, or `resources/` subdirectories.

The `SKILL.md` format is an **open standard** supported across major AI coding tools — making these skills portable and tool-agnostic.

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

Both share `~/.dremio/profiles.yaml` for credential management. The skill includes **60+ documentation deep-links**.

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

- **Medallion architecture** — Bronze (raw) → Silver (cleaned) → Gold (consumption) with Dremio-specific implementation
- **Views vs tables** — Decision framework for when to materialize vs virtualize
- **Reflections strategy** — When and how to create raw vs aggregation reflections
- **Partitioning guidelines** — Size-based recommendations and anti-patterns
- **Dimensional modeling** — Star schema, SCD Type 2 with MERGE
- **Semantic layer design** — Organizing Spaces, Folders, Wiki, and Tags
- **Maintenance schedules** — OPTIMIZE, VACUUM, refresh cadences

---

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

---

## Using These Skills with AI Tools

The `SKILL.md` format is an open standard supported by major AI coding tools. Below are setup instructions for each tool, verified against current documentation and behavior.

### Google Antigravity

[Antigravity](https://antigravity.google) natively supports the `SKILL.md` format. It discovers skills via a directory hierarchy and lazily loads them only when relevant to the current task.

**Workspace-specific** (for a single project):
```bash
# Copy skills into your project
mkdir -p .agent/skills
cp -r dremio-skills-repo/dremio-sql-skill .agent/skills/
cp -r dremio-skills-repo/dremio-iceberg-skill .agent/skills/
```

**Global** (available across all projects):
```bash
mkdir -p ~/.gemini/antigravity/skills
cp -r dremio-skills-repo/*-skill ~/.gemini/antigravity/skills/
```

The agent automatically discovers skills by reading YAML frontmatter (`name`, `description`). When a skill is relevant, it reads the full `SKILL.md` body and follows its instructions.

---

### Claude Code

[Claude Code](https://docs.anthropic.com/en/docs/claude-code) supports skills via the `.claude/skills/` directory. Each skill is a folder containing a `SKILL.md` file that Claude loads on demand.

**Setup:**
```bash
mkdir -p .claude/skills
cp -r dremio-skills-repo/dremio-sql-skill .claude/skills/
cp -r dremio-skills-repo/dremio-python-skill .claude/skills/
```

**How it works:** Claude Code uses progressive disclosure — it reads skill names and descriptions first, then loads the full `SKILL.md` content only when the skill is relevant to the current task. This keeps context efficient.

You can also place project-wide instructions in a `CLAUDE.md` file at the project root to point Claude toward relevant skills or provide additional Dremio context.

---

### Claude CoWork

[Claude CoWork](https://claude.com/cowork) is Anthropic's autonomous desktop agent (available on macOS and Windows with Claude Max/Pro/Team/Enterprise plans). It can read local files and coordinate sub-agents for complex, multi-step tasks.

**Setup:** CoWork accesses files from designated local folders. Place skills in a folder that CoWork can access:

```bash
# Create a skills directory in your CoWork-accessible folder
mkdir -p ~/dremio-skills
cp -r dremio-skills-repo/*-skill ~/dremio-skills/
```

Then tell CoWork about the skills by referencing them in your prompt:

> "I have Dremio agent skills in `~/dremio-skills/`. Read the relevant SKILL.md files when I ask about Dremio. For SQL questions, read `dremio-sql-skill/SKILL.md`. For data source connections, read `dremio-connector-skill/SKILL.md`."

You can also create folder-specific instructions within CoWork to automatically load relevant skills. CoWork supports the same Agent Skills open standard used by Claude Code.

---

### OpenAI Codex CLI & App

[OpenAI Codex](https://github.com/openai/codex) (CLI and desktop app) supports the `AGENTS.md` standard for project-level instructions and the Agent Skills open standard for modular expertise.

**Option 1: AGENTS.md** — Create an `AGENTS.md` file at the root of your project that references the skills:

```markdown
# Dremio Agent Skills

This project uses Dremio. When working with Dremio, read the relevant
skill files from the `skills/` directory:

- `skills/dremio-sql-skill/SKILL.md` — Dremio SQL dialect reference
- `skills/dremio-python-skill/SKILL.md` — Python libraries for Dremio
- `skills/dremio-connector-skill/SKILL.md` — Adding data sources
- `skills/dremio-iceberg-skill/SKILL.md` — Iceberg table operations
```

**Option 2: Skills directory** — Codex App supports a skills directory:

```bash
mkdir -p skills
cp -r dremio-skills-repo/*-skill skills/
```

Codex reads `AGENTS.md` automatically and uses it to guide its behavior. The file closest to the working directory takes precedence.

---

### Gemini CLI

[Gemini CLI](https://github.com/google-gemini/gemini-cli) uses `GEMINI.md` files for project-specific context and also discovers agent skills from `AGENTS.md` files.

**Option 1: GEMINI.md** — Create a `GEMINI.md` file at your project root that imports or references the skills:

```markdown
# Dremio Context

When working with Dremio SQL, refer to `skills/dremio-sql-skill/SKILL.md`.
When adding a data source, refer to `skills/dremio-connector-skill/SKILL.md`.
When writing Python for Dremio, refer to `skills/dremio-python-skill/SKILL.md`.
```

**Option 2: AGENTS.md** — Gemini CLI also recognizes `AGENTS.md` files for agent-level instructions. Create one at the project root with the same references as shown in the Codex section.

**Option 3: Custom commands** — You can create reusable slash commands in `~/.gemini/commands/` or `.gemini/commands/` as `.toml` files that reference specific skills.

```bash
mkdir -p skills
cp -r dremio-skills-repo/*-skill skills/
```

---

### Cursor

[Cursor](https://www.cursor.com/) uses `.cursor/rules/` for project-level AI instructions (as `.mdc` files) and also supports the Agent Skills standard.

**Option 1: Rules** — Create a rule file that tells Cursor about the Dremio skills:

```bash
mkdir -p .cursor/rules
```

Create `.cursor/rules/dremio.mdc`:
```
---
description: Dremio SQL and data operations
globs: ["*.sql", "*.py"]
---

When writing SQL for Dremio, read `skills/dremio-sql-skill/SKILL.md` for dialect specifics.
When writing Python for Dremio, read `skills/dremio-python-skill/SKILL.md`.
When adding data sources, read `skills/dremio-connector-skill/SKILL.md`.
```

**Option 2: Skills directory** — Cursor supports installing skills from directories:

```bash
mkdir -p skills
cp -r dremio-skills-repo/*-skill skills/
```

**Option 3: AGENTS.md** — Cursor also reads `AGENTS.md` files at the project root.

---

### Windsurf

[Windsurf](https://windsurf.com/) (Cascade AI) supports both rules and skills as first-class features.

**Skills (recommended):**
```bash
# Workspace-specific
mkdir -p .windsurf/skills
cp -r dremio-skills-repo/dremio-sql-skill .windsurf/skills/
cp -r dremio-skills-repo/dremio-connector-skill .windsurf/skills/

# Global (available in all projects)
mkdir -p ~/.codeium/windsurf/skills
cp -r dremio-skills-repo/*-skill ~/.codeium/windsurf/skills/
```

Cascade discovers skills automatically by reading `SKILL.md` frontmatter and invokes them when relevant. You can also trigger a skill manually by typing `@skill-name` in the Cascade input.

**Rules (complementary):**
```bash
mkdir -p .windsurf/rules
```

Create `.windsurf/rules/dremio.md`:
```markdown
When this project involves Dremio, refer to the Dremio skills in
`.windsurf/skills/` for SQL syntax, connector setup, and Python library usage.
```

Windsurf also reads `AGENTS.md` files for context-aware instructions.

---

### Zed

[Zed](https://zed.dev/) recognizes multiple agent instruction formats. Its Agent Panel reads files at the project root to provide context-aware assistance.

**Setup:** Zed automatically detects these files at the project root:
- `AGENTS.md`
- `AGENT.md`
- `CLAUDE.md`
- `GEMINI.md`
- `.cursorrules`
- `.windsurfrules`
- `.clinerules`
- `.rules`
- `.github/copilot-instructions.md`

**Recommended approach:** Create an `AGENTS.md` at the project root that references the skills:

```markdown
# Dremio Skills

When working with Dremio, read the relevant skill file from `skills/`:

- SQL syntax: `skills/dremio-sql-skill/SKILL.md`
- Python libraries: `skills/dremio-python-skill/SKILL.md`
- Data sources: `skills/dremio-connector-skill/SKILL.md`
- Iceberg tables: `skills/dremio-iceberg-skill/SKILL.md`
- Data modeling: `skills/dremio-lakehouse-modeling-skill/SKILL.md`
```

```bash
mkdir -p skills
cp -r dremio-skills-repo/*-skill skills/
```

You can also add rules to Zed's **Rules Library** for default instructions that apply to all agent interactions, or create project-level `.rules` files.

---

## Using Skills in Web-Based AI Chat Apps

Web-based AI chat applications (such as ChatGPT, Claude.ai, Gemini, etc.) don't have direct file system access, so skills can't be auto-discovered. However, the markdown content of these skills is still extremely valuable as **context you can paste into conversations**.

### How to Use

1. **Copy & paste the SKILL.md content** — Open the relevant `SKILL.md` file and paste its full contents (or the relevant sections) into the chat as context at the start of your conversation.

2. **Use as a project knowledge file** — Many web-based AI tools now support uploading files or creating "projects" with reference documents. Upload the `SKILL.md` files as knowledge for your project:
   - **Claude.ai Projects** — Add `SKILL.md` files as project knowledge files. Claude will reference them automatically in project conversations.
   - **ChatGPT (Custom GPTs / Projects)** — Upload `SKILL.md` files as knowledge base documents for a custom GPT or project.
   - **Google AI Studio** — Attach `SKILL.md` files as context documents when using Gemini.

3. **Reference the documentation links** — Even without the full skill content, the documentation URL tables in each skill are useful. Paste the relevant URL table into a chat and ask the AI to read specific pages for detailed syntax.

### What Works Well

| Approach | Best For |
|---|---|
| Paste full SKILL.md into chat | One-off questions, quick sessions |
| Upload as project knowledge | Ongoing projects, repeated Dremio work |
| Paste just the doc URL tables | When you need the AI to look up specific syntax |
| Paste just the code examples | When you want the AI to follow specific patterns |

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

## Contributing a New Skill

1. **Create a new folder** at the repository root with a descriptive, kebab-case name.
2. **Add a `SKILL.md`** with YAML frontmatter (`name`, `description`) and detailed markdown instructions.
3. **Optionally include** `scripts/`, `examples/`, or `resources/` subdirectories.
4. **Update this README** to index the new skill and add it to the table of contents.
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
