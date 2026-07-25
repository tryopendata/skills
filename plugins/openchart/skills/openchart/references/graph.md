# Graph (Network Visualization)

Network and relationship visualizations using nodes and edges. Rendered on canvas with force-directed, radial, or hierarchical layouts. Supports zoom, pan, node dragging, search, and selection.

## Data Model

Unlike charts which use flat `data` arrays with encoding channels, graphs use a **nodes + edges** model. For the full shapes (`GraphSpec`, `GraphNode`, `GraphEdge`, `GraphEncoding`, `GraphEncodingChannel`, `GraphLayoutConfig`, `NodeOverride`), load `index.d.ts`.

The model in one breath: nodes are `{ id: string, ...data }`, edges are `{ source: string, target: string, ...data }` referencing node ids. All other fields on either object are arbitrary data available to encodings, tooltips, and clustering.

## Graph Encoding

Graph encoding uses `GraphEncodingChannel` (load from `index.d.ts`). All channels are optional since a graph renders fine with uniform appearance. Supported channels: `nodeColor`, `nodeSize`, `edgeColor`, `edgeWidth`, `edgeStyle`, `nodeLabel`.

When `scale.domain` and `scale.range` are provided, the engine uses them directly instead of auto-deriving from data. This controls deterministic color assignment:

```typescript
encoding: {
  nodeColor: {
    field: "dept",
    type: "nominal",
    scale: {
      domain: ["Engineering", "Design", "Product"],
      range: ["#58a6ff", "#3fb950", "#d29922"],
    },
  },
}
```

| Channel | Effect | Type constraint |
| --- | --- | --- |
| nodeColor | Color nodes by category or value | nominal, ordinal, quantitative |
| nodeSize | Scale node radius by value. `scale.type` defaults to `'sqrt'`; set `'linear'` for a linear ramp. `scale.range` sets the px extent (default `[3, 12]`) | quantitative |
| nodeOpacity | Opacity mapping for nodes. Quantitative maps linearly to `[0.25, 1]`; `scale.range` overrides. No `edgeOpacity` channel exists (edge alpha is owned by interaction) | nominal, ordinal, quantitative |
| edgeColor | Color edges by category or value | nominal, ordinal, quantitative |
| edgeWidth | Scale edge width by value (0.5-4px) | quantitative |
| edgeStyle | Map edge line style (solid/dashed/dotted) | nominal, ordinal |
| nodeLabel | Display text labels on nodes | any |

When `nodeColor` encoding is set, it takes precedence over community-based coloring from `layout.clustering`. Communities still affect spatial grouping, but colors come from the encoding.

**Channel `sort`** (graph channels only): defaults to `'ascending'` for a stable category order. This is a deliberate divergence from chart channels, which keep data order. Remedy to restore data order: `sort: null`. **Channel `highlight`** (a value list, e.g. `nodeColor: { highlight: ['Design'] }`): emphasizes those categories on load and dims the rest. Distinct from the runtime `highlight()` verb and `interaction.hover.mode`.

## Node Overrides

`nodeOverrides` is `Record<string, NodeOverride>` keyed by node id — useful for seed-node styling. Override fields: `fill`, `radius`, `strokeWidth`, `stroke`, `alwaysShowLabel`. Load `NodeOverride` from `index.d.ts` for the exact shape.

Reach for `nodeOverrides` for seed-node styling and the `alwaysShowLabel` importance threshold (there is no label-visibility encoding channel). For runtime dimming/emphasis, prefer the `highlight()` verb over recompiling with overrides. Note `highlight({ category })` can't exempt a seed node; use the `{ nodeIds }` form for that.

## Layout Configuration

`layout: GraphLayoutConfig` (load from `index.d.ts`) chooses the simulation type and tunes the force parameters. Defaults worth knowing:

- `centerForce` defaults to `true`
- `collisionPadding` defaults to 2px
- `chargeStrength`, `linkDistance`, `linkStrength` use d3-force defaults if omitted
- `clustering.field` enables cluster forces — nodes with the same field value are pulled together

| Layout | Best for | Notes |
| --- | --- | --- |
| `force` | General networks | Default. Nodes repel, edges attract. Good for most graphs. |
| `radial` | Hub-and-spoke | Arranges nodes in concentric circles. |
| `hierarchical` | Trees, DAGs | Top-down or left-right arrangement. |

Plain-language layout tuning (prefer these presets over raw d3 numbers):

| Field | Values | Effect |
| --- | --- | --- |
| `seed` | number | Deterministic layout. Same spec + seed gives an identical settled layout within one execution path. |
| `energy` | `gentle` / `balanced` / `energetic` | Repulsion preset. A raw `chargeStrength` wins over the preset. |
| `settle` | `quick` / `balanced` / `thorough` | Cool-down speed. Higher = faster settle, less final refinement. |
| `warmup` | `true` / number / `false` | Headless settle ticks before first paint (`true` = 100 ticks under a 250ms budget). Lives in `layout`, not `animation`: warmup reduces motion, so it survives `animation: false`. |

## Animation

Graphs animate by default (`animation: true`). This is the opposite of charts, which are opt-in. Opt out with `animation: false`. The `animation` object (`GraphAnimationConfig`) tunes each phase; the shared ease vocabulary is `'smooth'` / `'snappy'`.

| Field | Default | Controls |
| --- | --- | --- |
| `enter` | `{ duration: 600, ease: 'smooth', stagger: true }` | Node/edge reveal on first render |
| `update` | `{ duration: 300, ease: 'smooth' }` | Enter-fade for nodes added via `update()` |
| `exit` | `{ duration: 300, ease: 'smooth' }` | Ghost fade-out for removed nodes/edges |
| `camera` | `{ duration: 'auto' }` | Camera-flight easing (`zoomToFit`/`zoomToNode`/`flyTo`) |
| `hover` | `{ duration: 150, ease: 'smooth' }` | Hover emphasis crossfade |

## Interaction

`interaction: GraphInteractionConfig` tunes hover/selection and opt-in physics feel.

| Field | Default | Effect |
| --- | --- | --- |
| `hover.mode` | `neighbors` | Hover emphasis: `neighbors` / `category` / `node` / `none` |
| `hover.dimOpacity` | `0.15` | Node dim tier for un-emphasized nodes |
| `select.flyTo` | `false` | Selecting a node flies the camera to it |
| `cursorRepulsion` | off | `true` or `{ radius, strength }`. Nodes drift from the pointer. Gated off under reduced-motion and above the node-count cap |
| `springyDrag` | off | Springy node drag. User-initiated, so no reduced-motion gate |

## Viewport Behavior

The graph canvas fills the full container height. Chrome (title, subtitle, source) overlays on top as an absolutely positioned element with `pointer-events: none`, so the canvas is interactive beneath it.

**Initial fit:** On the first simulation tick, the viewport fits all nodes using `ZoomTransform.fitBounds()`. After that, user interaction takes over and the simulation continues settling without affecting zoom/pan.

**Scale cap:** `fitBounds` caps scale at 1x (`Math.min(1, ...)`), so graphs never zoom in past their natural size. Small graphs center in the viewport at 1:1 scale; large graphs zoom out to fit.

**Spread heuristic:** For graphs with 50+ nodes, fitBounds applies a speculative bounding box expansion of `1 + sqrt(nodes) / 120` on the first tick. This prevents initial over-zoom while the force simulation is still settling. A 100-node graph gets ~8% expansion; a 2500-node graph gets ~21%. The heuristic only applies at the initial fit, not on subsequent `zoomToFit()` calls.

**Centering:** Both X and Y axes are centered, placing the graph in the middle of the viewport.

**Gesture mode:** During pan/zoom gestures, the renderer skips drawing node labels and glow effects for performance. Labels reappear 150ms after the gesture stops.

## Rendering

**React:**
```tsx
import { Graph } from '@opendata-ai/openchart-react';

<Graph
  spec={graphSpec}
  darkMode="auto"
  onNodeClick={(node) => console.log(node)}
  onNodeDoubleClick={(node) => console.log('double', node)}
  onSelectionChange={(nodeIds) => console.log(nodeIds)}
/>
```

**Vue:**
```vue
import { Graph } from '@opendata-ai/openchart-vue';

<Graph :spec="graphSpec" darkMode="auto" />
```

**Svelte:**
```svelte
import { Graph } from '@opendata-ai/openchart-svelte';

<Graph {spec} darkMode="auto" />
```

**Vanilla JS:**
```typescript
import { createGraph } from "@opendata-ai/openchart-vanilla";

const graph = createGraph(container, spec, {
  darkMode: "auto",
  responsive: true,
  onNodeClick: (node) => console.log(node),
  onNodeDoubleClick: (node) => console.log('double', node),
  onSelectionChange: (nodeIds) => console.log(nodeIds),
});

// Instance API
graph.update(newSpec);            // diff prev↔next; preserves positions on a visual-only change, local reheat on structural
graph.search("query");           // highlight matching nodes
graph.clearSearch();
graph.getSearchMatches();         // ids matching the active query
graph.highlight(target, opts);   // emphasize nodes: { nodeIds } | { category } | { search }; eased crossfade
graph.clearHighlight();
graph.getHighlight();             // currently highlighted ids, or null
graph.getLegend();                // headless legend snapshot (node + edge categories)
graph.zoomToFit(opts);           // fit all nodes; animated by default, { duration: 0 } snaps
graph.zoomToNode("node-1", opts);// fly to a node and zoom in
graph.flyTo({ x, y, k }, opts);  // fly the camera to a graph-space target
graph.centerAt(x, y, opts);      // center on a point, keep zoom
graph.getCamera();                // { x, y, k }
graph.selectNode("node-1", opts);// select a node ({ fly: true } also flies to it)
graph.getSelectedNodes();
graph.resize();
graph.destroy();
```

Camera methods animate by default. Pass `{ duration: 0 }` to snap. `update()` no longer resets the camera and preserves selection for surviving node ids.

## Event Handlers

| Handler | Signature | Fires when |
| --- | --- | --- |
| `onNodeClick` | `(node: Record<string, unknown>) => void` | Node clicked |
| `onNodeDoubleClick` | `(node: Record<string, unknown>) => void` | Node double-clicked |
| `onNodeHover` | `(node: Record<string, unknown> \| null) => void` | Node hover enter/leave |
| `onEdgeHover` | `(edge: Record<string, unknown> \| null) => void` | Edge hover enter/leave |
| `onSelectionChange` | `(nodeIds: string[]) => void` | Selection changes |
| `onLegendHover` | `(entry: { field: string; value: string } \| null) => void` | Legend entry hover |
| `onLegendToggle` | `(activeValues: string[]) => void` | Legend toggle state changes (empty = all) |
| `onHighlightChange` | `(nodeIds: string[] \| null) => void` | Highlight set changes (programmatic or legend) |
| `onCameraChange` | `(camera: { x: number; y: number; k: number }) => void` | Camera moves (rAF-coalesced) |

Options: `tooltip` accepts `boolean \| { formatter }`; `legend` accepts `boolean \| { interactive?, counts? }` (set `legend: false` if you render your own); `fitOnLoad` (default true). In React/Vue/Svelte, function-valued options like `tooltip.formatter` are read live per-render, so they can change without recreating the graph.

## Interactions

Graphs support these built-in interactions:
- **Pan:** Click and drag the canvas background
- **Zoom:** Scroll wheel or pinch gesture
- **Node drag:** Click and drag a node to reposition it
- **Select:** Click a node to select it
- **Keyboard:** Arrow keys to navigate between connected nodes, +/- to zoom, 0 to fit all

## Example

```json
{
  "type": "graph",
  "nodes": [
    { "id": "alice", "name": "Alice", "dept": "Engineering", "commits": 342 },
    { "id": "bob", "name": "Bob", "dept": "Engineering", "commits": 256 },
    { "id": "carol", "name": "Carol", "dept": "Design", "commits": 189 },
    { "id": "dave", "name": "Dave", "dept": "Design", "commits": 145 },
    { "id": "eve", "name": "Eve", "dept": "Product", "commits": 98 },
    { "id": "frank", "name": "Frank", "dept": "Product", "commits": 67 }
  ],
  "edges": [
    { "source": "alice", "target": "bob", "weight": 45 },
    { "source": "alice", "target": "carol", "weight": 23 },
    { "source": "bob", "target": "carol", "weight": 18 },
    { "source": "carol", "target": "dave", "weight": 31 },
    { "source": "dave", "target": "eve", "weight": 12 },
    { "source": "eve", "target": "frank", "weight": 28 },
    { "source": "alice", "target": "eve", "weight": 8 }
  ],
  "encoding": {
    "nodeColor": {
      "field": "dept",
      "type": "nominal",
      "scale": {
        "domain": ["Design", "Engineering", "Product"],
        "range": ["#3fb950", "#58a6ff", "#d29922"]
      }
    },
    "nodeSize": { "field": "commits", "type": "quantitative" },
    "edgeWidth": { "field": "weight", "type": "quantitative" },
    "nodeLabel": { "field": "name" }
  },
  "layout": {
    "type": "force",
    "clustering": { "field": "dept" }
  },
  "nodeOverrides": {
    "alice": { "fill": "#22c55e", "radius": 10, "alwaysShowLabel": true }
  },
  "chrome": {
    "title": "Engineering drives cross-team collaboration",
    "subtitle": "Code review connections between team members, weighted by review count",
    "source": "Source: GitHub activity data"
  }
}
```
