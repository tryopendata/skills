# Composition (Cross-Dataset Joins)

Join two datasets via the API without writing SQL. Preview the result interactively, then download the full join as CSV.

## Workflow

1. **Discover joinable targets** via `/joinable`
2. **Preview the join** via `/compose/preview` (check column overlap, join quality)
3. **Download the result** via `/compose/download.csv` (auth required)

## List Joinable Datasets

```
GET /v1/datasets/{provider}/{dataset}/joinable
```

**Auth:** Public (no auth required).

Returns datasets that can be joined with the given dataset, ranked by confidence. Filters out low-signal joins (date-only at low confidence, cross-community). Falls back to a small set of low-signal matches when the filter would otherwise return nothing.

```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/joinable'
```

### Response

```json
{
  "source_dataset": {
    "provider": "nces",
    "slug": "naep",
    "name": "NAEP Scores"
  },
  "targets": [
    {
      "dataset_id": "uuid-here",
      "provider": "census",
      "slug": "saipe",
      "name": "Small Area Income and Poverty Estimates",
      "confidence": 0.85,
      "join_type": "geographic",
      "source_columns": ["jurisdiction_name"],
      "target_columns": ["name"],
      "same_community": true,
      "target_row_count": 58000,
      "hop_count": 1,
      "low_signal": false
    }
  ],
  "used_fallback": false
}
```

### Target Fields

| Field | Type | Description |
|-------|------|-------------|
| `dataset_id` | UUID | Target dataset ID |
| `provider` | string | Target provider slug |
| `slug` | string | Target dataset slug |
| `name` | string | Human-readable name |
| `confidence` | float | Join confidence (0-1) |
| `join_type` | string | Join relationship type (geographic, temporal, etc.) |
| `source_columns` | string[] | Suggested columns in the source dataset |
| `target_columns` | string[] | Suggested columns in the target dataset |
| `same_community` | boolean | Whether both datasets are in the same graph community |
| `target_row_count` | int or null | Row count of target dataset |
| `hop_count` | int | Graph distance (always 1 for direct joins) |
| `low_signal` | boolean | Whether this is a low-confidence fallback match |

When Neo4j is unavailable, returns an empty `targets` list (not a 503).

## Preview a Join

```
POST /v1/datasets/{provider}/{dataset}/compose/preview
```

**Auth:** Optional. Anonymous users get 100 rows and 10 attempts per session. Authenticated users get 5000 rows.

Executes a LEFT JOIN between the base dataset and one target. Exactly one join spec per request.

### Request

```json
{
  "joins": [
    {
      "target": "census/saipe",
      "source_column": "jurisdiction_name",
      "join_column": "name"
    }
  ],
  "columns": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `joins` | array (exactly 1) | Single join spec |
| `joins[].target` | string | Target dataset path (`provider/slug`) |
| `joins[].source_column` | string | Column in the base dataset to join on |
| `joins[].join_column` | string | Column in the target dataset to join on |
| `columns` | string[] or null | Columns to project. Null = auto-select |

### Response

```json
{
  "data": [{"jurisdiction_name": "Illinois", "year": 2024, "score": 267, "saipe_poverty_rate": 11.2}],
  "columns": ["jurisdiction_name", "year", "score", "saipe_poverty_rate"],
  "column_types": ["string", "integer", "integer", "float"],
  "column_sources": {
    "jurisdiction_name": "naep",
    "year": "naep",
    "score": "naep",
    "saipe_poverty_rate": "saipe"
  },
  "populated_rows": 45,
  "cardinality_warning": false,
  "effective_joins": [{"target": "census/saipe", "source_column": "jurisdiction_name", "join_column": "name"}]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data` | object[] | Result rows |
| `columns` | string[] | Column names |
| `column_types` | string[] | Column types |
| `column_sources` | object | Maps each column to the dataset it came from |
| `populated_rows` | int | Rows where the join matched (join quality signal) |
| `cardinality_warning` | boolean | True when join fan-out looks suspicious |
| `effective_joins` | object[] | Echo of the join spec used |

### Cardinality Preflight

For large base datasets (>10k rows), the API runs a COUNT(*) before executing the preview:
- If count exceeds 1M rows: returns **400** with `CARDINALITY_EXCEEDED`
- If count exceeds 2x the base rows (or 100k): runs the preview but emits `cardinality_warning: true` and caps rows at 100k

### Error Codes

| Status | When |
|--------|------|
| 400 | Invalid join column, lookup error, or cardinality exceeded |
| 404 | Dataset not found or has no data |
| 408 | Query timeout |
| 429 | Anonymous session ran out of preview attempts |

## Download Composed CSV

```
GET /v1/datasets/{provider}/{dataset}/compose/download.csv
```

**Auth:** Required (returns 401 if not authenticated).

Streams the composed join as CSV. Mirrors the shape of the preview endpoint.

### Query Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `target` | string | yes | Target dataset path (e.g., `census/saipe`) |
| `source_column` | string | yes | Column in the base dataset to join on |
| `join_column` | string | yes | Column in the target dataset to join on |

```bash
curl -H "Authorization: Bearer od_live_..." \
  'https://api.tryopendata.ai/v1/datasets/nces/naep/compose/download.csv?target=census/saipe&source_column=jurisdiction_name&join_column=name' \
  -o composed.csv
```

### Limits

| Limit | Value |
|-------|-------|
| Max rows | 100,000 |
| Rate limit | 10 downloads/hour per user |
| Concurrent streams | 5 (process-wide) |
| Wall-clock timeout | 300s |
| DuckDB query timeout | 60s |

### Error Codes

| Status | When |
|--------|------|
| 401 | Not authenticated |
| 404 | Dataset not found or has no data |
| 429 | Rate limit exceeded |
| 503 | At capacity (includes `Retry-After: 30` header) |

## Column Naming in Composed Results

Base dataset columns keep their original names. Joined columns are prefixed with the target dataset slug: `{slug}_{column}`. For example, joining with `census/saipe` produces columns like `saipe_poverty_rate`, `saipe_median_income`.

## Composition vs SQL Joins

| | Composition endpoints | `POST /v1/query` (SQL) |
|---|---|---|
| **Auth** | Preview: optional. Download: required | Required |
| **Complexity** | Single join, no SQL knowledge needed | Arbitrary SQL, multiple joins |
| **Cardinality protection** | Built-in preflight checks | None (hits timeout/row limits) |
| **Output** | JSON preview or CSV stream | JSON (columnar or objects) |
| **Use case** | Quick enrichment, data exploration | Complex analysis, custom aggregations |
