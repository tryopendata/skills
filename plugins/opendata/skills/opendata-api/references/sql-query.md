# SQL Query

Execute raw SQL against a dataset. Requires authentication.

## Endpoint

```
POST /v1/datasets/{provider}/{dataset}/query
```

**Auth required:** API key (`Authorization: Bearer` header) or session cookie.

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
| JOINs | All JOIN types, including cross-dataset joins via `POST /v1/query` |
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

## Cross-Dataset Joins

Use `POST /v1/query` to join data across multiple datasets in a single SQL query.

### Cross-Dataset Endpoint

```
POST /v1/query
```

Same auth, request body, and response shape as the per-dataset endpoint. The difference: table references in your SQL are resolved to datasets automatically.

### Table Reference Syntax

Reference datasets as `provider.dataset` (dot notation) or `"provider/dataset"` (quoted slash notation):

```sql
-- Dot notation (clean, works for simple names)
FROM fred.gdp g JOIN fred.cpi c ON g.date = c.date

-- Quoted slash notation (works for all names, including hyphens)
FROM "fred/gdp" g JOIN "fred/unemployment-rate" u ON g.date = u.date
```

Use quoted slash notation for dataset names with hyphens (dot notation won't parse `fred.unemployment-rate` because the hyphen is a minus operator).

### Cross-Dataset Examples

**Economic indicators:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/query' \
  -H 'Authorization: Bearer $OPENDATA_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT g.date, g.value as gdp, u.value as unemployment FROM fred.gdp g JOIN \"fred/unemployment-rate\" u ON g.date = u.date WHERE EXTRACT(YEAR FROM g.date) >= 2020 ORDER BY g.date"}'
```

**With CTEs:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/query' \
  -H 'Authorization: Bearer $OPENDATA_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "WITH recent_gdp AS (SELECT date, value FROM fred.gdp WHERE EXTRACT(YEAR FROM date) >= 2020) SELECT r.date, r.value as gdp, c.value as cpi FROM recent_gdp r JOIN fred.cpi c ON r.date = c.date ORDER BY r.date"}'
```

### Cross-Dataset Limits

| Limit | Value |
|-------|-------|
| Max datasets per query | 5 |
| Combined parquet size | 150 MB |
| Timeout | Same as per-dataset (5s default, 10s max) |
| Row limit | Same as per-dataset (10,000 max) |

### Cross-Dataset Error Codes

| Status | Code | When |
|--------|------|------|
| 400 | `INVALID_TABLE_REF` | Bare table name without provider (use `provider.dataset` notation) |
| 400 | `DATASET_LIMIT_EXCEEDED` | More than 5 datasets referenced |
| 400 | `SIZE_LIMIT_EXCEEDED` | Combined parquet files exceed 150 MB |
| 400 | `NO_DATASETS` | No dataset references found in query |
| 404 | - | Referenced dataset not found |

## Resource Limits

| Limit | Default | Max |
|-------|---------|-----|
| Timeout | 5 seconds | 10 seconds |
| Row limit | 10,000 | 10,000 |
| Memory | 512 MB | 512 MB |

## Fallback Strategy

**If the SQL endpoint fails with a 5xx error**, use the REST aggregation endpoint (`?aggregate=...&group_by=...`) for the same analysis. REST aggregation is more reliable for common operations like counts, averages, and group-by queries.

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
  -H 'Authorization: Bearer $OPENDATA_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT * FROM data WHERE year >= 2020 ORDER BY year DESC LIMIT 10"}'
```

**Aggregation with GROUP BY:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/nces/naep/query' \
  -H 'Authorization: Bearer $OPENDATA_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT jurisdiction, AVG(score) as avg_score FROM data WHERE year = 2024 GROUP BY jurisdiction ORDER BY avg_score DESC"}'
```

**Window function:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/worldbank/gdp-per-capita/query' \
  -H 'Authorization: Bearer $OPENDATA_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT country_name, year, gdp_per_capita, LAG(gdp_per_capita) OVER (PARTITION BY country_name ORDER BY year) as prev_year FROM data WHERE country_name = '\''United States'\'' ORDER BY year DESC LIMIT 10"}'
```

**CTE for top-N per group:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/nces/naep/query' \
  -H 'Authorization: Bearer $OPENDATA_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "WITH ranked AS (SELECT *, ROW_NUMBER() OVER (PARTITION BY subject ORDER BY score DESC) as rn FROM data WHERE year = 2024) SELECT * FROM ranked WHERE rn <= 5"}'
```

**Year-over-year percent change:**
```bash
curl -X POST 'https://api.tryopendata.ai/v1/datasets/fred/gdp/query' \
  -H 'Authorization: Bearer $OPENDATA_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT date, value as gdp, ROUND(100.0 * (value - LAG(value) OVER (ORDER BY date)) / LAG(value) OVER (ORDER BY date), 1) as yoy_pct_change FROM data WHERE EXTRACT(YEAR FROM date) >= 2020 ORDER BY date"}'
```
