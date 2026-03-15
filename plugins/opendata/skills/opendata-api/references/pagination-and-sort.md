# Pagination and Sort

## Offset Pagination

Standard offset/limit pagination for random access.

```bash
# First page (default: 100 rows)
curl '.../nces/naep?limit=50'

# Second page
curl '.../nces/naep?limit=50&offset=50'

# Third page
curl '.../nces/naep?limit=50&offset=100'
```

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| `limit` | 100 | 1-1000 | Max rows per request |
| `offset` | 0 | 0+ | Rows to skip |

The response includes `total_rows` (unfiltered count) and `filtered_rows` (count after filters, when filters are applied) so you can calculate total pages.

## Cursor Pagination

For stable pagination through large or frequently updated datasets. Pass the `next_cursor` from a previous response to get the next page.

```bash
# First page
curl '.../nces/naep?limit=50&sort=year'
# Response includes: "next_cursor": "eyJ..."

# Next page
curl '.../nces/naep?limit=50&cursor=eyJ...'
```

`cursor` and `offset` are mutually exclusive. Using both returns a 400 error.

When there are no more results, `next_cursor` is `null`.

## Sorting

Sort by one or more columns. Prefix with `-` for descending order.

```bash
# Ascending by year
curl '.../nces/naep?sort=year'

# Descending by year
curl '.../nces/naep?sort=-year'

# Multi-column: year desc, then score asc
curl '.../nces/naep?sort=-year,score'
```

When no `sort` is specified:
- With a view that has `default_sort`: uses the view's default
- Without a view: sorts by the first column (for consistent pagination)

Sort column names are validated against the schema. Invalid column names produce an error.
