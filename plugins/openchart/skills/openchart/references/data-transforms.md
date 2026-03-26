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
