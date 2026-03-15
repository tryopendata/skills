# Graph (Network Visualization)

Network and relationship visualizations using nodes and edges. Rendered on canvas with force-directed, radial, or hierarchical layouts. Supports zoom, pan, node dragging, search, and selection.

## Data Model

Unlike charts which use flat `data` arrays with encoding channels, graphs use a **nodes + edges** model:

```typescript
// Each node must have an `id`, plus any data fields
interface GraphNode {
  id: string;
  [key: string]: unknown;  // arbitrary data: label, group, weight, etc.
}

// Each edge connects two nodes by id
interface GraphEdge {
  source: string;           // source node id
  target: string;           // target node id
  [key: string]: unknown;   // arbitrary data: weight, type, etc.
}
```

## GraphSpec

```typescript
{
  type: "graph",
  nodes: GraphNode[],          // REQUIRED: array of {id, ...data}
  edges: GraphEdge[],          // REQUIRED: array of {source, target, ...data}
  encoding?: GraphEncoding,    // visual mappings for nodes/edges
  layout?: GraphLayoutConfig,  // force, radial, or hierarchical
  chrome?: Chrome,             // title, subtitle, source, etc.
  annotations?: Annotation[],
  theme?: ThemeConfig,
  darkMode?: DarkMode,
}
```

## Graph Encoding

Graph encoding channels are different from chart encoding channels. All are optional since a graph renders fine with uniform appearance.

```typescript
encoding?: {
  nodeColor?: { field: string, type?: "nominal"|"ordinal" },
  nodeSize?: { field: string, type?: "quantitative" },
  edgeColor?: { field: string, type?: "nominal"|"ordinal" },
  edgeWidth?: { field: string, type?: "quantitative" },
  nodeLabel?: { field: string, type?: FieldType },
}
```

| Channel | Effect | Type constraint |
| --- | --- | --- |
| nodeColor | Color nodes by category | nominal, ordinal |
| nodeSize | Scale node radius by value (3-20px) | quantitative |
| edgeColor | Color edges by category | nominal, ordinal |
| edgeWidth | Scale edge width by value (0.5-4px) | quantitative |
| nodeLabel | Display text labels on nodes | any |

## Layout Configuration

```typescript
layout?: {
  type: "force" | "radial" | "hierarchical",
  clustering?: { field: string },  // group nodes by field for cluster forces
  chargeStrength?: number,         // repulsion (negative values, default varies)
  linkDistance?: number,            // target distance between linked nodes
}
```

| Layout | Best for | Notes |
| --- | --- | --- |
| `force` | General networks | Default. Nodes repel, edges attract. Good for most graphs. |
| `radial` | Hub-and-spoke | Arranges nodes in concentric circles. |
| `hierarchical` | Trees, DAGs | Top-down or left-right arrangement. |

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
    "nodeColor": { "field": "dept", "type": "nominal" },
    "nodeSize": { "field": "commits", "type": "quantitative" },
    "edgeWidth": { "field": "weight", "type": "quantitative" },
    "nodeLabel": { "field": "name" }
  },
  "layout": {
    "type": "force",
    "clustering": { "field": "dept" }
  },
  "chrome": {
    "title": "Engineering drives cross-team collaboration",
    "subtitle": "Code review connections between team members, weighted by review count",
    "source": "Source: GitHub activity data"
  }
}
```
