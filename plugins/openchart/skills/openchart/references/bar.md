# Bar Chart

Rankings and comparisons. Horizontal bars with category labels on the y-axis.

**Bar vs Column:** Bar = horizontal (category on y-axis, value on x-axis). Use bar for ranked lists where category labels need room to read. Use column for time periods.

## Encoding Rules

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | Yes | quantitative |
| y | Yes | nominal, ordinal |
| color | No | nominal, ordinal, quantitative |
| size | No | quantitative |
| detail | No | nominal |

## Spec

```typescript
{
  type: "bar",
  data: DataRow[],
  encoding: {
    x: { field: string, type: "quantitative", axis?, scale? },
    y: { field: string, type: "nominal"|"ordinal", axis?, scale? },
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

Use `color` with a nominal/ordinal field to create stacked bars.

**Format strings:** Both `axis.format` and `labels.format` support d3-format strings with an optional literal suffix (e.g., `".1f%"` outputs `12.5%`). Set both to keep axis ticks and bar value labels consistent. See SKILL.md for the full format string reference.

## Builder

```typescript
import { barChart } from "@opendata-ai/openchart-core";

// Note: barChart(data, category, value) - category maps to y, value maps to x
const spec = barChart(data, "country", "gdp", {
  chrome: { title: "GDP by country" },
});
```

## Example

```json
{
  "type": "bar",
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
