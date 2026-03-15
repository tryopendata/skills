# Aggregation

Server-side aggregation using `?aggregate` and `?group_by` parameters. Produces flat summary rows instead of raw data.

## Syntax

```
?aggregate=func(column),func(column)
?group_by=column&aggregate=func(column)
```

## Supported Functions

| Function | Description | Example |
|----------|-------------|---------|
| `count(*)` | Count all rows | `aggregate=count(*)` |
| `count(col)` | Count non-null values | `aggregate=count(score)` |
| `count_distinct(col)` | Count distinct values | `aggregate=count_distinct(state)` |
| `sum(col)` | Sum values | `aggregate=sum(enrollment)` |
| `avg(col)` | Average | `aggregate=avg(score)` |
| `min(col)` | Minimum | `aggregate=min(year)` |
| `max(col)` | Maximum | `aggregate=max(score)` |

## Output Column Names

Aggregate results use auto-generated aliases:

| Expression | Output column |
|------------|--------------|
| `count(*)` | `count` |
| `avg(score)` | `avg_score` |
| `sum(enrollment)` | `sum_enrollment` |
| `count_distinct(state)` | `count_distinct_state` |

## Examples

**Total row count (no group_by):**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?aggregate=count(*)'
# Returns: [{"count": 2288}]
```

**Average score by year:**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?group_by=year&aggregate=avg(score)&sort=year'
# Returns: [{"year": 2003, "avg_score": 234.5}, {"year": 2005, "avg_score": 237.1}, ...]
```

**Multiple aggregates:**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?group_by=year&aggregate=avg(score),min(score),max(score),count(*)&sort=-year'
```

**Filtered aggregation:**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?group_by=year&aggregate=avg(score)&filter%5Bjurisdiction_name%5D=Illinois&sort=year'
```

**Multi-column grouping:**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?group_by=year,subject&aggregate=avg(score)&sort=year'
```

## How It Works

When `aggregate` is provided:
1. Filters are applied first (WHERE clause)
2. Rows are grouped by `group_by` columns (if any)
3. Aggregate functions compute summary values per group
4. Results are sorted by group_by columns (default) or by `sort` param
5. Standard `limit`/`offset` pagination applies to the grouped results

Without `group_by`, the entire (filtered) dataset collapses to a single summary row.

## Restrictions

**`aggregate` and `nest_fields`/`nest_field` are mutually exclusive.** Nesting produces hierarchical grouping with raw rows; aggregation produces flat summary rows. Combining them returns a 400 error.

**Column names must exist in the schema.** Referencing a nonexistent column returns a 400 error listing available columns.

**Only allowlisted functions.** Anything outside `count, sum, avg, min, max, count_distinct` is rejected. This is a security measure against SQL injection.

**`count(*)` is the only function that accepts `*`.** Using `avg(*)` or `sum(*)` returns a 400 error.
