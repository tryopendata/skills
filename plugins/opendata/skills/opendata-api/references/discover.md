# Discover API Reference

The discover endpoint returns datasets matching a natural language query, enriched with metadata designed for LLM agents and programmatic integrations.

## Endpoint

```
GET /v1/discover
```

**Auth:** Required. API key (`od_live_*` via `Authorization: Bearer` header) or Clerk session cookie.

## Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `q` | string | **required** | Natural language query |
| `provider` | string | null | Filter by provider slug |
| `category` | string | null | Filter by category/subject tag |
| `format` | string | null | Filter by data format |
| `limit` | int (1-20) | 5 | Max results |
| `status` | string | "ready" | Filter by dataset status |
| `include_columns` | bool | true | Include column schemas per result |

## Response Shape

```json
{
  "results": [
    {
      "name": "Consumer Price Index - All Urban Consumers",
      "provider": "bls",
      "path": "/v1/datasets/bls/cpi-u",
      "description": "CPI-U measures the average change over time in prices...",
      "rows": 1847520,
      "format": "csv",
      "categories": ["economics", "inflation"],
      "temporal_granularity": "monthly",
      "geographic_scope": "national",
      "canonical_questions": [
        "What is the current inflation rate?",
        "How have consumer prices changed over time?"
      ],
      "methodology_summary": "Based on prices of a market basket...",
      "known_limitations": ["Does not cover rural consumers"],
      "columns": [
        {
          "name": "year",
          "type": "integer",
          "description": "Reference year",
          "sample_values": [2020, 2021, 2022, 2023, 2024],
          "distinct_count": 78
        }
      ],
      "relevance": 0.94,
      "semantic_score": 0.82,
      "quality_score": 0.87,
      "is_enriched": true
    }
  ],
  "total": 3,
  "query": "monthly inflation data",
  "processing_time_ms": 142
}
```

## Field Descriptions

### Result fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Human-readable dataset name |
| `provider` | string | Provider slug (e.g. `bls`, `census`) |
| `path` | string | API path to query this dataset |
| `description` | string | What the dataset contains |
| `rows` | int | Total row count |
| `format` | string | Original source format |
| `categories` | string[] | Subject tags assigned during enrichment |
| `temporal_granularity` | string | Time resolution (daily, monthly, annual, etc.) |
| `geographic_scope` | string | Geographic coverage (national, state, county, etc.) |
| `canonical_questions` | string[] | Example questions this dataset can answer |
| `methodology_summary` | string | Brief description of how the data was collected |
| `known_limitations` | string[] | Caveats and coverage gaps |
| `columns` | object[] | Column schemas (when `include_columns=true`) |
| `total_columns` | int | Total column count (only present when columns are capped at 50) |
| `relevance` | float | Combined relevance score (0-1) |
| `semantic_score` | float | Semantic similarity to query (0-1) |
| `quality_score` | float | Dataset quality/completeness score (0-1) |
| `is_enriched` | bool | Whether LLM enrichment metadata is available |

### Column fields (within `columns`)

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Column name |
| `type` | string | Data type (integer, varchar, double, etc.) |
| `description` | string | What the column represents |
| `sample_values` | any[] | Representative values |
| `distinct_count` | int | Number of unique values |

### Top-level response fields

| Field | Type | Description |
|-------|------|-------------|
| `results` | object[] | Matching datasets |
| `total` | int | Total number of matches |
| `query` | string | Echo of the input query |
| `processing_time_ms` | int | Server-side processing time in milliseconds |

## Common Patterns

### Basic discovery

```bash
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/discover?q=county+poverty+data"
```

### Using results to query data

Take the `path` from a discover result and use it to fetch rows:

```bash
# 1. Discover datasets about inflation
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/discover?q=monthly+inflation+data"

# 2. Use query_endpoint from the top result to fetch data
curl "https://api.tryopendata.ai/v1/datasets/bls/cpi-u?filter[year][gte]=2020&sort=-year&limit=50"
```

### Filter by provider

```bash
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/discover?q=employment&provider=bls"
```

### Without column schemas

Omit column schemas for a lighter response when you only need dataset-level metadata:

```bash
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/discover?q=inflation&include_columns=false"
```

## Differences from /v1/search

| | `/v1/search` | `/v1/discover` |
|---|---|---|
| **Auth** | Not required | Required (API key or session) |
| **Audience** | Human users, web UI | LLM agents, programmatic clients |
| **Results** | Basic metadata, facets | Enriched metadata, column schemas, canonical questions |
| **Scoring** | Text relevance | Semantic + quality scoring |
| **Pagination** | offset/limit (up to 100) | limit only (max 20) |
| **Facets** | Yes (provider, format, category) | No |
| **Column schemas** | No | Yes (via `include_columns`) |
