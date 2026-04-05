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
