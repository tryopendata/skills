# Pie & Donut Charts

Part-to-whole composition. Both use `mark: "arc"`. Donut is preferred over pie (center space can show total).

- **Pie:** `mark: "arc"` (no innerRadius, or `innerRadius: 0`)
- **Donut:** `mark: { type: "arc", innerRadius: 40 }`

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

## Spec shape

For the full `MarkDef` for `mark: "arc"` and the encoding surface, load `MarkDef`, `Encoding`, and `ChartSpec` from `index.d.ts`. The arc-specific field worth knowing is `innerRadius` — `0` (default) is a pie; `> 0` is a donut.

**Half-donut (election-style results):** restrict the sweep with `startAngle`/`endAngle` in radians. `mark: { type: "arc", innerRadius: 0.55, startAngle: -Math.PI/2, endAngle: Math.PI/2 }` gives a Datawrapper-style semicircle. The engine resizes a partial sweep to fill the chart area, so the half-donut renders at full size rather than half. (For a seat-dot hemicycle rather than proportional wedges, use `mark: "parliament"` instead.)

## Builder

```typescript
import { pieChart } from "@opendata-ai/openchart-core";

// pieChart(data, category, value) - category maps to color, value maps to y
// For a pie: pieChart produces mark: "arc"
// For a donut: pieChart(data, cat, val, { innerRadius: 40 }) produces mark: { type: "arc", innerRadius: 40 }
const spec = pieChart(data, "source", "share", {
  chrome: { title: "Energy mix breakdown" },
});
```

## Example

```json
{
  "mark": { "type": "arc", "innerRadius": 40 },
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

## Waffle: part-to-whole as counts

When the story is "how many out of 100" (concrete counts, not just an angle), reach for `mark: "waffle"` instead of arc. It draws a unit grid where each cell is a fraction of the whole.

- **Encoding:** `color` is the category (required); the quantitative share goes on `y` (aliased as `theta`, the same part-to-whole channel arc uses). No positional axes.
- **`units`** (default 100) sets the total cell count -- the "x of 100" framing. **`columns`** (default 10) sets grid width; rows derive from `units / columns` (so the default is a 10x10 square).
- Shares normalize to `units` via largest-remainder rounding, so cells always sum exactly. A small nonzero share can round to **0 cells** -- there's deliberately no minimum-one-cell floor -- but the category still appears in the legend.
- Use `color.highlight` to single out one category against grayed cells.
