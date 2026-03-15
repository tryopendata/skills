# Pie & Donut Charts

Part-to-whole composition. Donut is preferred over pie (center space can show total).

**Important:** Keep categories under 6. For 7+ categories, group the smallest into "Other" or use a bar chart instead. Pie/donut shows composition of ONE whole. Don't use it for comparing across groups.

## Encoding Rules

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | No | (not used) |
| y | Yes | quantitative |
| color | Yes | nominal, ordinal |
| size | No | quantitative |
| detail | No | nominal |

No x-axis. The `y` channel is the slice value, `color` is the category.

## Spec

```typescript
{
  type: "pie" | "donut",
  data: DataRow[],
  encoding: {
    y: { field: string, type: "quantitative" },
    color: { field: string, type: "nominal"|"ordinal" },
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

## Builder

```typescript
import { pieChart } from "@opendata-ai/openchart-core";

// pieChart(data, category, value) - category maps to color, value maps to y
const spec = pieChart(data, "source", "share", {
  chrome: { title: "Energy mix breakdown" },
});
```

## Example

```json
{
  "type": "donut",
  "data": [
    { "source": "Solar", "share": 42 },
    { "source": "Wind", "share": 31 },
    { "source": "Hydro", "share": 18 },
    { "source": "Other", "share": 9 }
  ],
  "encoding": {
    "y": { "field": "share", "type": "quantitative" },
    "color": { "field": "source", "type": "nominal" }
  },
  "chrome": {
    "title": "Solar dominates new renewable capacity",
    "subtitle": "Share of new renewable energy installations, 2024",
    "source": "Source: IRENA"
  }
}
```
