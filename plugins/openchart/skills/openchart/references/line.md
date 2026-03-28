# Line Chart

Trends over time or ordered sequences. Connects data points with lines.

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

Use `color` to differentiate 2-5 series. For 6+ series, filter to the top 5 or use `detail` to group without color encoding.

## Spec

```typescript
{
  mark: "line" | { type: "line", point?: boolean | "transparent", interpolate?: "linear"|"monotone"|"step"|"step-before"|"step-after"|"basis"|"cardinal"|"natural" },
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
  labels?: { density?: "all"|"auto"|"endpoints"|"none", format?: string },
  legend?: { position?: LegendPosition },
  responsive?: boolean,
  theme?: ThemeConfig,
  darkMode?: "auto"|"force"|"off",
}
```

**Tip:** Use `labels: { density: "endpoints" }` for line charts to show only the first and last value per series.

**Caution with `endpoints` and long series names:** Endpoint labels include the series name from the `color` field. Long names like `"Waukegan (68% low-income)"` reserve a large right margin, creating dead space. If series names are more than ~15 characters, either abbreviate them or use `labels: { density: "none" }` with `legend: { position: "top" }` instead.

## Axis Formatting

Always format Y-axis ticks with units and provide enough tick density for readers to identify values.

**Tick density:** Add `tickCount: 5` or `tickCount: 6` so the axis shows intermediate values. Without it, many charts default to showing only 0 and the max, which makes it impossible to read specific data points from the chart.

**Percentage format:** When data values represent percentages (e.g., `12.5` means 12.5%), use `format: ".0f%"` (or `".1f%"` for one decimal). This renders ticks as `10%`, `20%`, `30%` instead of bare numbers. Remove `(%)` from the axis title when the format already includes `%` to avoid redundancy.

```json
"y": {
  "field": "rate",
  "type": "quantitative",
  "axis": { "title": "Chronic absence rate", "format": ".0f%", "tickCount": 5 },
  "scale": { "domain": [0, 50] }
}
```

**Match labels to axis format:** If you set `labels: { density: "all" }`, use the same format string: `labels: { density: "all", format: ".0f%" }`.

## Y-Domain Sizing

Set the y-domain ceiling to roughly 5-10% above the highest data value. A chart with data peaking at 48.8 should use `domain: [0, 52]`, not `[0, 55]`. Too much headroom wastes chart space and compresses the visual differences between series. Leave just enough room for any annotation text that sits above the peak.

**Per-series styling:** Use `seriesStyles` to differentiate reference series from primary data (e.g., dashed lines for benchmarks, reduced opacity for context series). See [series-styles.md](series-styles.md) for the full spec.

## Builder

```typescript
import { lineChart } from "@opendata-ai/openchart-core";

const spec = lineChart(data, "date", "revenue", {
  color: "region",
  chrome: { title: "Revenue trend by region" },
});
```

## Example

```json
{
  "mark": "line",
  "data": [
    { "year": "2020", "rate": 5.4 },
    { "year": "2021", "rate": 8.1 },
    { "year": "2022", "rate": 14.2 },
    { "year": "2023", "rate": 19.7 },
    { "year": "2024", "rate": 23.1 }
  ],
  "encoding": {
    "x": { "field": "year", "type": "temporal" },
    "y": {
      "field": "rate",
      "type": "quantitative",
      "axis": { "title": "Adoption rate", "format": ".0f%", "tickCount": 5 }
    }
  },
  "chrome": {
    "title": "EV adoption accelerated sharply after 2021",
    "subtitle": "Percentage of new car sales that are electric, US market",
    "source": "Source: Bureau of Transportation Statistics"
  },
  "labels": { "density": "endpoints" }
}
```
