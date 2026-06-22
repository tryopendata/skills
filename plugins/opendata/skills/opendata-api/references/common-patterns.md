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

## Enrich a Dataset with Composition

Use the compose endpoints when you want to add columns from one dataset to another without writing SQL. Simpler than the SQL join approach, with built-in cardinality protection.

```bash
# 1. What can NAEP scores join with?
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/joinable'

# 2. Preview: add poverty rates alongside scores
curl -X POST 'https://api.tryopendata.ai/v1/datasets/nces/naep/compose/preview' \
  -H 'Content-Type: application/json' \
  -d '{"joins": [{"target": "census/saipe", "source_column": "jurisdiction_name", "join_column": "name"}]}'

# 3. Download the full joined result
curl -H "Authorization: Bearer ${OPENDATA_API_KEY}" \
  'https://api.tryopendata.ai/v1/datasets/nces/naep/compose/download.csv?target=census/saipe&source_column=jurisdiction_name&join_column=name' \
  -o enriched.csv
```

Check `populated_rows` in the preview response to gauge join quality. If it's low relative to total rows, the join columns may not align well. See [composition.md](references/composition.md) for the full API reference.

## Find Related and Connected Datasets

Use graph intelligence to discover datasets related to one you're already working with:

```bash
# Related datasets (semantic + join + graph signals)
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/related'

# Dataset metadata with graph scores
curl 'https://api.tryopendata.ai/v1/datasets/nces/naep/meta?include_graph=true'

# Browse all datasets in the same community
curl 'https://api.tryopendata.ai/v1/graph/communities/7/datasets?sort=importance'

# Search with graph-boosted ranking (automatic, always on)
curl 'https://api.tryopendata.ai/v1/search?q=education+scores'
```

See [graph.md](references/graph.md) for details on graph scores and community browsing.

## Join Data Across Datasets (SQL)

Use `POST /v1/query` to join multiple datasets in a single SQL query. Reference datasets as `"provider/dataset"` (quoted slash notation) or `provider.dataset` (dot notation). For simpler joins, consider the [composition endpoints](references/composition.md) instead.

```bash
# Join NAEP scores with poverty rates by state
curl -X POST 'https://api.tryopendata.ai/v1/query' \
  -H "Authorization: Bearer ${OPENDATA_API_KEY}" \
  -H 'Content-Type: application/json' \
  -d '{"sql": "SELECT n.jurisdiction_name, AVG(n.score) as avg_score, s.poverty_rate FROM \"nces/naep\" n JOIN \"census/saipe\" s ON n.jurisdiction_name = s.name AND n.year = s.year WHERE n.year = ? GROUP BY n.jurisdiction_name, s.poverty_rate ORDER BY avg_score DESC", "params": [2024]}'
```

Limits: max 5 datasets per query, 150 MB combined parquet, 10s timeout. See [sql-query.md](references/sql-query.md) for details.

**When identifiers don't match across datasets**, use string functions to normalize them in the query. For example, NDC drug codes differ between FDA (hyphenated text like `0003-0893-21`) and CMS (11-digit integers like `300893021`). Use LPAD and SPLIT_PART to create a common format:

```sql
-- Normalize NDC codes across FDA and CMS datasets
ON LPAD(SPLIT_PART(fda.product_ndc, '-', 1), 5, '0')
   || LPAD(SPLIT_PART(fda.product_ndc, '-', 2), 4, '0')
 = SUBSTR(LPAD(CAST(cms.ndc AS VARCHAR), 11, '0'), 1, 9)
```

**Fallback: client-side join.** If the SQL endpoint returns errors, query each dataset separately and join by shared dimensions (state, year, FIPS code) in your application:

```bash
curl '.../nces/naep?filter%5Byear%5D=2024&group_by=jurisdiction_name&aggregate=avg(score)' -o naep.json
curl '.../census/saipe?filter%5Byear%5D=2024&fields=state_postal,poverty_rate' -o poverty.json
# Join locally by state name
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
2. **Check for views** via `/meta` (`available_views`) — curated views often provide human-readable labels and pre-filtered subsets. Use colon syntax in SQL to query them: `FROM "bls/cpi-u:enriched"`
3. **Sample a few rows** — confirm values match what the column descriptions claim (e.g., verify whether a "price" column is in dollars, cents, or index points)
4. **Inspect null rates** — some columns exist in the schema but are null across all records

If a dataset's schema is opaque, look for alternative datasets covering the same topic via the discover endpoint.

## Large Dataset Aggregation (1M+ Rows)

For large datasets, prefer server-side aggregation. Try SQL first, fall back to REST if it fails:

```bash
# Option A: SQL (more flexible)
curl -X POST ".../datasets/provider/dataset/query" \
  -H "Authorization: Bearer ${OPENDATA_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"sql": "SELECT year, category, COUNT(*) as count FROM data WHERE year >= 2000 GROUP BY 1, 2 ORDER BY 1"}'

# Option B: REST aggregation (more reliable)
curl -H "Authorization: Bearer ${OPENDATA_API_KEY}" \
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

## Healthcare Dataset Join Patterns

Healthcare datasets often use different identifier formats across agencies. Common join strategies:

**Drug data (CMS + FDA):** CMS datasets (Part D, NADAC, Medicaid) identify drugs by brand name or NDC integer. FDA NDC Directory uses hyphenated NDC text. Join through the FDA NDC Directory as a bridge table, matching by brand name to Part D and by normalized NDC code to NADAC.

**Geographic data (CDC + Census + CMS):** Most datasets share state FIPS codes or state names. County-level data uses 5-digit FIPS codes. Always verify the identifier column type: some store FIPS as integers (dropping leading zeros), others as zero-padded strings.

**Temporal alignment:** CMS data is typically annual (calendar year). CDC provisional data uses 12-month trailing periods. OECD data may lag 1-2 years. When joining across agencies, check that year values actually represent the same time period.
