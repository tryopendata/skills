# Area Chart

Trends with volume emphasis. Filled region below the line.

## Encoding Rules

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | Yes | temporal, ordinal |
| y | Yes | quantitative |
| y2 | No | quantitative -- second boundary for confidence bands/ranges |
| color | No | nominal, ordinal |
| size | No | quantitative |
| detail | No | nominal |
| opacity | No | quantitative |
| strokeDash | No | nominal, ordinal |
| tooltip | No | any |

Same encoding as line charts. Use area when you want to emphasize the magnitude (volume under the curve), not just the trend direction. Add `y2` for confidence intervals or range bands (fill between two data series).

## Stacking (Multi-Series)

**Multi-series area charts default to overlap, not stacked.** Each series renders as a semi-transparent fill drawn over the others. This is the opposite of bars (which default to stacked when colored).

| `encoding.y.stack` | Result |
|---|---|
| `undefined` (default) / `false` / `null` | Overlap. Each series fills from baseline to its own value, with reduced opacity so layers show through. Use for absolute values where comparing magnitudes per series matters. |
| `"zero"` / `true` | Stacked from zero. Use for composition / share-over-time. |
| `"normalize"` | Percentage stack. Domain becomes [0, 1]; each x sums to 100%. Use for share-of-total trends. |
| `"center"` | Streamgraph (centered around a baseline). Use for aesthetic flow visualization where exact values are secondary. |

**Pick by story:** "How did each series move?" -> overlap. "How did the mix shift?" -> `"zero"` or `"normalize"`.

## Crosshair

Show a vertical line that snaps to the nearest data point on hover:

```json
{ "crosshair": true }
```

Renders a dashed vertical line at the hovered data point's x-coordinate. Only applies to line and area charts. Off by default.

## Spec

```typescript
{
  mark: "area" | {
    type: "area",
    interpolate?: "linear"|"monotone"|"step"|"step-before"|"step-after"|"basis"|"cardinal"|"natural",
    fill?: string | GradientDef,   // solid color or gradient fill (see gradients.md)
    stroke?: string,               // top-line stroke color
    strokeWidth?: number,
    point?: boolean | "transparent",  // show point markers on each data point
    opacity?: number,
  },
  data: DataRow[],
  encoding: {
    x: { field: string, type: "temporal"|"ordinal", axis?, scale? },
    y: { field: string, type: "quantitative", axis?, scale? },
    y2?: { field: string, type: "quantitative" },  // upper/lower band boundary
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

## Confidence Band / Range Area

Use `y2` encoding to fill between two data fields. The area fill spans from `y` to `y2`:

```json
{
  "mark": { "type": "area", "opacity": 0.25 },
  "data": [
    { "month": "Jan", "low": 52, "high": 68 },
    { "month": "Feb", "low": 54, "high": 71 },
    { "month": "Mar", "low": 58, "high": 76 }
  ],
  "encoding": {
    "x": { "field": "month", "type": "ordinal" },
    "y": { "field": "low", "type": "quantitative" },
    "y2": { "field": "high", "type": "quantitative" }
  }
}
```

## Gradient Fill

Use `mark.fill` with a `GradientDef` for a top-to-bottom fade. When a gradient is provided, the area's `fillOpacity` defaults to 1 and the gradient's own `stop.opacity` values control the fade:

```json
{
  "mark": {
    "type": "area",
    "stroke": "#E07B39",
    "strokeWidth": 2,
    "fill": {
      "gradient": "linear",
      "x1": 0, "y1": 0, "x2": 0, "y2": 1,
      "stops": [
        { "offset": 0, "color": "#E07B39", "opacity": 1.0 },
        { "offset": 1, "color": "#E07B39", "opacity": 0.2 }
      ]
    }
  }
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
