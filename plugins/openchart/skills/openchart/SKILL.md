---
name: openchart
description: >
  Generates OpenChart (https://github.com/tryopendata/openchart) chart, table, graph, sankey,
  and tilemap specs from data, and guides editorial design decisions. Use when creating visualizations,
  building charts, rendering data tables, generating VizSpec JSON, creating network graphs,
  building sankey/flow diagrams, building US state tile grid maps, answering questions about OpenChart
  types and encoding rules, or making design decisions about chart type selection, color strategy,
  typography, annotations, and editorial framing. Also covers custom D3.js infographics for cases
  beyond declarative specs.
---

# Data Visualization with OpenChart

<!-- Remove migration note after 2026-07 -->
> **Note:** SVG design capabilities (logos, icons, graphics) have moved to the `opendesign` plugin. Install with `/plugin install opendesign@opendata-skills`.

## Source of truth: load the types first

**Before authoring or modifying a spec, load the OpenChart type definitions.** They are the canonical source for field shapes, enums, and defaults. This skill complements the types — it does not replace them. When this skill and the types disagree, the types win.

Try these locations in order; stop at the first one that resolves:

1. **Installed package** (most common): `node_modules/@opendata-ai/openchart-core/dist/index.d.ts`. The full chart, table, graph, sankey, and tilemap spec surface is rolled into this single bundled `.d.ts`.
2. **Source repo** (if you're working in the openchart monorepo): `packages/core/src/types/spec.ts` and `packages/core/src/types/layout.ts`. JSDoc comments here are richer than the bundled `.d.ts`.
3. **Published CDN** (no local install, network available): `https://unpkg.com/@opendata-ai/openchart-core/dist/index.d.ts`.
4. **Fallback:** if none of the above are reachable, use the type sketches in this skill and **flag the uncertainty** in your response so the user knows you authored without canonical types.

The names worth grepping for once you have a types file open: `ChartSpec`, `TableSpec`, `GraphSpec`, `SankeySpec`, `TileMapSpec`, `Encoding`, `EncodingChannel`, `MarkDef`, `Chrome`, `Metric`, `EndpointLabelsConfig`, `Annotation` (union), `TextAnnotation`, `RangeAnnotation`, `RuleAnnotation`, `LegendConfig`, `LabelSpec`, `SeriesStyle`, `AnimationSpec`, `ThemeConfig`.

## What this skill carries that the types don't

The types tell you the shape of a valid spec. This skill carries the things types can't express:

- **Behavioral defaults that live in the normalizer/compiler**, not in the type signature. (E.g. area defaults to overlap, not stacked. Bar defaults to stacked when colored.)
- **Cross-field interactions and truth tables.** (E.g. how `legend.show`, `endpointLabels`, and the legacy end-of-line labels interact.)
- **Runtime semantics.** (E.g. legend toggle recompiles, rebalances the y-axis, locks the color scale, hides per-series UI, refuses to hide the last visible series.)
- **Editorial judgment.** (E.g. when to pick a bar vs a line, color strategy, annotation density, data-resolution budgets.)
- **Worked examples** for common stories.
- **Failure modes and gotchas** that you'd only learn after rendering and looking at the result.

If you find yourself restating a field shape this skill already documents, prefer the types — they're authoritative and won't drift.

**Core concept:** Write a VizSpec JSON object, render with `<Chart>` / `<DataTable>` / `<Graph>` / `<Sankey>` / `<TileMap>` (React/Vue/Svelte) or `createChart()` / `createTable()` / `createGraph()` / `createSankey()` / `createTileMap()` (vanilla JS). The engine validates, compiles, and renders. Specs are plain JSON, no imperative drawing. See https://github.com/tryopendata/openchart for the rendering engine.

**CSS is required.** OpenChart's stylesheet must be loaded for proper rendering (chrome, tables, tooltips, brand watermark). Framework imports handle this automatically, but CDN/standalone HTML needs an explicit `<link>`:

```html
<link rel="stylesheet" href="https://esm.sh/@opendata-ai/openchart-vanilla/styles.css">
```

See [rendering reference](references/rendering.md) for details.

## Chart Selection Decision Tree

```
Single KPI / KPI row         -> spec.metrics: Metric[] (label+value+delta, rendered between subtitle and chart)
Temporal x-axis column?      -> 1 series: line | 2-5 series: line + color | 6+: filter to top 5
Categorical + numeric?       -> Ranked list: bar (horizontal) | Periodic (Q1, Jan): bar (vertical) | 2-6 composition: arc
Two numeric columns?         -> point (optional size/color for 3rd/4th dims)
Categorical + series + num?  -> stacked bar (use color for series)
Distribution/spread?         -> circle (strip plot)
Nodes + edges / network?     -> graph (force/radial/hierarchical layout)
Flow between stages?         -> sankey (source/target/value)
US state-level data?         -> tilemap (state codes + values, equal-weight grid)
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
| `type: "tilemap"` | US state-level data (equal-weight grid) | state codes + values | [references/tilemap.md](references/tilemap.md) |

**Bar orientation:** The engine infers orientation from encoding. `x: nominal/ordinal + y: quantitative` = vertical (column-style). `x: quantitative + y: nominal/ordinal` = horizontal bar. Override with `mark: { type: "bar", orient: "horizontal" | "vertical" }`.

**Arc variants:** `mark: "arc"` renders a pie chart by default. Add `innerRadius > 0` to get a donut: `mark: { type: "arc", innerRadius: 40 }`.

**Collapsed mark types:** These mark aliases no longer exist as separate values. Use the canonical marks instead:

| Old mark | Use instead |
| --- | --- |
| `"column"` | `"bar"` (engine infers vertical from encoding) |
| `"pie"` | `"arc"` |
| `"donut"` | `{ type: "arc", innerRadius: 40 }` |
| `"scatter"` | `"point"` |
| `"dot"` | `"circle"` |

## Reference Routing

**Always load when generating a new chart spec:** [encoding-channels.md](references/encoding-channels.md), [format-strings.md](references/format-strings.md), [color-strategy.md](references/color-strategy.md)

| When the task involves... | Load |
| --- | --- |
| Dual-axis charts, independent y-scales | See Layer Composition section above -- no extra reference needed |
| Adding annotations, callouts, reference lines | [annotations.md](references/annotations.md) |
| onEdit callback, selection, inline editing | [editing.md](references/editing.md) |
| Responsive layout, mobile, breakpoint overrides | [responsive.md](references/responsive.md) |
| Theme customization (colors, fonts, spacing) | [theme.md](references/theme.md) |
| Data transforms (window, filter, aggregate, etc.) | [data-transforms.md](references/data-transforms.md) |
| Rendering setup (React, Vue, Svelte, vanilla) | [rendering.md](references/rendering.md) |
| Choosing chart type for a story | [chart-selection.md](references/chart-selection.md) |
| Writing titles, subtitles, annotation text | [editorial-writing.md](references/editorial-writing.md) |
| Font sizing, type hierarchy | [typography.md](references/typography.md) |
| Per-series visual overrides (dashed lines, opacity) | [series-styles.md](references/series-styles.md) |
| Gradient fills (linear, radial, per-mark) | [gradients.md](references/gradients.md) |
| Entrance animations, easing, stagger, reduced motion | [animation.md](references/animation.md) |
| Sankey diagram (flows between stages) | [sankey.md](references/sankey.md) |
| US state tile grid map | [tilemap.md](references/tilemap.md) |
| Final design quality check | [design-review.md](references/design-review.md) |
| Checking rendered output for defects | [visual-qa.md](references/visual-qa.md) |

**Common reference bundles:**
- **New chart:** encoding-channels + format-strings + color-strategy + editorial-writing + (mark-specific ref)
- **Design polish:** design-review + visual-qa + editorial-writing
- **D3 infographic:** d3-core-patterns + infographic-design + (topic-specific D3 ref)

## Spec discriminant

- **Charts** use `mark` (`"line"`, `"bar"`, `"area"`, `"arc"`, `"point"`, `"circle"`, `"text"`, `"rule"`, `"tick"`, `"rect"`).
- **Tables, graphs, sankey, tilemap** use `type` (`"table"` | `"graph"` | `"sankey"` | `"tilemap"`).

For the full top-level shape — every optional field, exact enum values, defaults — load `ChartSpec`, `TableSpec`, `GraphSpec`, `SankeySpec`, `TileMapSpec` from `index.d.ts` (see "Source of truth" above).

**Behavior worth knowing that the types don't tell you:**

- `animation` is **off by default**. Set `true` for sensible per-mark entrance defaults; pass an `AnimationConfig` for per-phase control. See [animation.md](references/animation.md).
- `crosshair` is **off by default** and only renders on line/area charts.
- `endpointLabels` is **auto-on for ≥2-series line/area** and auto-suppresses the traditional legend in that case. See the "Endpoint Labels" section below for the full suppression truth table.
- `hiddenSeries` on the spec hides series on first render. The vanilla adapter also maintains a separate runtime hidden set populated by legend clicks; that triggers full engine recompile (y-axis rebalance, locked color scale, per-series UI hide). See "Legend Toggle (Runtime)" below.
- `display: 'sparkline'` strips chrome, axes, legend, watermark, animation, and crosshair for inline KPI-card use. Explicit per-field overrides still win (set `chrome.title` and you'll still get a title in sparkline mode).

**Chrome elements** (`chrome.eyebrow` / `title` / `subtitle` / `source` / `byline` / `footer` / `brand`): each takes `string | ChromeText`. The `eyebrow` is a tracked, accent-tinted kicker above the title. The `brand` is a right-anchored block on the footer row paired with a small accent dot — setting it suppresses the default `tryOpenData.ai` watermark. All chrome text supports `\n` for explicit line breaks and auto-wraps at the container width. Exact field shape: see `Chrome` and `ChromeText` in `index.d.ts`.

**KPI metric row** (charts only): `metrics: Metric[]` at the top level (not inside `chrome`) renders a horizontal row of label+value cells between the subtitle and the chart area. Each cell can carry a delta and a secondary value. Auto-stripped in sparkline mode and at narrow/short containers, or when value text would overflow its cell. Exact field shape: see `Metric` in `index.d.ts`.

## Mark (Charts Only)

`mark` is either a string (`"line"`, `"area"`, `"bar"`, `"arc"`, `"point"`, `"circle"`, `"text"`, `"rule"`, `"tick"`, `"rect"`) or an object — see `MarkDef` in `index.d.ts` for the full field set (`type`, `point`, `interpolate`, `orient`, `innerRadius`, `fill`, `stroke`, `strokeWidth`, `opacity`, etc.).

**Behavior the types don't tell you:**

- **Bar orientation is inferred from encoding types.** `x: nominal/ordinal/temporal + y: quantitative` = vertical column. `x: quantitative + y: nominal/ordinal` = horizontal bar. Override with `mark: { type: "bar", orient: "horizontal" | "vertical" }` only when the inference is wrong.
- **`mark: "arc"` is a pie by default.** Pass `innerRadius > 0` to get a donut.
- **`mark.fill` accepts a `GradientDef`** for linear or radial gradients. Gradients can also appear as conditional color values in `encoding.color`. See [gradients.md](references/gradients.md).
- **Default entrance animation depends on mark type** when `animation: true`. Bars clip-reveal from the baseline; lines draw progressively; areas draw + fade; arcs scale from center; points pop in with scale + fade; text/rules/ticks fade in. See [animation.md](references/animation.md) for per-mark defaults and per-phase overrides.

**Collapsed mark aliases (don't use these):**

| Old mark | Use instead |
| --- | --- |
| `"column"` | `"bar"` (engine infers vertical from encoding) |
| `"pie"` | `"arc"` |
| `"donut"` | `{ type: "arc", innerRadius: 40 }` |
| `"scatter"` | `"point"` |
| `"dot"` | `"circle"` |

**Examples:**
```json
{ "mark": { "type": "line", "point": true, "interpolate": "monotone" } }
{ "mark": { "type": "arc", "innerRadius": 40 } }
{ "mark": { "type": "bar", "fill": { "gradient": "linear", "stops": [{"offset": 0, "color": "#1b7fa3"}, {"offset": 1, "color": "#1b7fa3", "opacity": 0.4}] } } }
```

## Encoding Channels (Charts Only)

Charts map data to visuals via encoding channels: `x`, `y`, `color`, `size`, `detail`, `opacity`, `shape`, `strokeDash`, `text`, `tooltip`, `x2`, `y2`, `theta`, `radius`, `order`. Each channel is an `EncodingChannel` (`field`, `type`, `aggregate`, `axis`, `scale`, `bin`, `timeUnit`, `sort`, `format`, `title`, `stack`, `condition`, `value`). For the full shape and enum values: load `Encoding` and `EncodingChannel` from `index.d.ts`.

**Behavior the types don't tell you:**

- `field` + `type` is the minimum for any channel. `type` must be one of `"quantitative" | "temporal" | "nominal" | "ordinal"` — picking the wrong one is the most common spec authoring bug (see Spec Anti-Patterns below).
- `axis.format` is **d3-format with a literal-suffix extension** (e.g. `".1f%"`). The literal suffix is OpenChart's add-on; native d3 doesn't support it. See [format-strings.md](references/format-strings.md) and the Format Strings section below for the percent-form pitfall.
- `scale.nice: true` is the default for quantitative and temporal scales — it rounds the domain outward to clean tick values. On temporal scales this can shift the domain by years; set `nice: false` when you need precise control.
- `stack` defaults differ by mark: bar = stacked when colored, area = overlap when colored, line = n/a. See the per-mark default table in [encoding-channels.md](references/encoding-channels.md).
- See [encoding-channels.md](references/encoding-channels.md) for conditional encoding examples and the full per-channel guide.

## Data Transforms (Charts Only)

Apply transforms (filter, bin, calculate, timeUnit, aggregate, fold, window) to data before encoding. Transforms run in array order. Filters support data-relative time references for temporal fields. See [data transforms reference](references/data-transforms.md).

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

### Dual-Axis Charts (Independent Y-Scales)

When two series have incompatible value ranges (e.g., revenue in millions vs. headcount in thousands), use `resolve: { scale: { y: "independent" } }` on the layer spec. Layer 0 gets the left y-axis; layer 1 gets the right y-axis. Both share the x-axis.

```json
{
  "resolve": { "scale": { "y": "independent" } },
  "layer": [
    {
      "mark": { "type": "bar", "opacity": 0.85 },
      "data": [
        { "year": "2022", "revenue": 8000000 },
        { "year": "2023", "revenue": -5000000 }
      ],
      "encoding": {
        "x": { "field": "year", "type": "ordinal" },
        "y": {
          "field": "revenue",
          "type": "quantitative",
          "axis": { "title": "Net Revenue ($)", "format": "~s", "labelColor": "#3E7CB1" }
        }
      },
      "labels": { "density": "none" }
    },
    {
      "mark": { "type": "line", "stroke": "#E07B39", "strokeWidth": 2.5, "point": true, "interpolate": "monotone" },
      "data": [
        { "year": "2022", "enrollment": 52800 },
        { "year": "2023", "enrollment": 51600 }
      ],
      "encoding": {
        "x": { "field": "year", "type": "ordinal" },
        "y": {
          "field": "enrollment",
          "type": "quantitative",
          "axis": { "title": "Enrollment", "format": "~s", "labelColor": "#E07B39" }
        }
      },
      "labels": { "density": "none" }
    }
  ]
}
```

**Dual-axis rules:**
- Max 2 layers -- there are only left and right y-axes.
- Both layers must have compatible x-field types (both `ordinal`, both `temporal`, etc.).
- Use `axis.labelColor` on each layer's y-encoding to color the axis labels to match the series -- this is the standard dual-axis pattern (Datawrapper, Highcharts style).
- Use `labels: { density: "none" }` on both layers to avoid label collisions between the two series.
- The engine zero-aligns both y-scales so zero sits at the same pixel height on both axes. Annotations target the primary (left) y-scale.
- Works with any combination: bar + line, bar + area, area + line.

## Legend Configuration (Charts Only)

`legend` accepts `position`, `show`, `columns`, `symbolLimit`, `maxRows`, `offset`, `exclude`. Full shape: `LegendConfig` in `index.d.ts`.

**Behavior:** Position is responsive by default — the engine picks `top` / `right` / `bottom` / `bottom-right` / `inline` based on container width. Set `position` to override. Use `show: false` when the legend is redundant (e.g. bar charts where the y-axis already labels each category). For multi-series line/area, leaving `show` unset triggers auto-suppression in favor of the endpoint chip column — see the truth table below.

## Endpoint Labels (Multi-Series Line/Area)

For multi-series line/area charts, the engine renders a column of **chip+swatch labels** at the trailing edge of each series (rounded pill with a colored bar swatch + label + last value). Auto-on for ≥2 series; off for single-series and non-line/area.

`endpointLabels` accepts `boolean | EndpointLabelsConfig` (`show`, `valueField`, `format`, `width`, `showMarker`, `showLeader`, `markerStyle`). Full shape: `EndpointLabelsConfig` in `index.d.ts`.

**Suppression truth table** (≥2-series line/area). The traditional legend, the endpoint column, and the legacy end-of-line labels are three knobs that interact. The implementation lives in `packages/engine/src/legend/suppression.ts` and is the single source of truth — if this table ever drifts, the engine wins.

| `legend.show` | `endpointLabels` | Traditional legend | Endpoint column | End-of-line labels |
|---|---|---|---|---|
| unset | unset | hidden (auto-suppressed) | shown (default) | hidden |
| `true` | unset | shown | shown | hidden |
| unset | `false` | shown (auto-suppress revoked) | hidden | hidden |
| `false` | `false` | hidden | hidden | shown (last-resort) |
| `true` | `false` | shown | hidden | hidden |
| `false` | `true` | hidden | shown | hidden |
| `true` | `true` | shown | shown | hidden |

Single-series charts: column hidden by default (nothing to identify).

**Common patterns:**
- Default multi-series: leave both unset -- you get the endpoint column, no traditional legend.
- "I want a top legend instead": `legend: { position: 'top' }, endpointLabels: false`.
- "Both legend and endpoint column": `legend: { show: true }` (endpointLabels stays auto-on).

## Legend Toggle (Runtime)

Clicking a legend entry hides/shows the corresponding series at runtime. This goes through engine recompile (not CSS hide), so:

- The y-axis **rebalances** to the remaining visible series.
- The color scale **stays locked** -- remaining lines keep their original palette colors (engine injects a stable `scale.domain` from the unfiltered data).
- **Per-series UI hides** with the line: endpoint chip, leader, dot annotation, and any text annotation anchored to that series.
- The **last visible series can't be hidden** (the toggle is a no-op).
- Range annotations and reference lines pass through unchanged (they anchor to constant axis values, not series).

Pass `onLegendToggle` to observe these clicks; you don't need to wire up `hiddenSeries` yourself for default behavior. Use `hiddenSeries` on the spec to start with specific series hidden on first render.

## Label Density (Charts Only)

`labels` accepts `density`, `format`, `prefix` (full shape: `LabelSpec` in `index.d.ts`). The choice that actually drives the chart is `density`:

| `density` | Behavior | Use when |
| --- | --- | --- |
| `"auto"` (default) | Show labels with collision detection | Most charts |
| `"all"` | Show every label, no collision detection | Few data points, precise values matter |
| `"endpoints"` | First and last per series only (legacy end-of-line labels on line/area) | Single-series line emphasizing start/end |
| `"none"` | No labels (tooltips + legend only) | Dense data, clean look |

For **multi-series line/area**, prefer `endpointLabels` (the chip+swatch column) over `density: "endpoints"` (the legacy fallback). The chip column wraps long names and resolves collisions; the legacy labels reserve a large right margin for long series names.

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

Use `seriesStyles: Record<string, SeriesStyle>` to override visuals for individual series, keyed by the color-field value. Use this for "highlight one series, dim the rest" patterns or for a dashed reference series alongside primary data. Field shape: `SeriesStyle` in `index.d.ts`. Editorial guidance + examples: [series-styles.md](references/series-styles.md).

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
| **Bar stacking is intentional** | If using `color` encoding on a bar chart, verify stacking mode. Default is stacked (`stack: "zero"`), which adds values together visually. For side-by-side comparison bars (e.g., 2018 vs 2022), set `stack: null` on the quantitative encoding. Stacked bars sum values visually, so a comparison chart will show bars extending to the sum of both values. |
| **Area stacking is intentional** | Area charts default to **overlap** (each series drawn over the others, semi-transparent). For composition/share-over-time, opt in with `stack: "zero"` on the y-channel (or `"normalize"` for percentage stacking, `"center"` for streamgraph). This is the opposite of bars: bars default to stacked, areas default to overlap. |
| **Y-domain fits the data** | Domain ceiling should be ~5-10% above the highest data value. `[0, 55]` for data peaking at 48.8 wastes space. Use `[0, 52]`. **For bar/column charts with narrow data ranges** (e.g., values between 200 and 280), don't default the floor to 0 - it makes variations invisible. Set the domain floor near the minimum value. Exception: charts where zero is a meaningful baseline (percent change from 0, counts). |
| **Annotations clear of data AND each other** | The engine auto-resolves annotation-to-annotation collisions, but start with good separation for cleaner results. Prefer 0-2 text annotations; use reflines for additional callouts. On scatter/bubble, use 40-100px offsets into empty quadrants with connectors. When using 2+ text annotations, verify with `playwright-cli`. |
| **Subtitle is intentional about wrapping** | Unintentional wrapping with orphaned fragments looks broken. Abbreviate, restructure, or use `\n` for explicit line breaks. Use shorthand keys in the subtitle (e.g., `"LI = low-income"`) rather than spelling everything out. |
| **Endpoint labels won't eat the chart** | The default chip+swatch column (`endpointLabels`) wraps long names at `width` (default 96px) and resolves collisions, so it handles long series names well. Only the legacy fallback (`labels: { density: "endpoints" }` with no `endpointLabels` and no top legend) reserves a huge right margin for long names — if you've forced that path, either abbreviate series names or switch to `legend: { position: "top" }, endpointLabels: false`. |
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
| Using chart mark for US state data | Use `type: "tilemap"` with state code keys |
| Not specifying axis format for currency/pct | Add `axis: { format: "$,.0f" }` or `".1f%"` |
| Using `".1%"` when data is already in percent form | `".1%"` multiplies by 100 (d3 convention). If data is `12.5` meaning 12.5%, use `".1f%"` (literal suffix) |
| Axis format and label format inconsistent | Set both `axis.format` and `labels.format` to the same pattern so ticks and data labels match |
| Using `darkMode: "auto"` in class-based dark mode apps | `"auto"` checks `prefers-color-scheme` only. For class-based toggles (Astro, Next.js), observe DOM and map to `"force"`/`"off"`. See [rendering](references/rendering.md) |
| Temporal scale with `nice: true` (default) creating dead space | `nice` rounds the domain outward (e.g., `2010-01` becomes `2008`). Set `scale: { domain: ["2010-01", "2026-01"], nice: false }` for precise control |
| Using `type` instead of `mark` for charts | Charts use `mark: "line"` (not `type: "line"`). Only tables, graphs, sankey, and tilemap use `type`. |

For design anti-patterns (titles, color, annotations), see [design review](references/design-review.md).

## Known Gotchas

Rendering and component behaviors that aren't obvious from the spec alone.

| Gotcha | Behavior | Fix |
| --- | --- | --- |
| Refline labels only support top/bottom | `labelAnchor` on refline annotations only accepts `"top"` or `"bottom"`. Left/right values are accepted in the type but have no visible effect on reflines (they do work on range annotations). | Set `label: ""` on the refline and add a separate `type: "text"` annotation positioned where you want a side label. |
| Endpoint chip labels wrap by width, not `\n` | The chip+swatch column wraps long series names at `endpointLabels.width` (default 96px). `\n` in the series name does not create a hard break. | Either shorten the series name in the data, or raise `width` if you have horizontal room. |
| Area defaults to overlap, not stacked | Multi-series area charts now overlap by default. Setting `color` without `stack: "zero"` on the y-channel produces semi-transparent overlapping fills, not a stacked composition. | For share-over-time, opt in with `encoding: { y: { ..., stack: "zero" } }`. For percentage, use `"normalize"`. |
| `connector: 'drop-line'` only flips against the chart edge | The drop-line connector renders a vertical line through the data point's x and lays the label beside it. The auto-flip only checks against the chart area edge -- it does not avoid neighboring marks or other annotations. | Place the annotation away from cluttered regions; if collisions persist, switch to `connector: 'curve'` with a manual `offset`. |
| DataTable CSS overrides unreliable | Custom CSS targeting `.oc-table-wrapper td` may not apply due to CSS specificity. | Use the DataTable `style` prop for inline overrides: `<DataTable style={{ paddingLeft: 10 }} spec={...} />`. |
| Scatter plots auto-set `zero: false` | Unlike other chart types, scatter/point marks automatically set `scale.zero: false` on both axes if not explicitly configured. This means scatter domains fit tightly to data. | To include zero, explicitly set `scale: { zero: true }` on the relevant axis. Be aware that scatter and bar/line charts handle zero differently by default. |
| Constant colors require `mark.fill`, not encoding | `encoding.color: { value: "#hex" }` will error. The color encoding channel requires a `field` that maps to data. | Use `mark: { type: "bar", fill: "#1b7fa3" }` for constant colors across all marks. |
| Default gradient direction is top-to-bottom | A gradient with no explicit `x1/y1/x2/y2` defaults to vertical (top-to-bottom). On horizontal bars, the engine auto-orients this to left-to-right. On other marks, set coordinates explicitly. | For left-to-right: `x1:0, y1:0, x2:1, y2:0`. For top-to-bottom (default): omit coordinates or use `x1:0, y1:0, x2:0, y2:1`. |
| Layer scale mismatch | Second layer renders at wrong positions when layers have different value ranges. | For independent y-scales (e.g., revenue + enrollment), use `resolve: { scale: { y: "independent" } }` on the layer spec -- this renders both series correctly with left and right y-axes. For simple overlays where both series share the same scale, set an explicit `scale.domain` on both layers. Prefer `annotations` with `refline` over a full layer when you just need to add a threshold line. |

## Custom D3.js Infographics

When a visualization goes beyond what declarative specs can handle (creative metaphors, unusual layouts, treemaps, generative art, scrollytelling), fall back to raw D3.js + SVG. Note: sankey and tilemap are first-class types with their own spec formats. See [references/sankey.md](references/sankey.md) and [references/tilemap.md](references/tilemap.md). Use the D3 reference only for heavily customized layouts. These references cover D3 implementation patterns:

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
