---
name: opendata-api
description: Query the OpenData API for data research and analysis. Use when fetching dataset rows, filtering, sorting, aggregating, inspecting columns, or building data pipelines against OpenData endpoints.
---

# OpenData Query API

Query datasets stored as Parquet files through a REST API backed by DuckDB. The API returns JSON by default, with support for CSV, TSV, and XLSX exports.

**Base URL:** `https://api.tryopendata.ai` (production) or `http://localhost:8000` (local dev). Default to use production

All data endpoints live under `/v1/datasets/`.

## Endpoints

| Method | Path                                               | Description                                                  |
| ------ | -------------------------------------------------- | ------------------------------------------------------------ |
| GET    | `/v1/datasets/{provider}/{dataset}`                | Query dataset rows (flat) or list subdatasets (hierarchical) |
| GET    | `/v1/datasets/{provider}/{dataset}/{subdataset}`   | Query subdataset rows                                        |
| GET    | `/v1/datasets/{provider}/{dataset}/columns`        | Column metadata and statistics                               |
| GET    | `/v1/datasets/{provider}/{dataset}/columns/{name}` | Single column detail with full value list                    |
| GET    | `/v1/datasets/{provider}/{dataset}/meta`           | Dataset metadata (schema, views, capabilities)               |
| GET    | `/v1/datasets/{provider}/{dataset}/views`          | List available views                                         |
| POST   | `/v1/datasets/{provider}/{dataset}/query`          | Execute SQL query (authenticated)                            |
| GET    | `/v1/discover`                                     | Search datasets with enriched metadata for LLM agents        |

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

## SQL Query

The `POST /v1/datasets/{provider}/{dataset}/query` endpoint accepts raw SQL and executes it against the dataset. Requires authentication (API key or session). The dataset table is available as `data` or `"provider/dataset"`. SQL is validated against an allowlist (SELECT only, no DDL/DML/IO) and runs with resource limits (5s timeout, 10k rows, 512MB memory).

## Discovery

The `GET /v1/discover` endpoint returns datasets matching a natural language query, enriched with metadata tailored for LLM agents and programmatic integrations. Results include column schemas, canonical questions, methodology summaries, and relevance scores. Unlike `/v1/search`, discover is authenticated and optimized for machine consumption rather than human browsing.

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
