# Column Chart

Periodic data and categorical comparisons. Vertical bars with category labels on the x-axis.

**Column vs Bar:** Column = vertical (category on x-axis, value on y-axis). Use column for time periods (Q1, Jan, 2023) or when comparing a small number of categories. Use bar for ranked lists.

## Encoding Rules

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | Yes | nominal, ordinal, temporal |
| y | Yes | quantitative |
| color | No | nominal, ordinal, quantitative |
| size | No | quantitative |
| detail | No | nominal |

## Spec

```typescript
{
  type: "column",
  data: DataRow[],
  encoding: {
    x: { field: string, type: "nominal"|"ordinal"|"temporal", axis?, scale? },
    y: { field: string, type: "quantitative", axis?, scale? },
    color?: { field: string, type: "nominal"|"ordinal"|"quantitative" },
  },
  chrome?: Chrome,
  annotations?: Annotation[],
  labels?: LabelConfig,
  legend?: LegendConfig,
  responsive?: boolean,
  theme?: ThemeConfig,
  darkMode?: DarkMode,
}
```

Use `color` with a nominal/ordinal field to create stacked or grouped columns.

**Format strings:** Both `axis.format` and `labels.format` support d3-format strings with an optional literal suffix (e.g., `"$,.2~f"` outputs `$3.1` with trailing zeros trimmed, `".1f%"` outputs `12.5%`). Set both to keep axis ticks and column value labels consistent. See SKILL.md for the full format string reference.

## Builder

```typescript
import { columnChart } from "@opendata-ai/openchart-core";

const spec = columnChart(data, "quarter", "revenue", {
  color: "department",
  chrome: { title: "Revenue by quarter" },
});
```

## Example

```json
{
  "type": "column",
  "data": [
    { "quarter": "Q1", "sales": 420 },
    { "quarter": "Q2", "sales": 510 },
    { "quarter": "Q3", "sales": 390 },
    { "quarter": "Q4", "sales": 680 }
  ],
  "encoding": {
    "x": { "field": "quarter", "type": "ordinal" },
    "y": {
      "field": "sales",
      "type": "quantitative",
      "axis": { "format": "$,.0f", "label": "Sales (thousands)" }
    }
  },
  "chrome": {
    "title": "Q4 sales outpaced the rest of the year",
    "subtitle": "Quarterly sales in thousands, 2024",
    "source": "Source: Internal sales data"
  },
  "labels": { "density": "all", "format": "$,.0f" }
}
```
