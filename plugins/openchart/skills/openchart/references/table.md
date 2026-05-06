# Data Table

Rich data tables with sorting, search, pagination, and inline visualizations.

## TableSpec

For the full shape (`TableSpec`, `ColumnConfig`, the per-feature configs `HeatmapConfig`/`BarConfig`/`SparklineConfig`/`ImageConfig`), load `TableSpec` from `index.d.ts`. Animation reuses the chart `AnimationSpec`.

Behavioral defaults the type doesn't show:

| Field | Default |
| --- | --- |
| `responsive` | `true` |
| `search` | off (omit to hide the search box) |
| `pagination` | off; pass `{ pageSize: N }` to enable |
| `compact` | off (full padding + font sizes) |
| `column.sortable` | `true` |
| `column.align` | `"right"` for numeric data, `"left"` otherwise |
| `column.image` size | 24x24 px |
| `column.sparkline.type` | `"line"` |

**Visual feature precedence** (if a column sets multiple): `sparkline > bar > heatmap > image > flag > categoryColors`.

## Builder

```typescript
import { dataTable } from "@opendata-ai/openchart-core";

// Auto-generates columns from data keys if omitted
const spec = dataTable(data, { search: true, pagination: { pageSize: 20 } });

// Or provide explicit columns
const spec = dataTable(data, {
  columns: [
    { key: "name", label: "Name" },
    { key: "revenue", label: "Revenue", format: "$,.0f", bar: { color: "#1b7fa3" } },
  ],
});
```

## Event Handlers

**Vanilla / all frameworks:**

| Handler | Signature | Fires when |
| --- | --- | --- |
| `onRowClick` | `(row: DataRow) => void` | User clicks a row |
| `onStateChange` | `(state: { sort?, search?, page? }) => void` | Any table state changes |

**React-only convenience props** (decompose `onStateChange` into individual callbacks):

| Handler | Signature | Fires when |
| --- | --- | --- |
| `onSortChange` | `(sort: SortState \| null) => void` | Sort changes |
| `onSearchChange` | `(query: string) => void` | Search input changes |
| `onPageChange` | `(page: number) => void` | Pagination changes |

**SortState:** `{ column: string, direction: "asc" | "desc" }`

Sort cycling: clicking the same column cycles none -> asc -> desc -> none. Clicking a different column resets to asc.

## Vanilla Instance API

```typescript
import { createTable } from "@opendata-ai/openchart-vanilla";

const table = createTable(container, spec, {
  responsive: true,
  onRowClick: (row) => console.log(row),
  onStateChange: (state) => console.log(state),  // { sort, search, page }
});

table.update(newSpec);                  // re-render with new spec
table.resize();                         // recalculate dimensions
table.export("csv");                    // export all filtered/sorted rows as CSV string
table.getState();                       // { sort: SortState|null, search: string, page: number }
table.setState({ search: "q", page: 0 }); // programmatic state control
table.destroy();                        // cleanup
```

## Controlled Mode (React)

Pass `sort`, `search`, or `page` props to control state externally:

```tsx
import { DataTable } from '@opendata-ai/openchart-react';

<DataTable
  spec={tableSpec}
  sort={sortState}
  search={searchQuery}
  page={currentPage}
  onSortChange={(sort) => setSortState(sort)}
  onSearchChange={(q) => setSearchQuery(q)}
  onPageChange={(p) => setCurrentPage(p)}
/>
```

Omit these props for uncontrolled mode (the table manages its own state).

## Example

```json
{
  "type": "table",
  "data": [
    {
      "city": "Tokyo",
      "country": "JP",
      "pop": 37.4,
      "density": 6158,
      "trend": [34.5, 35.2, 36.1, 37.0, 37.4]
    },
    {
      "city": "Delhi",
      "country": "IN",
      "pop": 32.9,
      "density": 11320,
      "trend": [25.7, 28.1, 30.3, 31.8, 32.9]
    },
    {
      "city": "Shanghai",
      "country": "CN",
      "pop": 29.2,
      "density": 3826,
      "trend": [24.2, 25.6, 27.1, 28.5, 29.2]
    },
    {
      "city": "Sao Paulo",
      "country": "BR",
      "pop": 22.6,
      "density": 7523,
      "trend": [20.9, 21.3, 21.8, 22.2, 22.6]
    },
    {
      "city": "Mexico City",
      "country": "MX",
      "pop": 22.1,
      "density": 9544,
      "trend": [20.1, 20.8, 21.3, 21.7, 22.1]
    }
  ],
  "columns": [
    { "key": "city", "label": "City" },
    { "key": "country", "label": "", "flag": true, "width": "48px" },
    {
      "key": "pop",
      "label": "Population (M)",
      "format": ".1f",
      "bar": { "color": "#1b7fa3" }
    },
    {
      "key": "density",
      "label": "Density /km2",
      "format": ",.0f",
      "heatmap": { "palette": "orange" }
    },
    {
      "key": "trend",
      "label": "2020-2024",
      "sparkline": { "type": "line", "color": "#6a9f58" }
    }
  ],
  "chrome": {
    "title": "World's largest metropolitan areas",
    "subtitle": "Population in millions, 2024 estimates",
    "source": "Source: UN World Urbanization Prospects"
  },
  "search": true,
  "pagination": { "pageSize": 10 },
  "stickyFirstColumn": true
}
```
