# Filter Syntax

Filters use bracket notation on the `filter` parameter. Two forms:

```
?filter[column]=value           # Implicit eq operator
?filter[column][operator]=value # Explicit operator
```

## Operators

| Operator | SQL | Example |
|----------|-----|---------|
| `eq` | `=` | `filter[year]=2024` or `filter[year][eq]=2024` |
| `ne` | `!=` | `filter[status][ne]=pending` |
| `gt` | `>` | `filter[year][gt]=2020` |
| `gte` | `>=` | `filter[year][gte]=2020` |
| `lt` | `<` | `filter[score][lt]=500` |
| `lte` | `<=` | `filter[score][lte]=300` |
| `in` | `IN` | `filter[state][in]=CA,TX,NY` |
| `like` | `LIKE` | `filter[name][like]=*Illinois*` (use `*` as wildcard) |

The `eq` operator is the default when no operator is specified.

## Examples

**Simple equality:**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?filter%5Bjurisdiction_name%5D=Illinois'
```

**Numeric range:**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?filter%5Byear%5D%5Bgte%5D=2015&filter%5Byear%5D%5Blte%5D=2024'
```

**Multiple values (IN):**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/census/saipe?filter%5Bstate_postal%5D%5Bin%5D=CA,TX,NY'
```

**Pattern matching (LIKE):**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?filter%5Bjurisdiction_name%5D%5Blike%5D=*New*'
```

**Combining multiple filters (AND logic):**
```bash
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?filter%5Bjurisdiction_name%5D=Illinois&filter%5Byear%5D%5Bgte%5D=2015&filter%5Bsubject%5D=Mathematics'
```

## Type Coercion

Filter values are automatically coerced based on column type:
- Integer columns: `"2024"` becomes `2024`
- Float columns: `"3.14"` becomes `3.14`
- Boolean columns: `"true"`, `"1"`, `"yes"` become `True`
- String columns: no coercion

## Column Validation

Filter column names are validated against the dataset schema. If you reference a column that doesn't exist, you get a `FilterError` with the list of valid columns.

## Gotchas

**URL-encode brackets.** `[` is `%5B`, `]` is `%5D`. Most HTTP clients and libraries handle this automatically, but raw curl needs encoding or quoting.

**Case-sensitive column names.** Column names are lowercase with underscores (e.g., `jurisdiction_name`, not `Jurisdiction Name`). Use the `/columns` endpoint to check exact names.

**Bare params are not filters.** `?year=2024` does nothing. You need `?filter[year]=2024`. The API returns an `unknown_param` warning if you make this mistake.

**LIKE wildcard is `*`, not `%`.** The API converts `*` to SQL `%` internally. Use `*` in filter values.
