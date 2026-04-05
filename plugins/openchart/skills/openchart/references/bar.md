# Bar Chart

Rankings, comparisons, and categorical data. Covers both horizontal bars and vertical columns.

**Orientation:** The engine infers orientation from encoding types. x=quantitative + y=nominal/ordinal = horizontal bar. x=nominal/ordinal/temporal + y=quantitative = vertical column. You can override with `mark: { type: "bar", orient: "horizontal" | "vertical" }`.

**Column vs Bar:** Column (vertical) works best for time periods (Q1, Jan, 2023) or comparing a small number of categories. Bar (horizontal) works best for ranked lists where category labels need room to breathe.

## Horizontal Bar Encoding

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | Yes | quantitative |
| y | Yes | nominal, ordinal |
| color | No | nominal, ordinal, quantitative |
| size | No | quantitative |
| detail | No | nominal |

## Vertical Bar (Column) Encoding

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
  mark: "bar" | { type: "bar", orient?: "horizontal" | "vertical" },
  data: DataRow[],
  encoding: {
    // Horizontal: x=quantitative, y=nominal/ordinal
    // Vertical:   x=nominal/ordinal/temporal, y=quantitative
    x: { field: string, type: FieldType, axis?, scale? },
    y: { field: string, type: FieldType, axis?, scale? },
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

Use `color` with a nominal/ordinal field to create multi-series bars. By default, multiple rows per category are **stacked** (cumulative segments). Set `stack: null` on the quantitative encoding channel to produce **grouped** (side-by-side) bars instead. See [encoding-channels.md](encoding-channels.md) for details.

**Format strings:** Both `axis.format` and `labels.format` support d3-format strings with an optional literal suffix (e.g., `".1f%"` outputs `12.5%`). Set both to keep axis ticks and bar value labels consistent. See [format-strings.md](format-strings.md) for the full reference.

## Builders

```typescript
import { barChart, columnChart } from "@opendata-ai/openchart-core";

// Horizontal bar: barChart(data, category, value)
const bar = barChart(data, "country", "gdp", {
  chrome: { title: "GDP by country" },
});

// Vertical column: columnChart(data, category, value)
const col = columnChart(data, "quarter", "revenue", {
  color: "department",
  chrome: { title: "Revenue by quarter" },
});
```

## Example: Horizontal Bar

```json
{
  "mark": "bar",
  "data": [
    { "country": "Luxembourg", "gdp": 126598 },
    { "country": "Ireland", "gdp": 106899 },
    { "country": "Switzerland", "gdp": 93720 },
    { "country": "Norway", "gdp": 89154 },
    { "country": "Singapore", "gdp": 82808 }
  ],
  "encoding": {
    "x": {
      "field": "gdp",
      "type": "quantitative",
      "axis": { "format": "$,.0f" }
    },
    "y": { "field": "country", "type": "nominal" }
  },
  "chrome": {
    "title": "Luxembourg leads the world in GDP per capita",
    "subtitle": "Top 5 countries by GDP per capita (PPP), 2024 estimates",
    "source": "Source: IMF World Economic Outlook"
  }
}
```

## Example: Grouped Horizontal Bar

```json
{
  "mark": "bar",
  "data": [
    { "expense": "Healthcare", "weeks": 4, "era": "1984" },
    { "expense": "Healthcare", "weeks": 28, "era": "2024" },
    { "expense": "Housing", "weeks": 10, "era": "1984" },
    { "expense": "Housing", "weeks": 22, "era": "2024" }
  ],
  "encoding": {
    "x": {
      "field": "weeks",
      "type": "quantitative",
      "stack": null
    },
    "y": { "field": "expense", "type": "nominal" },
    "color": { "field": "era", "type": "nominal" }
  },
  "chrome": {
    "title": "Everything costs more weeks of your life",
    "subtitle": "Weeks of median income needed per year, 1984 vs 2024"
  }
}
```

## Example: Percentage Stacked Bar (Normalize)

```json
{
  "mark": "bar",
  "data": [
    { "region": "West", "source": "Solar", "gwh": 120 },
    { "region": "West", "source": "Wind", "gwh": 80 },
    { "region": "West", "source": "Gas", "gwh": 200 },
    { "region": "East", "source": "Solar", "gwh": 60 },
    { "region": "East", "source": "Wind", "gwh": 140 },
    { "region": "East", "source": "Gas", "gwh": 300 }
  ],
  "encoding": {
    "x": { "field": "region", "type": "nominal" },
    "y": {
      "field": "gwh",
      "type": "quantitative",
      "stack": "normalize",
      "axis": { "format": ".0%" }
    },
    "color": { "field": "source", "type": "nominal" }
  },
  "chrome": {
    "title": "The West leans greener on energy mix",
    "subtitle": "Share of electricity generation by source"
  }
}
```

## Example: Vertical Column

```json
{
  "mark": "bar",
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
      "axis": { "format": "$,.0f", "title": "Sales (thousands)" }
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
