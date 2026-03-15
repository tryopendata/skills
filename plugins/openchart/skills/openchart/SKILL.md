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

## Chart Selection Decision Tree

```
Single value to highlight    -> Use chrome.title as a big number display
Temporal x-axis column?      -> 1 series: line | 2-5 series: line + color | 6+: filter to top 5
Categorical + numeric?       -> Ranked list: bar (horizontal) | Periodic (Q1, Jan): column | 2-6 composition: donut
Two numeric columns?         -> scatter (optional size/color for 3rd/4th dims)
Categorical + series + num?  -> stacked bar or stacked column (use color for series)
Distribution/spread?         -> dot (strip plot)
Nodes + edges / network?     -> graph (force/radial/hierarchical layout)
Tabular data overview?       -> table (with sparklines, heatmaps, bars)
Default                      -> bar
```

## Visualization Types

Each type has a detailed reference with full spec, encoding rules, and examples. Load the reference when you need the details.

| Type | Best for | Data model | Reference |
| --- | --- | --- | --- |
| `line` | Trends over time | x: temporal/ordinal, y: quantitative | [references/line.md](references/line.md) |
| `area` | Trends with volume emphasis | x: temporal/ordinal, y: quantitative | [references/area.md](references/area.md) |
| `bar` | Rankings, comparisons (horizontal) | x: quantitative, y: nominal/ordinal | [references/bar.md](references/bar.md) |
| `column` | Periodic data, categories (vertical) | x: nominal/ordinal/temporal, y: quantitative | [references/column.md](references/column.md) |
| `pie` | Part-to-whole (2-5 categories) | y: quantitative, color: nominal/ordinal | [references/pie-donut.md](references/pie-donut.md) |
| `donut` | Part-to-whole (preferred over pie) | y: quantitative, color: nominal/ordinal | [references/pie-donut.md](references/pie-donut.md) |
| `dot` | Distribution, strip plots | x: quantitative, y: nominal/ordinal | [references/dot.md](references/dot.md) |
| `scatter` | Correlation between two variables | x: quantitative, y: quantitative | [references/scatter.md](references/scatter.md) |
| `table` | Data tables with visual features | columns + data rows | [references/table.md](references/table.md) |
| `graph` | Networks, relationships, hierarchies | nodes + edges | [references/graph.md](references/graph.md) |

**Cross-cutting references:**
- [Annotations](references/annotations.md) (spec syntax for text callouts, ranges, reference lines)
- [Editing](references/editing.md) (onEdit callback, ElementEdit type, persisting drag positions for all elements)
- [Responsive](references/responsive.md) (breakpoints, layout strategies, designing for mobile)
- [Theme customization](references/theme.md) (colors, fonts, spacing config)
- [Rendering & APIs](references/rendering.md) (React, Vue, Svelte, Vanilla JS, events, builders)

**Design philosophy references:**

| Designing... | Load | Reference |
| --- | --- | --- |
| Chart type for a story | Story-driven chart selection | [references/chart-selection.md](references/chart-selection.md) |
| Color palette, emphasis | Color as narrative | [references/color-strategy.md](references/color-strategy.md) |
| Titles, subtitles, annotations | Editorial writing | [references/editorial-writing.md](references/editorial-writing.md) |
| Type sizing, hierarchy | Typography | [references/typography.md](references/typography.md) |
| Whether a chart is "done" | Design review | [references/design-review.md](references/design-review.md) |
| Rendered output QA | Visual defect scanning | [references/visual-qa.md](references/visual-qa.md) |
| Mobile / narrow containers | Responsive design | [references/responsive.md](references/responsive.md) |
| Rendered output verification | Visual defect scanning | [references/visual-qa.md](references/visual-qa.md) |

## Shared Spec Structure

All visualization types share these properties:

```typescript
{
  type: string,              // REQUIRED: discriminant ("line", "table", "graph", etc.)
  chrome?: {                 // editorial text elements
    title?: string | { text: string, style?: ChromeTextStyle },
    subtitle?: string | { text: string, style?: ChromeTextStyle },
    source?: string | { text: string, style?: ChromeTextStyle },
    byline?: string | { text: string, style?: ChromeTextStyle },
    footer?: string | { text: string, style?: ChromeTextStyle },
  },
  theme?: ThemeConfig,       // see references/theme.md
  darkMode?: "auto"|"force"|"off",  // default: "off"
  responsive?: boolean,      // default: true
}
```

**ChromeTextStyle:** `{ fontSize?: number, fontWeight?: number, fontFamily?: string, color?: string }`

## Encoding Channels (Charts Only)

Charts (line, area, bar, column, pie, donut, dot, scatter) use encoding channels to map data fields to visual properties:

```typescript
encoding: {
  x?: EncodingChannel,      // horizontal position
  y?: EncodingChannel,      // vertical position
  color?: EncodingChannel,  // series differentiation
  size?: EncodingChannel,   // bubble/dot scaling (quantitative)
  detail?: EncodingChannel, // grouping without visual mapping
}
```

**EncodingChannel:**
```typescript
{
  field: string,             // REQUIRED: column name in data
  type: FieldType,           // REQUIRED: "quantitative"|"temporal"|"nominal"|"ordinal"
  aggregate?: AggregateOp,   // "count"|"sum"|"mean"|"median"|"min"|"max"
  axis?: {
    label?: string,          // axis label (default: field name)
    format?: string,         // d3-format string, e.g. ",.0f" "$,.2f" ".1%"
    tickCount?: number,      // override tick count
    grid?: boolean,          // show gridlines
  },
  scale?: {
    domain?: [number, number] | string[],  // explicit domain
    type?: "linear"|"log"|"time"|"band"|"point"|"ordinal",
    nice?: boolean,          // clean tick values (default: true)
    zero?: boolean,          // include zero (default: true for quantitative)
  },
}
```

## Legend Configuration (Charts Only)

```typescript
legend?: {
  position?: "top"|"right"|"bottom"|"bottom-right"|"inline",
  show?: boolean,  // default: true. Set false to hide legend entirely.
}
```

Position is responsive by default (the engine picks based on container width). Set explicitly to override. Use `show: false` to hide the legend when it's redundant (e.g., bar charts where the y-axis already labels each category).

## Label Density (Charts Only)

```typescript
labels?: {
  density?: "all"|"auto"|"endpoints"|"none",  // default: "auto"
  format?: string,           // d3-format for label values (supports literal suffix, see below)
}
```

| Mode | Behavior | Use when |
| --- | --- | --- |
| `auto` | Show labels with collision detection | Default, most charts |
| `all` | Show every label, no collision detection | Few data points, precise values matter |
| `endpoints` | First and last per series only | Line charts, emphasize start/end |
| `none` | No labels (tooltips + legend only) | Dense data, clean look |

## Format Strings

Both `axis.format` and `labels.format` accept [d3-format](https://d3js.org/d3-format) strings, plus a literal suffix extension.

**Literal suffix pattern:** Append non-format characters after a valid d3-format specifier. The engine tries d3-format first; if it fails, it splits off the trailing suffix and applies d3-format to the rest.

| Format | Input | Output | Use case |
| --- | --- | --- | --- |
| `".1f"` | 12.5 | `12.5` | One decimal, no units |
| `".1f%"` | 12.5 | `12.5%` | Percentage (data already in %, not 0-1) |
| `".0f%"` | 12 | `12%` | Whole-number percentage |
| `"$,.0f"` | 1234 | `$1,234` | Currency with commas |
| `"$,.2f"` | 3.75 | `$3.75` | Currency with cents |
| `".1%"` | 0.125 | `12.5%` | d3 native percent (multiplies by 100) |
| `"~s"` | 10000 | `10k` | SI suffix (k, M, G) for large numbers |
| `"~s"` | 1500000 | `1.5M` | SI suffix, auto-scales |
| `"$~s"` | 78000 | `$78k` | Currency with SI suffix |
| `",.0f"` | 132979 | `132,979` | Comma-separated, no decimals |

**When data is already in percentage form** (e.g., `12.5` meaning 12.5%), use `".1f%"` not `".1%"`. The d3 `%` type multiplies by 100, so `0.125` becomes `12.5%` but `12.5` becomes `1,250.0%`.

## Per-Series Styling (Charts Only)

Override visual properties for individual series by name. Useful for making reference series visually distinct from primary data.

```typescript
seriesStyles?: Record<string, {
  lineStyle?: "solid"|"dashed"|"dotted",
  showPoints?: boolean,
  strokeWidth?: number,
  opacity?: number,
}>
```

**Example:** Show "Fed funds rate" as a dashed reference line alongside solid CPI data:
```json
{
  "seriesStyles": {
    "Fed funds rate": {
      "lineStyle": "dashed",
      "showPoints": false,
      "strokeWidth": 1.5,
      "opacity": 0.6
    }
  }
}
```

Series names come from the `color` encoding field values. Only specified series get overrides; others render normally.

## Breakpoint Overrides (Charts Only)

Override chrome, labels, legend, or annotations per breakpoint. See [responsive reference](references/responsive.md) for full details.

```typescript
overrides?: {
  compact?: { chrome?, labels?, legend?, annotations? },
  medium?: { chrome?, labels?, legend?, annotations? },
  full?: { chrome?, labels?, legend?, annotations? },
}
```

**Example:** Shorter title and hidden legend at mobile:
```json
{
  "chrome": { "title": "Inflation hit a 40-year high in June 2022" },
  "overrides": {
    "compact": {
      "chrome": { "title": "Inflation hit a 40-year high" },
      "legend": { "show": false }
    }
  }
}
```

## First Draft Checklist

Run these checks before outputting a spec. These catch the issues that most often require iteration after rendering.

| Check | What to verify |
| --- | --- |
| **Color encodes the story** | If one variable drives the narrative, color should reinforce it. Use the decision table in [color-strategy.md](references/color-strategy.md) to pick the right strategy and `theme.colors` array. Don't leave a scatter plot monochrome when a gradient would make the pattern obvious. |
| **Y-domain fits the data** | Domain ceiling should be ~5-10% above the highest data value. `[0, 55]` for data peaking at 48.8 wastes space. Use `[0, 52]`. |
| **Annotations clear of data AND each other** | The engine auto-resolves annotation-to-annotation collisions, but start with good separation for cleaner results. Prefer 0-2 text annotations; use reflines for additional callouts. On scatter/bubble, use 40-100px offsets into empty quadrants with connectors. When using 2+ text annotations, verify with `playwright-cli`. |
| **Subtitle fits one line** | If the subtitle wraps, the orphaned fragment looks broken. Abbreviate or restructure. Use shorthand keys in the subtitle (e.g., `"LI = low-income"`) rather than spelling everything out. |
| **Endpoint labels won't eat the chart** | `labels: { density: "endpoints" }` with series names > 15 chars reserves a huge right margin. Use `"none"` + `legend: { position: "top" }` instead, or abbreviate series names. |
| **Axis ticks show units** | Percentages should show `10%` not `10`. Use `format: ".0f%"` when data is already in percent form (e.g., 10 meaning 10%). Use `format: ".0%"` only when data is in decimal form (0.10 meaning 10%). Large numbers should use SI suffixes: `format: "~s"` turns 10000 into `10k` and 1000000 into `1M`. For currency: `format: "$~s"` gives `$10k`, `$1M`. See the format table below. |
| **Consistent color palette across related charts** | If multiple charts in the same article cover the same dimension (e.g., poverty), use the same color mapping (blue = low, red = high) so the reader builds a mental model. |

## Spec Anti-Patterns

| Mistake | Fix |
| --- | --- |
| Using `nominal` for numeric field | Use `quantitative` for numbers, `temporal` for dates |
| Using `ordinal` for temporal data | Use `temporal`; ordinal is for ordered categories |
| Huge inline data arrays (500+ rows) | Aggregate or sample before passing to spec |
| Forgetting encoding.color for multi-series | Line/bar with groups needs `color` channel |
| Bar chart for time series | Use line or column for temporal data |
| Using chart for network data | Use `type: "graph"` with nodes + edges |
| Not specifying axis format for currency/pct | Add `axis: { format: "$,.0f" }` or `".1f%"` |
| Using `".1%"` when data is already in percent form | `".1%"` multiplies by 100 (d3 convention). If data is `12.5` meaning 12.5%, use `".1f%"` (literal suffix) |
| Axis format and label format inconsistent | Set both `axis.format` and `labels.format` to the same pattern so ticks and data labels match |
| Using `darkMode: "auto"` in class-based dark mode apps | `"auto"` checks `prefers-color-scheme` only. For class-based toggles (Astro, Next.js), observe DOM and map to `"force"`/`"off"`. See [rendering](references/rendering.md) |
| Temporal scale with `nice: true` (default) creating dead space | `nice` rounds the domain outward (e.g., `2010-01` becomes `2008`). Set `scale: { domain: ["2010-01", "2026-01"], nice: false }` for precise control |

For design anti-patterns (titles, color, annotations), see [design review](references/design-review.md).

## Custom D3.js Infographics

When a visualization goes beyond what declarative specs can handle (creative metaphors, unusual layouts, treemaps, sankeys, generative art, scrollytelling), fall back to raw D3.js + SVG. These references cover D3 implementation patterns:

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

### Infographic Design System

When building custom D3 infographics (not OpenChart specs), use these defaults:

**Export dimensions:**

| Platform | Dimensions | Aspect Ratio |
|----------|------------|--------------|
| Twitter/X | 1200 x 675 | 16:9 |
| Instagram Feed | 1080 x 1080 | 1:1 |
| LinkedIn | 1200 x 627 | 1.91:1 |
| Instagram Story | 1080 x 1920 | 9:16 |

**Typography:** Inter (weights 400, 500, 600, 800). Headline 48px/800, subtitle 20px/400, data labels 16px/600 (`tabular-nums`), axis labels 14px/500, source 12px/400.

**Spacing:** Outer margin 48px, chart padding 24px, element spacing on 4px grid (8, 12, 16, 24, 32).

**Headline formula:** `[What happened] + [How much/when/who]` - sentence case, no period, under 10 words, numerals not words.

**Annotation hierarchy:** 1 callout max (14px+, leader line), 2-3 labels (12-14px, inline), 1 note max (11-12px, bottom).

**Watermark:** Bottom-right, 48px from edges, 60px height, 85% opacity, contrast-aware.
