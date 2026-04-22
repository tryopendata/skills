---
name: opendata-api
description: Query the OpenData API for data research and analysis. Use when fetching dataset rows, filtering, sorting, aggregating, inspecting columns, composing cross-dataset joins, exploring graph intelligence, or building data pipelines against OpenData endpoints.
---

# OpenData Query API

Query datasets stored as Parquet files through a REST API backed by DuckDB. The API returns JSON by default, with support for CSV, TSV, and XLSX exports.

**Base URL:** `https://api.tryopendata.ai` (production) or `http://localhost:8000` (local dev). Default to use production

## Authentication

All endpoints require authentication in production. Pass an API key via `Authorization: Bearer` header:

```bash
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/datasets/fred/cpi?limit=5"
```

Local dev (`localhost:8000`) does not require auth when running the standalone opendata server (`make quickstart`). The backend server (`make dev-all`) requires auth for write endpoints but allows unauthenticated reads.

## Quick Start

The dataset path IS the query endpoint. Just GET the dataset path with query params:

```bash
# Get the 5 most recent CPI values
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/datasets/fred/cpi?limit=5&sort=-date"
#                                           ^^^^^^^^^^^^^^^^
#                              This path returns data directly.

# Filter, sort, aggregate - all via query params on the same path
curl -H "Authorization: Bearer od_live_..." \
  'https://api.tryopendata.ai/v1/datasets/owid/gdp?filter[year][gte]=2020&sort=-gdp_per_capita&limit=10'
```

**Do NOT append `/query` to GET requests.** `GET /v1/datasets/fred/cpi/query` will fail with a `SUBDATASET_NOT_FOUND` error because the API interprets `query` as a subdataset name. The `POST /query` endpoint is a separate SQL interface (see below).

All data endpoints live under `/v1/datasets/`.

## Endpoints

| Method | Path                                               | Description                                                  |
| ------ | -------------------------------------------------- | ------------------------------------------------------------ |
| GET    | `/v1/datasets/{provider}/{dataset}`                | Query dataset rows (flat) or list subdatasets (hierarchical) |
| GET    | `/v1/datasets/{provider}/{dataset}/{subdataset}`   | Query subdataset rows                                        |
| GET    | `/v1/datasets/{provider}/{dataset}/columns`        | Column metadata and statistics                               |
| GET    | `/v1/datasets/{provider}/{dataset}/columns/{name}` | Single column detail with full value list                    |
| GET    | `/v1/datasets/{provider}/{dataset}/meta`           | Dataset metadata (schema, views, graph scores)               |
| GET    | `/v1/datasets/{provider}/{dataset}/views`          | List available views                                         |
| GET    | `/v1/datasets/{provider}/{dataset}/related`        | Related datasets (semantic + join + graph signals)           |
| GET    | `/v1/datasets/{provider}/{dataset}/joinable`       | List joinable datasets for composition                       |
| POST   | `/v1/datasets/{provider}/{dataset}/compose/preview`| Preview a cross-dataset join (LEFT JOIN)                     |
| GET    | `/v1/datasets/{provider}/{dataset}/compose/download.csv` | Download a composed join as CSV (auth required)        |
| POST   | `/v1/datasets/{provider}/{dataset}/query`          | Execute SQL query (authenticated)                            |
| GET    | `/v1/discover`                                     | Search datasets with enriched metadata for LLM agents        |
| POST   | `/v1/discover/batch`                               | Batch discover across multiple queries with deduplication    |
| POST   | `/v1/query`                                        | Cross-dataset SQL query (join multiple datasets)             |
| GET    | `/v1/search`                                       | Search datasets (includes graph intelligence scores)         |
| GET    | `/v1/categories/{slug}`                            | Browse datasets by category (supports graph sorting)         |
| GET    | `/v1/graph/communities/{community_id}/datasets`    | List datasets in a graph community by importance             |

## Subdatasets

Some datasets contain multiple tables (e.g., multi-sheet Excel workbooks, BLS series groups). For these:

- `GET /v1/datasets/{provider}/{dataset}` returns data for the default subdataset, or lists available subdatasets
- `GET /v1/datasets/{provider}/{dataset}/{subdataset}` queries a specific subdataset

If you get a `SUBDATASET_NOT_FOUND` error, the dataset likely has subdatasets. Check the error response's `suggestions` field - it includes a link to browse available subdatasets. Any unrecognized path segment after the dataset slug is interpreted as a subdataset name, which is why paths like `/query` or `/search` appended to a dataset path produce this error.

## Query Parameters

| Parameter         | Example                  | Description                                | Reference                                                   |
| ----------------- | ------------------------ | ------------------------------------------ | ----------------------------------------------------------- |
| `filter[col]`     | `filter[year]=2024`      | Filter rows by column value                | [filtering.md](references/filtering.md)                     |
| `filter[col][op]` | `filter[year][gte]=2020` | Filter with operator                       | [filtering.md](references/filtering.md)                     |
| `sort`            | `sort=-year`             | Sort by column (prefix `-` for desc)       | [pagination-and-sort.md](references/pagination-and-sort.md) |
| `limit`           | `limit=50`               | Max rows to return (1-1000, default 100)   | [pagination-and-sort.md](references/pagination-and-sort.md) |
| `offset`          | `offset=100`             | Skip N rows                                | [pagination-and-sort.md](references/pagination-and-sort.md) |
| `cursor`          | `cursor=...`             | Keyset pagination token                    | [pagination-and-sort.md](references/pagination-and-sort.md) |
| `fields`          | `fields=year,score`      | Column projection                          | [output-formats.md](references/output-formats.md)           |
| `format`          | `format=csv`             | Output format (json, csv, tsv, xlsx)       | [output-formats.md](references/output-formats.md)           |
| `aggregate`       | `aggregate=avg(score)`   | Aggregate functions                        | [aggregation.md](references/aggregation.md)                 |
| `group_by`        | `group_by=year`          | Group rows by column                       | [aggregation.md](references/aggregation.md)                 |
| `view`            | `view=timeseries`        | Apply a named view                         |                                                             |
| `expand`          | `expand=area`            | Expand joined dimensions inline            |                                                             |
| `include_sources` | `include_sources=true`   | Show `_source_url`, `_source_page` columns |                                                             |
| `response_format` | `response_format=columnar` | Response shape: `objects` (default) or `columnar` (compact) | [output-formats.md](references/output-formats.md)           |
| `include_graph`   | `include_graph=true`     | Attach graph scores to `/meta` response    | [graph.md](references/graph.md)                             |
| `debug`           | `debug=true`             | Include generated SQL and query echo       |                                                             |

## Common Pitfalls

**Use `filter[col]=val`, not `?col=val`.** Bare column names as query params are silently ignored. The API returns a structured warning, but you still get unfiltered data back.

```bash
# Wrong - returns ALL rows, with a warning
curl '.../nces/naep?year=2024'

# Right
curl '.../nces/naep?filter[year]=2024'
```

**URL-encode brackets in curl.** Some shells interpret `[` and `]`. Use `%5B` / `%5D` or quote the URL.

```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?filter%5Byear%5D=2024'
```

**Check `warnings` in the response.** Unknown parameters produce structured `QueryWarning` objects with `code`, `message`, and `param`. The `X-OpenData-Warnings` HTTP header also carries these for piped workflows.

**Use `?debug=true` to see generated SQL.** Returns a `debug` object with `debug.query` (echo of your parameters) and `debug.sql` (the DuckDB SQL that ran). Useful for verifying filters and sorts are applied correctly.

**`aggregate` and `nest_fields` are mutually exclusive.** You get a 400 error if you combine them. Aggregation produces flat summary rows; nesting produces grouped hierarchical data.

**SQL endpoint may return 5xx errors.** The POST `/query` endpoint can fail with server-side errors. When this happens, fall back to REST aggregation (`aggregate` + `group_by` params) for the same analysis. The REST endpoint is more reliable for straightforward aggregations.

**Sorting on computed aggregation columns works.** When using `aggregate` + `group_by`, you can sort on the computed column names (e.g., `sort=-count_event_id` for `aggregate=count(event_id)`). Invalid sort fields return a 400 with `valid_values` showing available options.

**Always use `api.tryopendata.ai` for POST endpoints.** The frontend at `tryopendata.ai/api/` proxies GET requests only. POST requests to `tryopendata.ai/api/v1/query` return 405. Use `api.tryopendata.ai/v1/query` directly for SQL and cross-dataset queries.

## SQL Query

The `POST /v1/datasets/{provider}/{dataset}/query` endpoint accepts raw SQL and executes it against the dataset. Requires authentication (API key or session). The dataset table is available as `data` or `"provider/dataset"`. SQL is validated against an allowlist (SELECT only, no DDL/DML/IO) and runs with resource limits (5s timeout, 10k rows, 512MB memory).

**Parameterized queries:** Use `?` placeholders with a `params` array to avoid string quoting issues:

```json
{
  "sql": "SELECT * FROM data WHERE country IN (?, ?) AND year >= ?",
  "params": ["United States", "Japan", 2020]
}
```

This eliminates the triple-nested escaping problem (SQL quotes inside JSON inside shell). See [sql-query.md](references/sql-query.md) for details.

## Composition (Cross-Dataset Joins)

The compose endpoints let you join two datasets and preview or download the result without writing SQL. Useful for enriching a dataset with columns from a related one (e.g., joining county-level education data with census demographics).

**Workflow:** Call `/joinable` to discover what can be joined, `/compose/preview` to check the result, then `/compose/download.csv` to export. See [composition.md](references/composition.md) for full details.

```bash
# 1. What can this dataset join with?
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/joinable'

# 2. Preview the join (anonymous: 100 rows, authenticated: 5000 rows)
curl -X POST 'https://api.tryopendata.ai/v1/datasets/nces/naep/compose/preview' \
  -H 'Content-Type: application/json' \
  -d '{"joins": [{"target": "census/saipe", "source_column": "jurisdiction_name", "join_column": "name"}]}'

# 3. Download the full join as CSV (auth required)
curl -H "Authorization: Bearer od_live_..." \
  'https://api.tryopendata.ai/v1/datasets/nces/naep/compose/download.csv?target=census/saipe&source_column=jurisdiction_name&join_column=name' \
  -o composed.csv
```

## Graph Intelligence

Datasets are connected in a knowledge graph (Neo4j). Graph scores like PageRank importance and bridge centrality are available on search results, dataset metadata, and related datasets. These help surface the most connected and structurally important datasets.

**On dataset metadata:** Pass `?include_graph=true` to `/meta` to get a `graph` block with importance, bridge_score, and community info.

**On search results:** All search results now include `importance`, `bridge_score`, `community_id`, `community_label`, and `graph_available` fields. Graph scores contribute to search ranking via a multiplicative boost.

**Community browsing:** Use `/v1/graph/communities/{id}/datasets` to list datasets in a Leiden community, sorted by importance. See [graph.md](references/graph.md) for details.

## Discovery

The `GET /v1/discover` endpoint returns datasets matching a natural language query, enriched with metadata tailored for LLM agents and programmatic integrations. Results include column schemas (with units, value ranges, display names), available views, canonical questions, methodology summaries, sample rows, and relevance scores. Unlike `/v1/search`, discover is authenticated and optimized for machine consumption rather than human browsing.

**Batch discover:** `POST /v1/discover/batch` accepts multiple queries in one call, deduplicates results, and returns per-query dataset references alongside the full metadata. See [discover.md](references/discover.md) for details.

## Reference Files

| File                                                                     | When to load                                               |
| ------------------------------------------------------------------------ | ---------------------------------------------------------- |
| [references/filtering.md](references/filtering.md)                       | Writing filter expressions, checking operator syntax       |
| [references/aggregation.md](references/aggregation.md)                   | Using group_by, aggregate functions, summary queries       |
| [references/pagination-and-sort.md](references/pagination-and-sort.md)   | Paginating large results, sorting, cursor-based pagination |
| [references/column-introspection.md](references/column-introspection.md) | Discovering schema, column types, value distributions      |
| [references/output-formats.md](references/output-formats.md)             | Exporting CSV/TSV/XLSX, field projection, system columns   |
| [references/common-patterns.md](references/common-patterns.md)           | Recipes for exploratory analysis and data research         |
| [references/sql-query.md](references/sql-query.md)                       | Raw SQL query endpoint, allowed functions, security model  |
| [references/discover.md](references/discover.md)                         | Using the discover endpoint, LLM agent integration, dataset discovery |
| [references/composition.md](references/composition.md)                   | Cross-dataset joins: joinable, preview, CSV download       |
| [references/graph.md](references/graph.md)                               | Graph intelligence: communities, importance, bridge scores |
