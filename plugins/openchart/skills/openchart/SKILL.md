---
name: openchart
description: >
  Generates OpenChart (https://github.com/tryopendata/openchart) chart, table, and graph
  specs from data, and guides editorial design decisions. Use when creating visualizations,
  building charts, rendering data tables, generating VizSpec JSON, creating network graphs,
  answering questions about OpenChart types and encoding rules, or making design decisions
  about chart type selection, color strategy, typography, annotations, and editorial framing.
  Also covers custom D3.js infographics for cases that go beyond declarative specs.
---

# Data Visualization with OpenChart

**Core concept:** Write a VizSpec JSON object, render with `<Chart spec={spec} />` / `<DataTable spec={spec} />` / `<Graph spec={spec} />` (React) or `createChart(container, spec)` / `createTable(container, spec)` / `createGraph(container, spec)` (vanilla JS). The engine validates, compiles, and renders. Specs are plain JSON, no imperative drawing. See https://github.com/tryopendata/openchart for the rendering engine.

**CSS is required.** OpenChart's stylesheet must be loaded for proper rendering (chrome, tables, tooltips, brand watermark). Framework imports handle this automatically, but CDN/standalone HTML needs an explicit `<link>`:

```html
<link rel="stylesheet" href="https://esm.sh/@opendata-ai/openchart-vanilla/styles.css">
```

See [rendering reference](references/rendering.md) for details.

## Chart Selection Decision Tree

```
Single value to highlight    -> Use chrome.title as a big number display
Temporal x-axis column?      -> 1 series: line | 2-5 series: line + color | 6+: filter to top 5
Categorical + numeric?       -> Ranked list: bar (horizontal) | Periodic (Q1, Jan): bar (vertical) | 2-6 composition: arc
Two numeric columns?         -> point (optional size/color for 3rd/4th dims)
Categorical + series + num?  -> stacked bar (use color for series)
Distribution/spread?         -> circle (strip plot)
Nodes + edges / network?     -> graph (force/radial/hierarchical layout)
Flow between stages?         -> sankey (source/target/value)
Tabular data overview?       -> table (with sparklines, heatmaps, bars)
Default                      -> bar
```

## Visualization Types

Each type has a detailed reference with full spec, encoding rules, and examples. Load the reference when you need the details.

| Mark / Type | Best for | Data model | Reference |
| --- | --- | --- | --- |
| `mark: "line"` | Trends over time | x: temporal/ordinal, y: quantitative | [references/line.md](references/line.md) |
| `mark: "area"` | Trends with volume emphasis | x: temporal/ordinal, y: quantitative | [references/area.md](references/area.md) |
| `mark: "bar"` | Rankings (horizontal) or periodic/categorical (vertical) | Orientation inferred from encoding (see below) | [references/bar.md](references/bar.md) |
| `mark: "arc"` | Part-to-whole (2-5 categories) | y: quantitative, color: nominal/ordinal | [references/pie-donut.md](references/pie-donut.md) |
| `mark: "point"` | Correlation between two variables | x: quantitative, y: quantitative | [references/scatter.md](references/scatter.md) |
| `mark: "circle"` | Distribution, strip plots | x: quantitative, y: nominal/ordinal | [references/dot.md](references/dot.md) |
| `mark: "text"` | Text labels positioned by x/y | x: any, y: any, text: nominal | - |
| `mark: "rule"` | Reference lines (horizontal or vertical) | x or y: quantitative/temporal | - |
| `mark: "tick"` | Tick marks for distributions | x: quantitative, y: nominal/ordinal | - |
| `mark: "rect"` | Rectangles for heatmaps | x: ordinal/nominal, y: ordinal/nominal, color: quantitative | - |
| `type: "table"` | Data tables with visual features | columns + data rows | [references/table.md](references/table.md) |
| `type: "graph"` | Networks, relationships, hierarchies | nodes + edges | [references/graph.md](references/graph.md) |
| `type: "sankey"` | Flows between stages/processes | source + target + value | [references/sankey.md](references/sankey.md) |

**Bar orientation:** The engine infers orientation from encoding. `x: nominal/ordinal + y: quantitative` = vertical (column-style). `x: quantitative + y: nominal/ordinal` = horizontal bar. Override with `mark: { type: "bar", orient: "horizontal" | "vertical" }`.

**Arc variants:** `mark: "arc"` renders a pie chart by default. Add `innerRadius > 0` to get a donut: `mark: { type: "arc", innerRadius: 40 }`.

**Collapsed types:** The old `column`, `pie`, `donut`, `scatter`, and `dot` types have been replaced. Use `bar` (orientation inferred), `arc` (with/without innerRadius), `point`, and `circle` respectively.

## Reference Routing

**Always load when generating a new chart spec:** [encoding-channels.md](references/encoding-channels.md), [format-strings.md](references/format-strings.md), [color-strategy.md](references/color-strategy.md)

| When the task involves... | Load |
| --- | --- |
| Adding annotations, callouts, reference lines | [annotations.md](references/annotations.md) |
| onEdit callback, selection, inline editing | [editing.md](references/editing.md) |
| Responsive layout, mobile, breakpoint overrides | [responsive.md](references/responsive.md) |
| Theme customization (colors, fonts, spacing) | [theme.md](references/theme.md) |
| Rendering setup (React, Vue, Svelte, vanilla) | [rendering.md](references/rendering.md) |
| Choosing chart type for a story | [chart-selection.md](references/chart-selection.md) |
| Writing titles, subtitles, annotation text | [editorial-writing.md](references/editorial-writing.md) |
| Font sizing, type hierarchy | [typography.md](references/typography.md) |
| Per-series visual overrides (dashed lines, opacity) | [series-styles.md](references/series-styles.md) |
| Entrance animations, easing, stagger, reduced motion | [animation.md](references/animation.md) |
| Sankey diagram (flows between stages) | [sankey.md](references/sankey.md) |
| Final design quality check | [design-review.md](references/design-review.md) |
| Checking rendered output for defects | [visual-qa.md](references/visual-qa.md) |

**Common reference bundles:**
- **New chart:** encoding-channels + format-strings + color-strategy + editorial-writing + (mark-specific ref)
- **Design polish:** design-review + visual-qa + editorial-writing
- **D3 infographic:** d3-core-patterns + infographic-design + (topic-specific D3 ref)

## Shared Spec Structure

Charts use `mark` as their discriminant. Tables and graphs use `type`.

```typescript
// Charts
{
  mark: string | MarkDef,    // REQUIRED: "line", "bar", "arc", etc. (see Mark section)
  chrome?: { ... },
  theme?: ThemeConfig,
  darkMode?: "auto"|"force"|"off",
  responsive?: boolean,
  animation?: AnimationSpec, // true | AnimationConfig. Off by default. See animation.md
}

// Tables and graphs
{
  type: "table" | "graph",   // REQUIRED: discriminant for non-chart specs
  chrome?: { ... },
  theme?: ThemeConfig,
  darkMode?: "auto"|"force"|"off",
  responsive?: boolean,
  animation?: AnimationSpec, // Tables only. Same API as charts. See animation.md
}
```

**Chrome** (shared by all types):
```typescript
chrome?: {
  title?: string | { text: string, style?: ChromeTextStyle },
  subtitle?: string | { text: string, style?: ChromeTextStyle },
  source?: string | { text: string, style?: ChromeTextStyle },
  byline?: string | { text: string, style?: ChromeTextStyle },
  footer?: string | { text: string, style?: ChromeTextStyle },
}
```

**ChromeTextStyle:** `{ fontSize?: number, fontWeight?: number, fontFamily?: string, color?: string }`

All chrome text fields support `\n` for explicit line breaks. Text also auto-wraps at the container width.

## Mark (Charts Only)

`mark` can be a string or an object with additional properties:

```typescript
// Simple string form
mark: "line" | "area" | "bar" | "arc" | "point" | "circle" | "text" | "rule" | "tick" | "rect"

// Object form with additional config
mark: {
  type: MarkType,                        // REQUIRED: same values as the string form
  point?: boolean | "transparent",       // show point markers on line/area marks
  interpolate?: "linear" | "monotone" | "step" | "step-before" | "step-after" | "basis" | "cardinal" | "natural",
  orient?: "horizontal" | "vertical",   // override inferred bar orientation
  innerRadius?: number,                  // >0 makes arc marks render as donuts
}
```

**Examples:**
```json
{ "mark": "bar" }
{ "mark": { "type": "line", "point": true, "interpolate": "monotone" } }
{ "mark": { "type": "arc", "innerRadius": 40 } }
{ "mark": { "type": "bar", "orient": "horizontal" } }
```

**Mark type determines default entrance animation.** When `animation: true` is set, each mark type gets an appropriate animation: bars use clip-path reveal from the baseline, lines draw progressively, areas draw + fade, arcs scale from center, and points pop in with scale + fade. Text, rules, and ticks fade in. See [animation reference](references/animation.md) for per-mark defaults and overrides.

## Encoding Channels (Charts Only)

Charts map data to visuals via encoding channels: x, y, color, size, detail, opacity, shape, strokeDash, text, tooltip, x2, y2, theta, radius. See [encoding channels reference](references/encoding-channels.md) for the full channel list and conditional encoding examples.

**EncodingChannel:**
```typescript
{
  field: string,             // REQUIRED: column name in data
  type: FieldType,           // REQUIRED: "quantitative"|"temporal"|"nominal"|"ordinal"
  aggregate?: AggregateOp,   // "count"|"sum"|"mean"|"median"|"min"|"max"
  axis?: {
    title?: string,          // axis title (default: field name). Deprecated alias: `label`
    format?: string,         // d3-format string, e.g. ",.0f" "$,.2f" ".1%"
    tickCount?: number,      // override tick count
    grid?: boolean,          // show gridlines
    labelAngle?: number,     // tick label rotation in degrees. Deprecated alias: `tickAngle`
  },
  scale?: {
    domain?: [number, number] | string[],  // explicit domain
    type?: "linear"|"log"|"time"|"band"|"point"|"ordinal"|"utc"|"pow"|"sqrt"|"symlog"|"quantile"|"quantize"|"threshold",
    nice?: boolean,          // clean tick values (default: true)
    zero?: boolean,          // include zero (default: true for quantitative)
  },
  condition?: { test: FilterPredicate, value: any },  // conditional encoding
  value?: any,               // fallback when condition test is false
}
```

## Data Transforms (Charts Only)

Apply transforms (filter, bin, calculate, timeUnit) to data before encoding. Transforms run in array order. See [data transforms reference](references/data-transforms.md).

## Layer Composition (Charts Only)

Overlay multiple marks in a single chart using `layer`. Each layer is a standalone spec with its own mark, data, and encoding.

```json
{
  "layer": [
    { "mark": "bar", "data": [...], "encoding": { ... } },
    { "mark": "line", "data": [...], "encoding": { ... } }
  ]
}
```

Layers share the same coordinate space. Use this for combo charts (bar + line), adding reference lines, or overlaying annotations.

## Legend Configuration (Charts Only)

```typescript
legend?: {
  position?: "top"|"right"|"bottom"|"bottom-right"|"inline",
  show?: boolean,     // default: true. Set false to hide legend entirely.
  columns?: number,   // columns for horizontal layout (controls row wrapping)
  symbolLimit?: number, // max entries before "+N more" truncation
  maxRows?: number,   // max rows for top-positioned legends (default: 2)
}
```

Position is responsive by default (the engine picks based on container width). Set explicitly to override. Use `show: false` to hide the legend when it's redundant (e.g., bar charts where the y-axis already labels each category).

## Label Density (Charts Only)

```typescript
labels?: {
  density?: "all"|"auto"|"endpoints"|"none",  // default: "auto"
  format?: string,           // d3-format for label values (supports literal suffix)
  prefix?: string,           // literal string prepended to each formatted label value
}
```

| Mode | Behavior | Use when |
| --- | --- | --- |
| `auto` | Show labels with collision detection | Default, most charts |
| `all` | Show every label, no collision detection | Few data points, precise values matter |
| `endpoints` | First and last per series only | Line charts, emphasize start/end |
| `none` | No labels (tooltips + legend only) | Dense data, clean look |

## Format Strings

Both `axis.format` and `labels.format` accept [d3-format](https://d3js.org/d3-format) strings plus a literal suffix extension. See [format strings reference](references/format-strings.md) for the full table.

| Format | Output example | Use case |
| --- | --- | --- |
| `".1f%"` | `12.5%` | Percentage (data already in %, not 0-1) |
| `"$,.0f"` | `$1,234` | Currency with commas |
| `"~s"` | `10k`, `1.5M` | SI suffix for large numbers |
| `",.0f"` | `132,979` | Comma-separated, no decimals |
| `".1%"` | `12.5%` (from 0.125) | d3 native percent (multiplies by 100) |

**Critical:** When data is already in percentage form (12.5 meaning 12.5%), use `".1f%"` not `".1%"`. The d3 `%` type multiplies by 100, so `12.5` becomes `1,250.0%`.

## Per-Series Styling (Charts Only)

Override visual properties (lineStyle, showPoints, strokeWidth, opacity) for individual series by name via `seriesStyles`. See [series styles reference](references/series-styles.md).

## Data Resolution

**Keep data arrays under 150 rows per series.** More data doesn't make a better chart - it makes a slower, harder-to-read one. Reduce resolution before building the spec, not after.

| Time span | Resolution | Typical rows | Example |
| --- | --- | --- | --- |
| < 1 year | Daily or weekly | 50-200 | Stock price last 6 months |
| 1-5 years | Monthly | 12-60 | Unemployment rate 2020-2025 |
| 5-25 years | Quarterly or annual | 20-100 | GDP since 2000 |
| 25-100+ years | Annual or decade | 25-100 | CO2 emissions since 1900 |

**How to reduce:** When querying APIs, aggregate before passing data to the spec:
- Use `group_by=year` with `aggregate=avg(value)` to go from monthly to annual
- Sample every Nth row for evenly-spaced data
- Filter to the time range that matters (don't chart 75 years when the story is about the last 10)

**Multi-series charts are multiplicative.** A 3-series line chart with 300 points per series = 900 data rows. Reduce each series to ~50-80 points for a clean result. For the same 25-year span, annual data (25 points x 3 series = 75 rows) reads better than monthly (300 x 3 = 900 rows).

**Why this matters beyond readability:** Large data arrays inflate the spec JSON, slow rendering, and generate massive accessibility tables in the DOM. A 900-row chart produces a ~19,000px tall screen-reader table that can break page layout if styles don't load correctly.

## First Draft Checklist

Run these checks before outputting a spec. These catch the issues that most often require iteration after rendering.

| Check | What to verify |
| --- | --- |
| **Data resolution is appropriate** | Check total rows in the data array. Over 150 per series? Aggregate to a coarser time grain or sample. A 25-year time series should use annual or quarterly data, not monthly. See Data Resolution table. |
| **Color encodes the story** | If one variable drives the narrative, color should reinforce it. Use the decision table in [color-strategy.md](references/color-strategy.md) to pick the right strategy and `theme.colors` array. Don't leave a scatter plot monochrome when a gradient would make the pattern obvious. |
| **Y-domain fits the data** | Domain ceiling should be ~5-10% above the highest data value. `[0, 55]` for data peaking at 48.8 wastes space. Use `[0, 52]`. **For bar/column charts with narrow data ranges** (e.g., values between 200 and 280), don't default the floor to 0 - it makes variations invisible. Set the domain floor near the minimum value. Exception: charts where zero is a meaningful baseline (percent change from 0, counts). |
| **Annotations clear of data AND each other** | The engine auto-resolves annotation-to-annotation collisions, but start with good separation for cleaner results. Prefer 0-2 text annotations; use reflines for additional callouts. On scatter/bubble, use 40-100px offsets into empty quadrants with connectors. When using 2+ text annotations, verify with `playwright-cli`. |
| **Subtitle is intentional about wrapping** | Unintentional wrapping with orphaned fragments looks broken. Abbreviate, restructure, or use `\n` for explicit line breaks. Use shorthand keys in the subtitle (e.g., `"LI = low-income"`) rather than spelling everything out. |
| **Endpoint labels won't eat the chart** | `labels: { density: "endpoints" }` with series names > 15 chars reserves a huge right margin. Use `"none"` + `legend: { position: "top" }` instead, or abbreviate series names. |
| **Axis ticks show units** | Percentages should show `10%` not `10`. Use `format: ".0f%"` when data is already in percent form (e.g., 10 meaning 10%). Use `format: ".0%"` only when data is in decimal form (0.10 meaning 10%). Large numbers should use SI suffixes: `format: "~s"` turns 10000 into `10k` and 1000000 into `1M`. For currency: `format: "$~s"` gives `$10k`, `$1M`. See the Format Strings table above. |
| **Consistent color palette across related charts** | If multiple charts in the same article cover the same dimension (e.g., poverty), use the same color mapping (blue = low, red = high) so the reader builds a mental model. |

## Spec Anti-Patterns

| Mistake | Fix |
| --- | --- |
| Using `nominal` for numeric field | Use `quantitative` for numbers, `temporal` for dates |
| Using `ordinal` for temporal data | Use `temporal`; ordinal is for ordered categories |
| Too many data points (>150 per series) | Aggregate or sample before building the spec. See Data Resolution table above. Monthly data over 25 years = 300 rows per series, use annual instead |
| Forgetting encoding.color for multi-series | Line/bar with groups needs `color` channel |
| Bar chart for time series | Use line for temporal data; bar with vertical orientation for periodic categories |
| Using chart mark for network data | Use `type: "graph"` with nodes + edges |
| Not specifying axis format for currency/pct | Add `axis: { format: "$,.0f" }` or `".1f%"` |
| Using `".1%"` when data is already in percent form | `".1%"` multiplies by 100 (d3 convention). If data is `12.5` meaning 12.5%, use `".1f%"` (literal suffix) |
| Axis format and label format inconsistent | Set both `axis.format` and `labels.format` to the same pattern so ticks and data labels match |
| Using `darkMode: "auto"` in class-based dark mode apps | `"auto"` checks `prefers-color-scheme` only. For class-based toggles (Astro, Next.js), observe DOM and map to `"force"`/`"off"`. See [rendering](references/rendering.md) |
| Temporal scale with `nice: true` (default) creating dead space | `nice` rounds the domain outward (e.g., `2010-01` becomes `2008`). Set `scale: { domain: ["2010-01", "2026-01"], nice: false }` for precise control |
| Using `type` instead of `mark` for charts | Charts use `mark: "line"` (not `type: "line"`). Only tables and graphs use `type`. |

For design anti-patterns (titles, color, annotations), see [design review](references/design-review.md).

## Known Gotchas

Rendering and component behaviors that aren't obvious from the spec alone.

| Gotcha | Behavior | Fix |
| --- | --- | --- |
| Refline labels only support top/bottom | `labelAnchor` on refline annotations only accepts `"top"` or `"bottom"`. Left/right values are accepted in the type but have no visible effect on reflines (they do work on range annotations). | Set `label: ""` on the refline and add a separate `type: "text"` annotation positioned where you want a side label. |
| Endpoint labels don't wrap | `\n` in series names does NOT create line breaks in endpoint labels. Long names consume chart width. | Shorten series names in the data instead. |
| DataTable CSS overrides unreliable | Custom CSS targeting `.oc-table-wrapper td` may not apply due to CSS specificity. | Use the DataTable `style` prop for inline overrides: `<DataTable style={{ paddingLeft: 10 }} spec={...} />`. |
| Scatter plots auto-set `zero: false` | Unlike other chart types, scatter/point marks automatically set `scale.zero: false` on both axes if not explicitly configured. This means scatter domains fit tightly to data. | To include zero, explicitly set `scale: { zero: true }` on the relevant axis. Be aware that scatter and bar/line charts handle zero differently by default. |

## Custom D3.js Infographics

When a visualization goes beyond what declarative specs can handle (creative metaphors, unusual layouts, treemaps, generative art, scrollytelling), fall back to raw D3.js + SVG. Note: sankey is now a first-class type with its own spec format. See [references/sankey.md](references/sankey.md). Use the D3 reference only for heavily customized sankey layouts. These references cover D3 implementation patterns:

| Topic | Reference |
| --- | --- |
| D3 selections, scales, axes, margin convention | [references/d3/d3-core-patterns.md](references/d3/d3-core-patterns.md) |
| Tufte principles, storytelling, publication design | [references/d3/infographic-design.md](references/d3/infographic-design.md) |
| Path morphing, generative art, creative coding | [references/d3/advanced-techniques.md](references/d3/advanced-techniques.md) |
| D3 transitions, scroll effects, easing, timing | [references/d3/animation-transitions.md](references/d3/animation-transitions.md) |
| Treemap, sunburst, sankey diagrams | [references/d3/chart-hierarchy.md](references/d3/chart-hierarchy.md) |
| D3 color APIs, programmatic palette generation, contrast checking | [references/d3/color-palettes.md](references/d3/color-palettes.md) |
| SVG text wrapping, collision detection, leader lines, ARIA | [references/d3/typography-labels.md](references/d3/typography-labels.md) |
| viewBox, ResizeObserver, responsive SVG patterns | [references/d3/responsive-svg.md](references/d3/responsive-svg.md) |
