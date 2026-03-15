# Column Introspection

Discover dataset schema, column types, and value distributions before writing queries.

## List All Columns

```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/columns'
```

Response:
```json
{
  "columns": [
    {
      "name": "year",
      "type": "integer",
      "raw_type": "BIGINT",
      "distinct_count": 11,
      "null_count": 0,
      "min": 2003,
      "max": 2024,
      "sample_values": [2003, 2005, 2007, 2009, 2011]
    },
    {
      "name": "jurisdiction_name",
      "type": "string",
      "raw_type": "VARCHAR",
      "distinct_count": 52,
      "null_count": 0,
      "sample_values": ["Alabama", "Alaska", "Arizona", "Arkansas", "California"]
    }
  ],
  "row_count": 2288
}
```

## Single Column Detail

Get full stats for one column. For low-cardinality columns (under 1000 distinct values), returns all distinct values instead of just a sample.

```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/columns/jurisdiction_name'
```

Response:
```json
{
  "name": "jurisdiction_name",
  "type": "string",
  "raw_type": "VARCHAR",
  "distinct_count": 52,
  "null_count": 0,
  "sample_values": ["Alabama", "Alaska", "Arizona", ..., "Wyoming"]
}
```

Returns 404 if the column doesn't exist, with `available_columns` in the error details.

## Column Fields

| Field | Description | Present for |
|-------|-------------|-------------|
| `name` | Column name | All columns |
| `type` | Normalized type: `string`, `integer`, `float`, `boolean`, `date`, `timestamp` | All columns |
| `raw_type` | DuckDB-native type: `VARCHAR`, `BIGINT`, `DOUBLE`, `BOOLEAN`, `DATE`, `TIMESTAMP` | All columns |
| `distinct_count` | Approximate count of distinct values (sampled) | All columns |
| `null_count` | Approximate null count (sampled) | All columns |
| `min` | Minimum value | Numeric, date, timestamp |
| `max` | Maximum value | Numeric, date, timestamp |
| `sample_values` | Sample of distinct values (up to 20 for all-columns, up to 1000 for single column) | Low-cardinality columns |

Stats are computed from a 10,000-row sample for efficiency on large datasets.

## Type Mapping

| DuckDB Type | Normalized Type |
|-------------|----------------|
| `BIGINT`, `INTEGER`, `SMALLINT`, `TINYINT`, `HUGEINT` | `integer` |
| `DOUBLE`, `FLOAT`, `DECIMAL` | `float` |
| `VARCHAR`, `BLOB` | `string` |
| `BOOLEAN` | `boolean` |
| `DATE` | `date` |
| `TIMESTAMP`, `TIMESTAMP WITH TIME ZONE` | `timestamp` |
| `TIME` | `time` |

## Workflow

Start with `/columns` to understand a dataset, then build targeted queries:

```bash
# 1. What columns exist?
curl '.../census/saipe/columns'

# 2. What states are available?
curl '.../census/saipe/columns/state_postal'

# 3. Now query with the right filter
curl '.../census/saipe?filter%5Bstate_postal%5D=IL&sort=-year&limit=10'
```
