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
| nodeSize | Scale node radius by value (3-12px) | quantitative |
| edgeColor | Color edges by category or value | nominal, ordinal, quantitative |
| edgeWidth | Scale edge width by value (0.5-4px) | quantitative |
| edgeStyle | Map edge line style (solid/dashed/dotted) | nominal, ordinal |
| nodeLabel | Display text labels on nodes | any |

When `nodeColor` encoding is set, it takes precedence over community-based coloring from `layout.clustering`. Communities still affect spatial grouping, but colors come from the encoding.

## Node Overrides

`nodeOverrides` is `Record<string, NodeOverride>` keyed by node id — useful for highlighting seed nodes. Override fields: `fill`, `radius`, `strokeWidth`, `stroke`, `alwaysShowLabel`. Load `NodeOverride` from `index.d.ts` for the exact shape.

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
graph.update(newSpec);       // re-render with new spec
graph.search("query");      // highlight matching nodes
graph.clearSearch();         // clear search highlights
graph.zoomToFit();           // fit all nodes in viewport
graph.zoomToNode("node-1");  // zoom to specific node
graph.selectNode("node-1");  // select a node
graph.getSelectedNodes();    // get selected node ids
graph.resize();              // recalculate dimensions
graph.destroy();             // cleanup
```

## Event Handlers

| Handler | Signature | Fires when |
| --- | --- | --- |
| `onNodeClick` | `(node: Record<string, unknown>) => void` | Node clicked |
| `onNodeDoubleClick` | `(node: Record<string, unknown>) => void` | Node double-clicked |
| `onSelectionChange` | `(nodeIds: string[]) => void` | Selection changes |

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
