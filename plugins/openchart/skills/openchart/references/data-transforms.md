# Data Transforms

Apply transforms to data before encoding. Transforms run in array order.

```typescript
transform?: Transform[]
```

## FilterTransform

Subset rows by predicate:

```json
{ "filter": { "field": "value", "gte": 100 } }
```

Supported operators: `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `in`, `nin`, `contains`, `startsWith`, `endsWith`.

## BinTransform

Bin a continuous field into discrete ranges:

```json
{ "bin": true, "field": "temperature", "as": "tempBin" }
```

## CalculateTransform

Derive new fields from existing data:

```json
{ "calculate": { "op": "/", "field": "revenue", "field2": "cost" }, "as": "margin" }
```

Supported ops: `+`, `-`, `*`, `/`.

## TimeUnitTransform

Truncate temporal fields to a unit:

```json
{ "timeUnit": "yearmonth", "field": "date", "as": "month" }
```

Supported units: `year`, `quarter`, `month`, `yearmonth`, `yearquarter`, `date`, `day`, `hours`, `minutes`.

## AggregateTransform

Group rows and compute summary statistics:

```json
{
  "aggregate": [
    { "op": "mean", "field": "temperature", "as": "avgTemp" },
    { "op": "count", "as": "n" }
  ],
  "groupby": ["region", "year"]
}
```

Supported ops: `count`, `sum`, `mean`, `median`, `min`, `max`, `variance`, `stdev`, `distinct`, `q1`, `q3`.

## FoldTransform

Pivot wide-format columns into long-format key/value rows:

```json
{ "fold": ["revenue", "cost", "profit"], "as": ["metric", "value"] }
```

Each input row produces one output row per folded field. `as` defaults to `["key", "value"]` if omitted. Use this to reshape wide data for multi-series charts where each column becomes a series.

## WindowTransform

Compute values relative to other rows in sort order within a partition. Supports lag, lead, diff, pct_change, cumsum, rank, and first_value.

```json
{
  "window": [{ "op": "lag", "field": "value", "offset": 1, "as": "prev_value" }],
  "sort": [{ "field": "date" }],
  "groupby": ["country"]
}
```

Window ops:

| Op | Description | Default offset |
| --- | --- | --- |
| `lag` | Value from N rows before | 1 |
| `lead` | Value from N rows ahead | 1 |
| `diff` | Current minus lagged value | 1 |
| `pct_change` | `(current - lagged) / lagged` | 1 |
| `cumsum` | Running total (nulls treated as 0) | - |
| `rank` | 1-based competition rank | - |
| `first_value` | First value in the partition | - |

**YoY change on monthly data:**
```json
{
  "window": [{ "op": "pct_change", "field": "cpi", "offset": 12, "as": "yoy_pct" }],
  "sort": [{ "field": "date" }]
}
```

**Index/rebase to 100 (recipe):** Compose `first_value` + calculate:
```json
"transform": [
  { "window": [{ "op": "first_value", "field": "price", "as": "base_price" }], "sort": [{ "field": "date" }], "groupby": ["country"] },
  { "calculate": { "op": "/", "field": "price", "field2": "base_price" }, "as": "indexed" },
  { "calculate": { "op": "*", "field": "indexed", "value": 100 }, "as": "rebased" }
]
```

## RelativeTimeRef

Filter comparison operators (`gte`, `gt`, `lte`, `lt`, `range`) accept data-relative references for temporal fields:

```json
{ "filter": { "field": "date", "gte": { "anchor": "max", "offset": -3, "unit": "year" } } }
```

This keeps the last 3 years of data relative to the most recent date in the dataset. `anchor` is `"max"` or `"min"` (computed from the data, not the clock). Units: `day`, `week`, `month`, `quarter`, `year`.

**1Y/3Y/5Y filter buttons:** Use different offsets:
```json
{ "filter": { "field": "date", "gte": { "anchor": "max", "offset": -1, "unit": "year" } } }
{ "filter": { "field": "date", "gte": { "anchor": "max", "offset": -3, "unit": "year" } } }
{ "filter": { "field": "date", "gte": { "anchor": "max", "offset": -5, "unit": "year" } } }
```

**Range with two relative refs:**
```json
{ "filter": { "field": "date", "range": [
  { "anchor": "max", "offset": -5, "unit": "year" },
  { "anchor": "max", "offset": -1, "unit": "year" }
] } }
```
