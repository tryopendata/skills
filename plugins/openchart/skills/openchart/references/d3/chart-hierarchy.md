# Hierarchy and Flow Charts

Treemaps, sunbursts, and sankey diagrams for hierarchical and flow data.

## Contents

- When to use
- Treemap implementation
- Sankey diagrams
- Data structure requirements
- Labeling strategies

## When to use

| Chart | Data Shape | Best For |
|-------|------------|----------|
| Treemap | Nested categories + values | Part-to-whole with hierarchy |
| Sunburst | Nested categories + values | Hierarchy with radial layout |
| Sankey | Flows between stages | How values move through a system |

**Treemap:** "What's the breakdown of X by category and subcategory?"
**Sankey:** "Where does X come from and where does it go?"

## Treemap implementation

```typescript
import { useMemo } from "react";
import { treemap, hierarchy, treemapSquarify } from "d3-hierarchy";
import { scaleOrdinal } from "d3-scale";

interface TreemapNode {
  name: string;
  value?: number;
  children?: TreemapNode[];
}

interface TreemapChartProps {
  data: TreemapNode;
  width: number;
  height: number;
  palette: ChartPalette;
}

const MARGIN = { top: 40, right: 10, bottom: 10, left: 10 };

export function TreemapChart({ data, width, height, palette }: TreemapChartProps) {
  const innerWidth = width - MARGIN.left - MARGIN.right;
  const innerHeight = height - MARGIN.top - MARGIN.bottom;

  const { root, colorScale } = useMemo(() => {
    // Build hierarchy
    const root = hierarchy(data)
      .sum(d => d.value ?? 0)
      .sort((a, b) => (b.value ?? 0) - (a.value ?? 0));

    // Apply treemap layout
    treemap<TreemapNode>()
      .size([innerWidth, innerHeight])
      .padding(2)
      .round(true)
      .tile(treemapSquarify)(root);

    // Color scale for top-level categories
    const topLevelNames = root.children?.map(d => d.data.name) ?? [];
    const colorScale = scaleOrdinal<string>()
      .domain(topLevelNames)
      .range([palette.primary, ...palette.secondary]);

    return { root, colorScale };
  }, [data, innerWidth, innerHeight, palette]);

  // Get leaves (the actual rectangles to draw)
  const leaves = root.leaves();

  return (
    <svg width={width} height={height} style={{ fontFamily: "Inter, sans-serif" }}>
      <rect width={width} height={height} fill={palette.background} />

      <g transform={`translate(${MARGIN.left},${MARGIN.top})`}>
        {leaves.map((leaf) => {
          const x0 = leaf.x0 ?? 0;
          const y0 = leaf.y0 ?? 0;
          const x1 = leaf.x1 ?? 0;
          const y1 = leaf.y1 ?? 0;
          const rectWidth = x1 - x0;
          const rectHeight = y1 - y0;

          // Get top-level ancestor for color
          let topAncestor = leaf;
          while (topAncestor.depth > 1 && topAncestor.parent) {
            topAncestor = topAncestor.parent;
          }

          const showLabel = rectWidth > 40 && rectHeight > 20;

          return (
            <g key={leaf.data.name}>
              <rect
                x={x0}
                y={y0}
                width={rectWidth}
                height={rectHeight}
                fill={colorScale(topAncestor.data.name)}
                stroke={palette.background}
                strokeWidth={1}
                rx={2}
              />
              {showLabel && (
                <>
                  <text
                    x={x0 + 6}
                    y={y0 + 16}
                    fill={palette.background}
                    fontSize={12}
                    fontWeight={500}
                    style={{
                      textShadow: "0 1px 2px rgba(0,0,0,0.3)",
                    }}
                  >
                    {leaf.data.name.length > rectWidth / 8
                      ? leaf.data.name.slice(0, Math.floor(rectWidth / 8)) + "…"
                      : leaf.data.name}
                  </text>
                  {rectHeight > 36 && (
                    <text
                      x={x0 + 6}
                      y={y0 + 32}
                      fill={palette.background}
                      fontSize={11}
                      fillOpacity={0.8}
                      style={{ fontFeatureSettings: '"tnum"' }}
                    >
                      {leaf.value?.toLocaleString()}
                    </text>
                  )}
                </>
              )}
            </g>
          );
        })}
      </g>
    </svg>
  );
}
```

### Data structure

Treemap expects hierarchical data:

```typescript
const data: TreemapNode = {
  name: "Root",
  children: [
    {
      name: "Category A",
      children: [
        { name: "Subcategory A1", value: 100 },
        { name: "Subcategory A2", value: 50 },
      ],
    },
    {
      name: "Category B",
      value: 75,  // Leaf node with value
    },
  ],
};
```

### Transforming flat data to hierarchy

```typescript
// From: [{ category: "Tech", subcategory: "Apple", value: 100 }, ...]
// To: TreemapNode hierarchy

function buildHierarchy(flatData: FlatRow[]): TreemapNode {
  const nested = new Map<string, Map<string, number>>();

  flatData.forEach(row => {
    if (!nested.has(row.category)) {
      nested.set(row.category, new Map());
    }
    nested.get(row.category)!.set(row.subcategory, row.value);
  });

  return {
    name: "Root",
    children: Array.from(nested.entries()).map(([category, subs]) => ({
      name: category,
      children: Array.from(subs.entries()).map(([sub, value]) => ({
        name: sub,
        value,
      })),
    })),
  };
}
```

## Sankey diagrams

**For standard sankey usage, see [references/sankey.md](../sankey.md).** Sankey is a first-class OpenChart type with its own declarative spec. The D3 implementation below is for advanced/custom layouts that go beyond what the declarative spec supports.

Show flows between stages. Requires the `d3-sankey` library.

```typescript
import { sankey, sankeyLinkHorizontal } from "d3-sankey";

interface SankeyNode {
  name: string;
}

interface SankeyLink {
  source: number | string;  // Node index or name
  target: number | string;
  value: number;
}

interface SankeyData {
  nodes: SankeyNode[];
  links: SankeyLink[];
}

interface SankeyChartProps {
  data: SankeyData;
  width: number;
  height: number;
  palette: ChartPalette;
}

const MARGIN = { top: 20, right: 100, bottom: 20, left: 100 };

export function SankeyChart({ data, width, height, palette }: SankeyChartProps) {
  const innerWidth = width - MARGIN.left - MARGIN.right;
  const innerHeight = height - MARGIN.top - MARGIN.bottom;

  const { nodes, links } = useMemo(() => {
    const sankeyGenerator = sankey<SankeyNode, SankeyLink>()
      .nodeId(d => d.name)
      .nodeWidth(24)
      .nodePadding(16)
      .extent([[0, 0], [innerWidth, innerHeight]]);

    return sankeyGenerator({
      nodes: data.nodes.map(d => ({ ...d })),
      links: data.links.map(d => ({ ...d })),
    });
  }, [data, innerWidth, innerHeight]);

  const colorScale = scaleOrdinal<string>()
    .domain(data.nodes.map(d => d.name))
    .range([palette.primary, ...palette.secondary, palette.positive, palette.negative]);

  return (
    <svg width={width} height={height} style={{ fontFamily: "Inter, sans-serif" }}>
      <rect width={width} height={height} fill={palette.background} />

      <g transform={`translate(${MARGIN.left},${MARGIN.top})`}>
        {/* Links */}
        {links.map((link, i) => (
          <path
            key={i}
            d={sankeyLinkHorizontal()(link) ?? ""}
            fill="none"
            stroke={colorScale((link.source as any).name)}
            strokeOpacity={0.4}
            strokeWidth={Math.max(1, link.width ?? 1)}
          />
        ))}

        {/* Nodes */}
        {nodes.map((node) => (
          <g key={node.name}>
            <rect
              x={node.x0}
              y={node.y0}
              width={(node.x1 ?? 0) - (node.x0 ?? 0)}
              height={(node.y1 ?? 0) - (node.y0 ?? 0)}
              fill={colorScale(node.name)}
              rx={2}
            />
            <text
              x={(node.x0 ?? 0) < innerWidth / 2 ? (node.x1 ?? 0) + 8 : (node.x0 ?? 0) - 8}
              y={((node.y0 ?? 0) + (node.y1 ?? 0)) / 2}
              textAnchor={(node.x0 ?? 0) < innerWidth / 2 ? "start" : "end"}
              dominantBaseline="middle"
              fill={palette.text}
              fontSize={12}
            >
              {node.name}
            </text>
          </g>
        ))}
      </g>
    </svg>
  );
}
```

### Sankey data structure

```typescript
const sankeyData: SankeyData = {
  nodes: [
    { name: "Source A" },
    { name: "Source B" },
    { name: "Process 1" },
    { name: "Process 2" },
    { name: "Output X" },
    { name: "Output Y" },
  ],
  links: [
    { source: "Source A", target: "Process 1", value: 100 },
    { source: "Source A", target: "Process 2", value: 50 },
    { source: "Source B", target: "Process 1", value: 30 },
    { source: "Process 1", target: "Output X", value: 80 },
    { source: "Process 1", target: "Output Y", value: 50 },
    { source: "Process 2", target: "Output Y", value: 50 },
  ],
};
```

## Labeling strategies

### Treemap labels

| Rect Size | Label Strategy |
|-----------|----------------|
| Large (>100x40) | Name + value inside |
| Medium (40-100 wide) | Name only, truncated |
| Small (<40 wide) | No label, show on hover |

```typescript
const showLabel = rectWidth > 40 && rectHeight > 20;
const showValue = rectWidth > 60 && rectHeight > 36;
const maxChars = Math.floor(rectWidth / 8);

{showLabel && (
  <text>
    {name.length > maxChars ? name.slice(0, maxChars - 1) + "…" : name}
  </text>
)}
```

### Sankey labels

Position labels outside nodes:
- Left nodes: label on the left
- Right nodes: label on the right
- Middle nodes: below or above

```typescript
const isLeft = (node.x0 ?? 0) < innerWidth / 3;
const isRight = (node.x0 ?? 0) > (innerWidth * 2) / 3;

const labelX = isLeft
  ? (node.x0 ?? 0) - 8
  : isRight
  ? (node.x1 ?? 0) + 8
  : ((node.x0 ?? 0) + (node.x1 ?? 0)) / 2;

const textAnchor = isLeft ? "end" : isRight ? "start" : "middle";
```

## Interaction patterns

### Treemap drill-down

```typescript
const [focusedNode, setFocusedNode] = useState<TreemapNode | null>(null);

const displayData = focusedNode ?? data;

// Add click handler to zoom into a category
<rect
  onClick={() => {
    if (leaf.children) {
      setFocusedNode(leaf.data);
    }
  }}
  style={{ cursor: leaf.children ? "pointer" : "default" }}
/>

// Breadcrumb to navigate back
{focusedNode && (
  <text
    x={MARGIN.left}
    y={24}
    onClick={() => setFocusedNode(null)}
    style={{ cursor: "pointer" }}
    fill={palette.primary}
    fontSize={12}
  >
    ← Back to overview
  </text>
)}
```

### Sankey highlight on hover

```typescript
const [hoveredNode, setHoveredNode] = useState<string | null>(null);

// Highlight connected links
const isConnected = (link: SankeyLink) =>
  (link.source as any).name === hoveredNode ||
  (link.target as any).name === hoveredNode;

<path
  strokeOpacity={hoveredNode ? (isConnected(link) ? 0.7 : 0.1) : 0.4}
/>
```

## When not to use

### Treemap

- Few categories (use bar chart)
- No meaningful hierarchy (use pie/bar)
- Exact comparison needed (area perception is imprecise)

### Sankey

- Simple two-stage flow (use grouped bar)
- Circular flows (sankey requires acyclic graphs)
- Many crossing links (becomes unreadable)
