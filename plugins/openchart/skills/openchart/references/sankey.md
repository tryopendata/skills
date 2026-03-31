# Sankey Diagram

Flow visualization showing how values move between stages or categories. Nodes represent entities, links represent flows with width proportional to value.

**Use cases:** Energy flows, budget allocation, user funnels, process flows, trade routes, migration patterns, cost breakdowns.

**Important:** Sankey uses its own component, not `<Chart>`. Import `Sankey` from `@opendata-ai/openchart-react`:

```tsx
import { Sankey } from '@opendata-ai/openchart-react';
// NOT: import { Chart } from '../components/Chart'  // Chart only handles mark-based specs

<Sankey spec={{ type: 'sankey', data: [...], encoding: {...} }} />
```

If your app uses class-based dark mode (not `prefers-color-scheme`), pass `darkMode="force"` or `darkMode="auto"` to the `<Sankey>` component prop.

## Encoding

| Channel | Required | Type |
| --- | --- | --- |
| source | Yes | nominal |
| target | Yes | nominal |
| value | Yes | quantitative |
| color | No | nominal |
| tooltip | No | any |

## SankeySpec

```typescript
{
  type: 'sankey',
  data: DataRow[],           // flat rows with source, target, value columns
  encoding: {
    source: { field: string, type: 'nominal' },
    target: { field: string, type: 'nominal' },
    value: { field: string, type: 'quantitative' },
    color?: { field: string, type: 'nominal' },
  },
  nodeWidth?: number,        // pixel width of node rectangles. Default: 12
  nodePadding?: number,      // vertical spacing between nodes. Default: 16
  nodeAlign?: 'left' | 'right' | 'center' | 'justify',  // Default: 'justify'
  iterations?: number,       // layout relaxation passes. Default: 6
  linkStyle?: 'gradient' | 'source' | 'target' | 'neutral',  // Default: 'gradient'
  chrome?: Chrome,
  legend?: LegendConfig,
  theme?: ThemeConfig,
  darkMode?: DarkMode,
  animation?: AnimationSpec,
  valueFormat?: string,       // d3-format for tooltip values. ".0f%" appends %, "$~s" for $10k
}
```

## Layout Options

| Option | Default | Effect |
| --- | --- | --- |
| `nodeWidth` | 12 | Wider nodes are easier to read but consume horizontal space. 10-24 is the practical range. |
| `nodePadding` | 16 | Vertical gap between nodes in the same column. Increase for fewer nodes, decrease for many. |
| `nodeAlign` | `'justify'` | `justify` spreads nodes across full height. `left` packs toward the source side. `right` packs toward the target side. `center` centers vertically. |
| `iterations` | 6 | More iterations reduce link crossings but take longer. 6 is fine for most diagrams. Bump to 12-32 for complex flows with many crossings. |
| `linkStyle` | `'gradient'` | `gradient` blends source-to-target color. `source` colors links by their origin node. `target` colors by destination. `neutral` uses a single muted tone. |

## Design Guidance

- **Gradient links are the default for a reason.** They visually connect source to target and look the most polished. Switch to `source` or `target` coloring only when you need to emphasize one side of the flow.
- **Link opacity** defaults to 0.35 with hover highlight to 0.7. This keeps the diagram readable without overwhelming the nodes.
- **Node labels** auto-position outside nodes based on column: left-column labels go left, right-column labels go right, middle columns go to whichever side has more room.
- **Theme palettes work out of the box.** Node colors come from the theme's categorical palette. Set `encoding.color` to override with a specific field.
- **Keep node count under 30.** Beyond that, the diagram becomes a tangle of crossing links. Aggregate small categories into "Other" to stay readable.

## Examples

### Energy Flow (3 columns)

```json
{
  "type": "sankey",
  "data": [
    { "source": "Coal", "target": "Electricity", "value": 250 },
    { "source": "Natural Gas", "target": "Electricity", "value": 180 },
    { "source": "Natural Gas", "target": "Heating", "value": 120 },
    { "source": "Solar", "target": "Electricity", "value": 90 },
    { "source": "Electricity", "target": "Residential", "value": 210 },
    { "source": "Electricity", "target": "Industrial", "value": 200 },
    { "source": "Electricity", "target": "Commercial", "value": 110 },
    { "source": "Heating", "target": "Residential", "value": 80 },
    { "source": "Heating", "target": "Commercial", "value": 40 }
  ],
  "encoding": {
    "source": { "field": "source", "type": "nominal" },
    "target": { "field": "target", "type": "nominal" },
    "value": { "field": "value", "type": "quantitative" }
  },
  "chrome": {
    "title": "Natural gas serves double duty as fuel and heat source",
    "subtitle": "U.S. energy flows by source and end use, terawatt-hours",
    "source": "Source: EIA Annual Energy Review"
  }
}
```

### Budget Allocation (2 columns)

```json
{
  "type": "sankey",
  "data": [
    { "source": "Federal Revenue", "target": "Defense", "value": 886 },
    { "source": "Federal Revenue", "target": "Social Security", "value": 1354 },
    { "source": "Federal Revenue", "target": "Medicare", "value": 848 },
    { "source": "Federal Revenue", "target": "Interest", "value": 659 },
    { "source": "Federal Revenue", "target": "Other", "value": 1520 }
  ],
  "encoding": {
    "source": { "field": "source", "type": "nominal" },
    "target": { "field": "target", "type": "nominal" },
    "value": { "field": "value", "type": "quantitative" }
  },
  "linkStyle": "target",
  "chrome": {
    "title": "Social Security is the largest single budget item",
    "subtitle": "Federal spending by category, billions USD, FY2024",
    "source": "Source: Congressional Budget Office"
  }
}
```

### User Funnel (4 stages)

```json
{
  "type": "sankey",
  "data": [
    { "source": "Website Visit", "target": "Sign Up", "value": 8500 },
    { "source": "Website Visit", "target": "Bounce", "value": 14200 },
    { "source": "Sign Up", "target": "Onboarding Complete", "value": 5100 },
    { "source": "Sign Up", "target": "Abandoned", "value": 3400 },
    { "source": "Onboarding Complete", "target": "Paid Conversion", "value": 1800 },
    { "source": "Onboarding Complete", "target": "Free Tier", "value": 3300 }
  ],
  "encoding": {
    "source": { "field": "source", "type": "nominal" },
    "target": { "field": "target", "type": "nominal" },
    "value": { "field": "value", "type": "quantitative" }
  },
  "nodeAlign": "left",
  "chrome": {
    "title": "Only 8% of visitors convert to paid users",
    "subtitle": "User acquisition funnel, trailing 30 days",
    "source": "Source: Product analytics"
  }
}
```

## When to Use vs. Alternatives

| Situation | Use |
| --- | --- |
| Flow between 3+ stages | Sankey |
| Simple 2-stage breakdown | Stacked bar (simpler, more accurate) |
| Bidirectional flows | Chord diagram (sankey is directional only) |
| Part-to-whole without flow | Arc/donut or treemap |
| Comparing totals across categories | Bar chart |

## Value Formatting

Use `valueFormat` to format flow values in tooltips and ARIA labels:

```json
{
  "type": "sankey",
  "valueFormat": ".0f%",
  "data": [...]
}
```

Accepts d3-format strings with literal suffix extension. Common patterns: `".0f%"` (append %), `"$,.0f"` (currency), `"$~s"` (SI currency like $10k).

## Dark Mode

The Sankey component uses `darkMode: "auto"` by default (reads `prefers-color-scheme`). For apps with class-based dark mode toggles (`.dark` on `<html>`), pass `darkMode="force"` or `darkMode="auto"` to the component or set `darkMode: "force"` in the spec.

In dark mode, link gradient opacity is raised to 0.75 (vs 0.5 in light mode) for visibility, and node colors use the original vivid palette rather than the dark-adapted colors that charts use for text/labels.

## Gotchas

| Issue | Details |
| --- | --- |
| Circular flows | Sankey requires an acyclic graph. If A flows to B and B flows back to A, the layout breaks. Aggregate bidirectional flows into net flows or use a chord diagram. |
| Too many link crossings | More than ~20-25 links start looking like spaghetti. Try different `nodeAlign` values (`justify` vs `left`) and increase `iterations` to reduce crossings. |
| Node count explosion | Keep under 30 nodes. Aggregate small nodes into "Other" categories. A 50-node sankey is unreadable. |
| Narrow links invisible | Links with very small values relative to the largest flow become hairlines. Filter out flows below a meaningful threshold or aggregate them. |
| Label overlap on dense columns | When a column has many nodes stacked tightly, labels collide. Increase `nodePadding` or abbreviate label text. |
| Node names with special characters | Node names are used in SVG gradient IDs. Special characters (spaces, `$`, `&`, etc.) are automatically sanitized to underscores. No action needed, but be aware that DevTools gradient IDs will show sanitized names. |
| Single-source fan-out | When one source node feeds many targets (e.g., "Income" splitting into 5+ categories), the gradient links all start from the same color and may look similar. Use intermediate grouping nodes (e.g., Income → Essential costs → Housing/Food/Transport) to create a richer color palette across flows. |
| React linked packages and Vite | When developing with `bun link` across repos, Vite aggressively caches pre-bundled dependencies. Always clear `node_modules/.vite` after rebuilding linked packages, and ensure ALL transitive packages are linked (core, engine, vanilla, react). |
