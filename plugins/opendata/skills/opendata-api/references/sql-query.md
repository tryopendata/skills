# SQL Query

Execute raw SQL against a dataset. Requires authentication.

## Endpoint

```
POST /v1/datasets/{provider}/{dataset}/query
```

**Auth required:** API key (`X-API-Key` header) or session cookie.

## Request

```json
{
  "sql": "SELECT year, AVG(score) as avg_score FROM data GROUP BY year ORDER BY year DESC",
  "timeout_ms": 5000,
  "row_limit": 1000
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `sql` | string | required | SQL query to execute |
| `timeout_ms` | integer | 5000 | Query timeout in milliseconds (max 10000) |
| `row_limit` | integer | 10000 | Max rows to return (max 10000) |

## Response

```json
{
  "data": [
    {"year": 2024, "avg_score": 267.5},
    {"year": 2023, "avg_score": 265.1}
  ],
  "columns": ["year", "avg_score"],
  "row_count": 2,
  "execution_time_ms": 42,
  "truncated": false
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data` | array | Query result rows |
| `columns` | array | Column names in result order |
| `row_count` | integer | Number of rows returned |
| `execution_time_ms` | number | Server-side execution time |
| `truncated` | boolean | `true` if results hit the row limit |

## Table Naming

The dataset is available as two table references:

| Reference | Example |
|-----------|---------|
| `data` | `SELECT * FROM data` |
| `"provider/dataset"` | `SELECT * FROM "nces/naep"` |

Use `data` for single-dataset queries. The qualified name is useful for clarity or when cross-referencing in documentation.

## Allowed SQL Features

| Category | Allowed |
|----------|---------|
| Statements | SELECT only |
| JOINs | All JOIN types (against same dataset) |
| Subqueries | Yes |
| CTEs | Non-recursive only (`WITH ... AS`) |
| Window functions | Yes |
| CASE/WHEN | Yes |
| UNION/INTERSECT/EXCEPT | Yes |

## Allowed Functions

**Aggregate:** COUNT, SUM, AVG, MIN, MAX

**Math:** ABS, CEIL, FLOOR, ROUND, SQRT, POWER

**String:** CONCAT, SUBSTRING, LENGTH, UPPER, LOWER, TRIM, REPLACE

**Date:** MAKE_DATE, DATE_TRUNC, DATE_PART, EXTRACT

**Null handling:** COALESCE, NULLIF

**Type casting:** CAST, TRY_CAST

**Window:** ROW_NUMBER, RANK, LAG, LEAD

## Blocked Features

- DDL/DML (CREATE, DROP, INSERT, UPDATE, DELETE, ALTER)
- I/O functions (read_csv, read_parquet, read_json, httpfs, etc.)
- PRAGMA and SET statements
- Recursive CTEs (`WITH RECURSIVE`)
- Multiple statements (semicolon-separated)
- System functions and information_schema access

## Resource Limits

| Limit | Default | Max |
|-------|---------|-----|
| Timeout | 5 seconds | 10 seconds |
| Row limit | 10,000 | 10,000 |
| Memory | 512 MB | 512 MB |

## Error Codes

| Status | Code | When |
|--------|------|------|
| 400 | `SQL_VALIDATION_ERROR` | SQL fails allowlist validation (blocked function, DDL, etc.) |
| 404 | - | Dataset not found or has no Parquet files |
| 408 | - | Query exceeded timeout |

## Examples

**Basic filter and sort:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/nces/naep/query' \
  -H 'X-API-Key: your-key' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT * FROM data WHERE year >= 2020 ORDER BY year DESC LIMIT 10"}'
```

**Aggregation with GROUP BY:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/nces/naep/query' \
  -H 'X-API-Key: your-key' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT jurisdiction, AVG(score) as avg_score FROM data WHERE year = 2024 GROUP BY jurisdiction ORDER BY avg_score DESC"}'
```

**Window function:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/worldbank/gdp-per-capita/query' \
  -H 'X-API-Key: your-key' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT country_name, year, gdp_per_capita, LAG(gdp_per_capita) OVER (PARTITION BY country_name ORDER BY year) as prev_year FROM data WHERE country_name = '\''United States'\'' ORDER BY year DESC LIMIT 10"}'
```

**CTE for top-N per group:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/nces/naep/query' \
  -H 'X-API-Key: your-key' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "WITH ranked AS (SELECT *, ROW_NUMBER() OVER (PARTITION BY subject ORDER BY score DESC) as rn FROM data WHERE year = 2024) SELECT * FROM ranked WHERE rn <= 5"}'
```
