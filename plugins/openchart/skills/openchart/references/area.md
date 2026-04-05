# Area Chart

Trends with volume emphasis. Filled region below the line.

## Encoding Rules

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | Yes | temporal, ordinal |
| y | Yes | quantitative |
| color | No | nominal, ordinal |
| size | No | quantitative |
| detail | No | nominal |
| opacity | No | quantitative |
| strokeDash | No | nominal, ordinal |
| tooltip | No | any |

Same encoding as line charts. Use area when you want to emphasize the magnitude (volume under the curve), not just the trend direction.

## Spec

```typescript
{
  mark: "area" | { type: "area", interpolate?: "linear"|"monotone"|"step"|"step-before"|"step-after"|"basis"|"cardinal"|"natural" },
  data: DataRow[],
  encoding: {
    x: { field: string, type: "temporal"|"ordinal", axis?, scale? },
    y: { field: string, type: "quantitative", axis?, scale? },
    color?: { field: string, type: "nominal"|"ordinal" },
    size?: { field: string, type: "quantitative" },
    detail?: { field: string, type: "nominal" },
    opacity?: { field: string, type: "quantitative" },
    strokeDash?: { field: string, type: "nominal"|"ordinal" },
    tooltip?: { field: string } | { field: string }[],
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

## Example

```json
{
  "mark": "area",
  "data": [
    { "quarter": "Q1 2023", "revenue": 1200 },
    { "quarter": "Q2 2023", "revenue": 1450 },
    { "quarter": "Q3 2023", "revenue": 1380 },
    { "quarter": "Q4 2023", "revenue": 1690 },
    { "quarter": "Q1 2024", "revenue": 1820 }
  ],
  "encoding": {
    "x": { "field": "quarter", "type": "ordinal" },
    "y": {
      "field": "revenue",
      "type": "quantitative",
      "axis": { "format": "$,.0f", "title": "Revenue" }
    }
  },
  "chrome": {
    "title": "Revenue grew 52% from Q1 2023 to Q1 2024",
    "subtitle": "Quarterly revenue in USD",
    "source": "Source: Company financials"
  }
}
```
