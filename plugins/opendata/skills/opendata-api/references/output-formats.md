# Output Formats

## Format Selection

Two ways to choose output format, in priority order:

1. **`Accept` header** (standard HTTP content negotiation)
2. **`?format` query parameter** (convenience for curl/browser)

| Format | Accept Header | Query Param | Content-Type |
|--------|--------------|-------------|--------------|
| JSON | `application/json` | `format=json` | `application/json` |
| CSV | `text/csv` | `format=csv` | `text/csv` |
| TSV | `text/tab-separated-values` | `format=tsv` | `text/tab-separated-values` |
| XLSX | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | `format=xlsx` | Excel spreadsheet |

Default is JSON when neither is specified.

## Columnar JSON Format

For JSON responses, an optional `response_format` parameter controls the response shape:

| Endpoint | Parameter | Default | Description |
|----------|-----------|---------|-------------|
| REST data (`GET /v1/datasets/...`) | `?response_format=objects` | `columnar` | Query param |
| SQL query (`POST .../query`) | `"response_format": "objects"` | `columnar` | Request body field |

**Columnar format** returns data as arrays of arrays instead of arrays of objects, saving ~45% tokens by not repeating column names on every row:

```json
{
  "columns": ["country", "year", "value"],
  "types": ["string", "integer", "float"],
  "rows": [
    ["United States", 2023, 13473],
    ["Japan", 2023, 5365]
  ],
  "total_rows": 4869,
  "row_count": 2
}
```

**Objects format** (traditional) repeats key names on every row:

```json
{
  "data": [
    {"country": "United States", "year": 2023, "value": 13473},
    {"country": "Japan", "year": 2023, "value": 5365}
  ]
}
```

Both REST and SQL endpoints default to columnar. Pass `?response_format=objects` (REST) or `"response_format": "objects"` (SQL body) to get the traditional object-per-row format.

## Examples

```bash
# JSON (default)
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?limit=10'

# CSV download
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?format=csv&limit=1000' -o naep.csv

# TSV
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?format=tsv&limit=1000'

# XLSX
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?format=xlsx&limit=1000' -o naep.xlsx

# Using Accept header
curl -H 'Accept: text/csv' 'https://api.tryopendata.ai/v1/datasets/nces/naep?limit=1000'
```

Non-JSON formats return raw data rows with no response envelope (no `data`, `total_rows`, etc.). Filters, sort, and limit still apply.

## Column Projection

Use `?fields` to select specific columns. Reduces response size and focuses on relevant data.

```bash
# Only year and score
curl '.../nces/naep?fields=year,score&limit=10'

# Combined with filters
curl '.../nces/naep?fields=year,score,jurisdiction_name&filter%5Bjurisdiction_name%5D=Illinois&sort=-year'

# CSV with specific columns
curl '.../nces/naep?fields=year,score&format=csv&limit=1000' -o subset.csv
```

Column names in `fields` are validated against the schema. Invalid names produce an error.

## System Columns

Datasets that combine multiple source files have hidden system columns tracking data provenance:

| Column | Description |
|--------|-------------|
| `_source_url` | URL the row was fetched from |
| `_source_page` | Page number within the source (for PDFs) |

These are hidden by default. Pass `?include_sources=true` to expose them:

```bash
curl '.../census/state-population?include_sources=true&limit=5'
```

## Debug Mode

Pass `?debug=true` to get query diagnostics in the response:

```bash
curl '.../nces/naep?filter%5Bjurisdiction_name%5D=Illinois&sort=-year&fields=year,score&debug=true'
```

Response includes a `debug` object:
```json
{
  "columns": [...],
  "rows": [...],
  "debug": {
    "query": {
      "filters": {"jurisdiction_name": {"eq": "Illinois"}},
      "sort": "-year",
      "fields": ["year", "score"]
    },
    "sql": "SELECT \"year\", \"score\" FROM read_parquet('...') WHERE \"jurisdiction_name\" = 'Illinois' ORDER BY \"year\" DESC LIMIT 100 OFFSET 0"
  }
}
```

The `query` sub-object mirrors your input parameters. The `sql` field shows the exact DuckDB SQL that executed. Only present when `?debug=true` is passed.
