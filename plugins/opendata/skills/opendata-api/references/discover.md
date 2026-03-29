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
      "name": "Life Expectancy",
      "provider": "owid",
      "path": "/v1/datasets/owid/life-expectancy",
      "description": "Life expectancy data by country...",
      "rows": 19504,
      "format": "auto",
      "categories": ["demographics", "health"],
      "temporal_granularity": "annual",
      "geographic_scope": "country",
      "canonical_questions": [
        "How has life expectancy changed in my country over the past 50 years?"
      ],
      "methodology_summary": "Compiled from UN Population Division...",
      "known_limitations": ["Historical estimates before 1950 are reconstructed"],
      "columns": [
        {
          "name": "life_expectancy",
          "type": "float",
          "sample_values": ["28.15", "37.93", "51.02"],
          "distinct_count": 996,
          "display_name": "Life Expectancy (years)",
          "unit": "years",
          "semantic_type": "measure",
          "value_range": {"min": 18.2, "max": 86.37}
        },
        {
          "name": "country",
          "type": "string",
          "sample_values": ["Afghanistan", "Japan", "Germany"],
          "distinct_count": 264,
          "display_name": "Country",
          "semantic_type": "geographic",
          "semantic_subtype": "country"
        }
      ],
      "available_views": [
        {"name": "latest", "description": "Most recent life expectancy by country"},
        {"name": "by-country", "description": "Life expectancy time series by country"}
      ],
      "sample_rows": [
        {"country": "Japan", "year": 2023, "life_expectancy": 84.71, "country_code": "JPN"}
      ],
      "relevance": 0.94,
      "semantic_score": 0.82,
      "quality_score": 0.87,
      "is_enriched": true
    }
  ],
  "total": 3,
  "query": "life expectancy by country",
  "processing_time_ms": 142,
  "available_categories": ["agriculture", "demographics", "economics", "education", "energy", "environment", "finance", "government", "health", "housing", "labor", "other", "redistricting", "transportation"]
}
```

**Note:** Null fields are omitted from the response. Only fields with values are included, keeping the payload clean.

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
| `available_views` | object[] | Named views that pre-aggregate or transform the raw data |
| `sample_rows` | object[] | Up to 5 representative rows (when available from enrichment) |
| `relevance` | float | Combined relevance score (0-1) |
| `semantic_score` | float | Semantic similarity to query (0-1) |
| `quality_score` | float | Dataset quality/completeness score (0-1) |
| `is_enriched` | bool | Whether LLM enrichment metadata is available |

### Column fields (within `columns`)

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Column name |
| `type` | string | Data type (integer, string, float, etc.) |
| `description` | string | What the column represents |
| `sample_values` | any[] | Representative values |
| `distinct_count` | int | Number of unique values |
| `display_name` | string | Human-readable column name (e.g. "GDP per Capita (2021 PPP $)") |
| `unit` | string | Unit of measurement (e.g. "USD", "percent", "per capita", "index") |
| `semantic_type` | string | Semantic classification: geographic, identifier, measure, temporal, category |
| `semantic_subtype` | string | Specific classification: country, state, county, fips_code, zip_code, year, month, date, code, name |
| `value_range` | object | `{"min": ..., "max": ...}` for numeric/date columns |

### Top-level response fields

| Field | Type | Description |
|-------|------|-------------|
| `results` | object[] | Matching datasets |
| `total` | int | Total number of matches |
| `query` | string | Echo of the input query |
| `processing_time_ms` | int | Server-side processing time in milliseconds |
| `available_categories` | string[] | All platform category slugs available for filtering with `category=` |

## Batch Discover

Discover multiple queries in a single call. Deduplicates datasets that appear in multiple results.

### Endpoint

```
POST /v1/discover/batch
```

### Request

```json
{
  "queries": [
    "healthcare spending per capita",
    "life expectancy by country",
    "employment labor statistics"
  ],
  "limit_per_query": 3,
  "deduplicate": true,
  "provider": null,
  "category": null,
  "format": null,
  "status": "ready"
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `queries` | string[] | **required** | 1-5 search queries |
| `limit_per_query` | int (1-10) | 5 | Max results per query |
| `deduplicate` | bool | true | Merge datasets appearing in multiple query results |
| `provider` | string | null | Filter all queries by provider |
| `category` | string | null | Filter all queries by category |
| `format` | string | null | Filter all queries by data format |
| `status` | string | "ready" | Filter all queries by status |

### Response

```json
{
  "query_results": [
    {
      "query": "healthcare spending per capita",
      "dataset_refs": [
        {"provider": "owid", "slug": "healthcare-spending", "relevance": 0.98},
        {"provider": "who", "slug": "health-expenditure", "relevance": 0.95}
      ]
    },
    {
      "query": "life expectancy by country",
      "dataset_refs": [
        {"provider": "owid", "slug": "life-expectancy", "relevance": 0.97},
        {"provider": "owid", "slug": "healthcare-spending", "relevance": 0.82}
      ]
    }
  ],
  "datasets": [
    { "... full DiscoverResult for each unique dataset ..." }
  ],
  "join_hints": [
    {
      "datasets": ["owid/healthcare-spending", "owid/life-expectancy"],
      "join_key": "country_code",
      "compatibility": "exact_match",
      "shared_values": 3
    }
  ],
  "processing_time_ms": 1234
}
```

`query_results` maps each query to its matched datasets with per-query relevance scores. `datasets` contains deduplicated full metadata, sorted by highest relevance. `join_hints` suggests which datasets can be joined and on which columns.

### Join Hints

When batch discover returns multiple datasets, `join_hints` identifies column pairs that are likely joinable across datasets. Use these to write cross-dataset SQL queries without manual schema inspection.

| Field | Type | Description |
|-------|------|-------------|
| `datasets` | string[] | Two dataset paths (provider/slug) that can be joined |
| `join_key` | string | Column name to join on |
| `compatibility` | string | Confidence level: `exact_match`, `semantic_match`, or `name_match` |
| `shared_values` | int | Estimated shared values from sample overlap (null if unknown) |

Compatibility levels:
- **exact_match**: Same column name and both classified as geographic or temporal. Highest confidence.
- **semantic_match**: Different column names but matching semantic subtype (e.g., both are "country"). Medium confidence.
- **name_match**: Same column name but no semantic type info. Lowest confidence.

### Batch Discover Use Cases

**Multi-agent research dispatch:** Discover all datasets at once, then dispatch agents with pre-identified datasets (skip discover in subagents).

```bash
# 1. Main thread discovers everything
curl -X POST -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  "https://api.tryopendata.ai/v1/discover/batch" \
  -d '{"queries": ["healthcare spending", "health outcomes", "drug prices"]}'

# 2. Dispatch agents with specific datasets - no discover needed
# Agent 1 gets owid/healthcare-spending, Agent 2 gets who/life-expectancy, etc.
```

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

# 2. Use path from the top result to fetch data
curl "https://api.tryopendata.ai/v1/datasets/bls/cpi-u?filter[year][gte]=2020&sort=-year&limit=50"
```

### Filter by provider

```bash
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/discover?q=employment&provider=bls"
```

### Filter by category

Use `available_categories` from any discover response to see valid values:

```bash
curl -H "Authorization: Bearer od_live_..." \
  "https://api.tryopendata.ai/v1/discover?q=spending&category=health"
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
| **Column schemas** | No | Yes (with units, value ranges, display names) |
| **Views** | No | Yes (available_views per dataset) |
| **Categories** | No | Yes (available_categories in response) |
| **Sample rows** | No | Yes (when enriched) |
| **Batch** | No | Yes (`POST /v1/discover/batch`) |
