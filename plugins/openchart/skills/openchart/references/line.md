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

## Spec shape

For the full `MarkDef` for `mark: "line"` (`point`, `interpolate`, `stroke`, `strokeWidth`, `opacity`) and the full encoding surface, load `MarkDef`, `Encoding`, and `ChartSpec` from `index.d.ts`. Behavioral notes specific to line follow.

**Tip (single series):** Use `labels: { density: "endpoints" }` to show only the first and last value per series.

**Multi-series default — endpoint chip column:** For ≥2-series line/area charts, the engine renders a column of chip+swatch labels at the trailing edge of each line by default (rounded pill with a colored bar swatch + name + last value), and auto-suppresses the traditional legend. Disable with `endpointLabels: false`. See the *Endpoint Labels (Multi-Series Line/Area)* section in SKILL.md for the full suppression truth table.

**Caution with `density: "endpoints"` and long series names (legacy end-of-line labels):** When `endpointLabels: false` and `legend: { show: false }`, the engine falls back to the legacy end-of-line labels — these include the series name and reserve a large right margin for long names. If series names are >15 chars, either abbreviate them or use the chip column instead.

**Runtime legend toggle:** Clicking a legend entry hides/shows that series via engine recompile. The y-axis rebalances, the color scale stays locked (other lines keep their original colors), and per-series UI hides — endpoint chip, marker, dot annotation, and any text annotation anchored to that series. The last visible series can't be hidden.

## Crosshair

Show a vertical line that snaps to the nearest data point on hover:

```json
{ "crosshair": true }
```

Renders a dashed vertical line at the hovered data point's x-coordinate. Only applies to line and area charts. Off by default.

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
