# Common Patterns

Recipes for data research tasks using the OpenData API.

## Explore an Unfamiliar Dataset

Start with column introspection, then sample a few rows.

```bash
# 1. See what columns exist and their types
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/columns'

# 2. Check a specific column's values
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/columns/subject'

# 3. Sample a few rows
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?limit=5'

# 4. Use debug mode to verify your query
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep?filter%5Bsubject%5D=Mathematics&limit=5&debug=true'
```

## Find Unique Values in a Column

Two approaches depending on what you need:

```bash
# Option A: Use the columns endpoint (fast, includes stats)
curl '.../nces/naep/columns/jurisdiction_name'
# Returns sample_values with all distinct values for low-cardinality columns

# Option B: Use group_by + count (includes frequency)
curl '.../nces/naep?group_by=subject&aggregate=count(*)&sort=-count'
# Returns each value with its row count
```

## Compare Groups Over Time

Filter to groups of interest, aggregate by time, compare.

```bash
# Average NAEP scores by year for Illinois vs California
curl '.../nces/naep?group_by=year,jurisdiction_name&aggregate=avg(score)&filter%5Bjurisdiction_name%5D%5Bin%5D=Illinois,California&sort=year'
```

## Get All Data for a Specific Entity

```bash
# All NAEP scores for Illinois, most recent first
curl '.../nces/naep?filter%5Bjurisdiction_name%5D=Illinois&sort=-year&limit=500'

# With specific columns only
curl '.../nces/naep?filter%5Bjurisdiction_name%5D=Illinois&fields=year,subject,score&sort=-year'
```

## Download for Local Analysis

```bash
# CSV for spreadsheet/pandas
curl '.../census/saipe?format=csv&limit=1000' -o saipe.csv

# Filtered CSV
curl '.../census/saipe?filter%5Bstate_postal%5D=IL&format=csv&limit=1000' -o illinois.csv

# XLSX for Excel
curl '.../census/saipe?format=xlsx&limit=1000' -o saipe.xlsx
```

## Summary Statistics

```bash
# Quick dataset overview
curl '.../nces/naep?aggregate=count(*),min(year),max(year),count_distinct(jurisdiction_name)'

# Aggregate with filter
curl '.../nces/naep?aggregate=avg(score),min(score),max(score)&filter%5Bsubject%5D=Mathematics&filter%5Byear%5D=2024'
```

## Paginate Through All Results

```bash
# Using offset pagination
curl '.../nces/naep?limit=100&offset=0'
curl '.../nces/naep?limit=100&offset=100'
curl '.../nces/naep?limit=100&offset=200'

# Using cursor pagination (more stable for large datasets)
CURSOR=""
while true; do
  if [ -z "$CURSOR" ]; then
    RESPONSE=$(curl -s '.../nces/naep?limit=100&sort=year')
  else
    RESPONSE=$(curl -s ".../nces/naep?limit=100&cursor=$CURSOR")
  fi
  # Process RESPONSE...
  CURSOR=$(echo "$RESPONSE" | jq -r '.next_cursor // empty')
  [ -z "$CURSOR" ] && break
done
```

## Find Correlating Data Across Datasets

When doing cross-dataset analysis, query each dataset separately and join client-side by shared dimensions (state, year, etc.).

```bash
# NAEP scores by state for 2024
curl '.../nces/naep?filter%5Byear%5D=2024&group_by=jurisdiction_name&aggregate=avg(score)&sort=jurisdiction_name' -o naep_2024.json

# Poverty rates by state for 2024
curl '.../census/saipe?filter%5Byear%5D=2024&fields=state_postal,poverty_rate&sort=state_postal' -o poverty_2024.json

# Join the two locally by state
```

## Verify Filters Are Working

Use `?debug=true` to confirm your query is doing what you expect:

```bash
curl '.../nces/naep?filter%5Bjurisdiction_name%5D=Illinois&filter%5Byear%5D%5Bgte%5D=2015&sort=-year&debug=true' | jq '.debug'
```

Check `debug.query.filters` to confirm filters were parsed correctly, and `debug.sql` to see the WHERE clause.

## Verify Dataset Quality Before Committing

Not all datasets have clean schemas. Before building analysis around a dataset:

1. **Check column descriptions** via `/columns` — if descriptions are null or use opaque codes, the dataset may be hard to use without domain knowledge
2. **Check for views** via `/views` — curated views often provide human-readable labels and pre-filtered subsets
3. **Sample a few rows** — confirm values match what the column descriptions claim (e.g., verify whether a "price" column is in dollars, cents, or index points)
4. **Inspect null rates** — some columns exist in the schema but are null across all records

If a dataset's schema is opaque, look for alternative datasets covering the same topic via the discover endpoint.

## Large Dataset Aggregation (1M+ Rows)

For large datasets, prefer server-side aggregation. Try SQL first, fall back to REST if it fails:

```bash
# Option A: SQL (more flexible)
curl -X POST ".../datasets/provider/dataset/query" \
  -H "Authorization: Bearer $OPENDATA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"sql": "SELECT year, category, COUNT(*) as count FROM data WHERE year >= 2000 GROUP BY 1, 2 ORDER BY 1"}'

# Option B: REST aggregation (more reliable)
curl -H "Authorization: Bearer $OPENDATA_API_KEY" \
  ".../datasets/provider/dataset?aggregate=count(id)&group_by=year&filter%5Byear%5D%5Bgte%5D=2000&limit=100"
```

## Local Computation for Derived Values

When you need a single derived number (median, correlation, percent change) rather than a full dataset for a chart, compute locally with Python stdlib:

```bash
python3 -c "
import json, sys, statistics
data = json.load(sys.stdin)
vals = [r['value'] for r in data]
print(f'median: {statistics.median(vals):.0f}')
" < data.json
```

Python's `json`, `statistics`, and `csv` modules are available without installing anything.
