# Rendering & APIs

Framework-specific rendering, event handlers, and builder functions.

## Styles

**Required.** Import the stylesheet from whichever package you installed. Without it, tooltips, legends, accessibility elements, and typography won't render correctly.

```typescript
// Import from your framework package:
import '@opendata-ai/openchart-react/styles.css';
// or
import '@opendata-ai/openchart-vue/styles.css';
// or
import '@opendata-ai/openchart-svelte/styles.css';
// or (vanilla JS)
import '@opendata-ai/openchart-vanilla/styles.css';
```

Import once at your app's entry point (e.g. `main.tsx`, `app.css`, `layout.svelte`). The stylesheet is plain CSS with `oc-` prefixed class names and CSS custom properties for theming. It includes dark mode overrides via the `.oc-dark` class, which the components manage automatically.

## React

```tsx
import '@opendata-ai/openchart-react/styles.css';
import { Chart, DataTable, Graph, VizThemeProvider } from '@opendata-ai/openchart-react';

<Chart spec={chartSpec} darkMode="auto" />
<DataTable spec={tableSpec} onRowClick={(row) => console.log(row)} />
<Graph spec={graphSpec} onNodeClick={(node) => console.log(node)} />

<VizThemeProvider theme={myTheme}>
  <Chart spec={spec1} />
  <DataTable spec={spec2} />
</VizThemeProvider>
```

### Dark Mode

The `darkMode` prop controls chart color scheme:

| Value | Behavior |
| --- | --- |
| `"off"` | Always light (default) |
| `"auto"` | Follows `prefers-color-scheme` media query |
| `"force"` | Always dark |

**Class-based dark mode (Astro, Next.js, etc.):** If your app toggles dark mode via a CSS class on `<html>` instead of relying on `prefers-color-scheme`, `darkMode: "auto"` won't react to theme changes. Create a wrapper component that reads the theme from the DOM and maps it to `"force"` or `"off"`:

```tsx
import { useEffect, useState } from "react";
import { Chart } from "@opendata-ai/openchart-react";

function ThemedChart({ spec, darkMode = "auto", ...props }) {
  const [docTheme, setDocTheme] = useState(() =>
    typeof document !== "undefined"
      ? document.documentElement.classList.contains("dark") ? "dark" : "light"
      : "dark"
  );

  useEffect(() => {
    setDocTheme(document.documentElement.classList.contains("dark") ? "dark" : "light");
    const observer = new MutationObserver(() => {
      setDocTheme(document.documentElement.classList.contains("dark") ? "dark" : "light");
    });
    observer.observe(document.documentElement, { attributes: true, attributeFilter: ["class"] });
    return () => observer.disconnect();
  }, []);

  const resolved = darkMode === "auto"
    ? docTheme === "dark" ? "force" : "off"
    : darkMode;

  return <Chart spec={spec} darkMode={resolved} {...props} />;
}
```

**Do NOT use React context** (e.g., `useContext(ThemeContext)`) for this in Astro or other island-based frameworks. Each `client:only` component is its own React tree with no shared context. Read theme state from the DOM instead.

## Vue

```vue
import { Chart, DataTable, Graph, VizThemeProvider } from '@opendata-ai/openchart-vue';

<Chart :spec="chartSpec" darkMode="auto" />
<DataTable :spec="tableSpec" />
<Graph :spec="graphSpec" />
```

## Svelte

```svelte
import { Chart, DataTable, Graph, ThemeProvider } from '@opendata-ai/openchart-svelte';

<Chart {spec} darkMode="auto" />
<DataTable {spec} />
<Graph {spec} />
```

## CDN / Standalone HTML

When using OpenChart via CDN (esm.sh, unpkg, etc.) or in a standalone HTML file, you must load the CSS separately. Without it, chrome elements (source, byline), the brand watermark, table styling, tooltips, and accessibility tables will be broken.

```html
<!-- Load the stylesheet -->
<link rel="stylesheet" href="https://esm.sh/@opendata-ai/openchart-vanilla/styles.css">

<script type="module">
import { createChart, createTable } from 'https://esm.sh/@opendata-ai/openchart-vanilla';

createChart(document.getElementById('chart'), spec);
createTable(document.getElementById('table'), tableSpec);
</script>
```

The CSS is required for: table styling (borders, alignment, heatmaps, bars), chart chrome (source, footer, byline text), brand watermark ("OpenData" in footer-right), tooltip appearance, legend layout, and screen-reader accessibility tables (hidden via `oc-sr-only` class).

## Vanilla JS

```typescript
import { createChart, createTable, createGraph } from "@opendata-ai/openchart-vanilla";

// Charts
const chart = createChart(container, spec, { darkMode: "auto", responsive: true });
chart.update(newSpec);
const svgString = chart.export("svg");           // returns string
const pngBlob = await chart.export("png");       // returns Promise<Blob>
const csvString = chart.export("csv");            // returns string
chart.destroy();

// Tables
const table = createTable(container, tableSpec, { responsive: true });
table.export("csv");                              // returns string (all filtered/sorted rows)
table.getState();                                 // { sort, search, page }
table.setState({ search: "query", page: 0 });    // programmatic state control
table.destroy();

// Graphs
const graph = createGraph(container, graphSpec, { responsive: true });
graph.search("query");
graph.zoomToFit();
graph.zoomToNode("node-id");
graph.selectNode("node-id");
graph.destroy();
```

## Event Handlers

**Charts:** `onMarkClick`, `onMarkHover`, `onMarkLeave`, `onLegendToggle`, `onAnnotationClick`, `onEdit`
**Tables:** `onRowClick`, `onSortChange`, `onSearchChange`, `onPageChange`
**Graphs:** `onNodeClick`, `onNodeDoubleClick`, `onSelectionChange`

`onEdit` is the unified edit callback. Passing it to `<Chart>` activates edit mode, making text annotations, connectors, range/refline labels, chrome elements, series labels, and the legend all draggable. It fires a typed `ElementEdit` discriminated union. See [editing reference](editing.md) for the full API and how to persist edits back to the spec.

## Builder Functions

Shorthand for common specs. Import from `@opendata-ai/openchart-core`.

```typescript
import { lineChart, barChart, columnChart, pieChart, scatterChart, dataTable, inferFieldType } from "@opendata-ai/openchart-core";

const spec = lineChart(data, "date", "revenue", { color: "region", chrome: { title: "Revenue by region" } });
const spec = barChart(data, "category", "value");  // note: barChart(data, category, value) not (data, x, y)
const spec = dataTable(data, { search: true, pagination: { pageSize: 20 } });

// inferFieldType samples data values and returns "quantitative"|"temporal"|"nominal"
const fieldType = inferFieldType(data, "fieldName");
```

Builder functions accept field names as strings (auto-infers type from data) or full `EncodingChannel` objects for explicit control.

## Animation Support

Entrance animations are CSS-driven and work automatically across all rendering targets. The vanilla renderer applies CSS classes and uses the Web Animations API for SVG path drawing. No framework-specific animation code is needed.

- React, Vue, and Svelte wrappers inherit animation support for free since it runs at the DOM/CSS level
- Animation is controlled via the `animation` spec property, not framework props
- `prefers-reduced-motion` is respected automatically across all frameworks
- Animation replays on each mount. For components that remount frequently (e.g., tab switches), consider disabling animation to avoid repeated entrance effects

See [animation reference](animation.md) for the full spec, easing presets, stagger behavior, and per-mark defaults.
