# D3.js Core Patterns

Modern D3 v7 patterns for building visualizations in React/TypeScript.

## Contents

- Data binding with `.join()`
- Scale types and usage
- Axis generation
- Margin convention
- Common utilities

## Data binding with `.join()`

The modern D3 pattern replaces verbose enter/update/exit with a single `.join()` call.

### Basic join

```javascript
svg.selectAll("circle")
  .data(data)
  .join("circle")
  .attr("cx", d => xScale(d.x))
  .attr("cy", d => yScale(d.y))
  .attr("r", 5)
  .attr("fill", palette.primary);
```

### Join with transitions

```javascript
svg.selectAll("rect")
  .data(data, d => d.id)  // key function for object constancy
  .join(
    enter => enter
      .append("rect")
      .attr("x", d => xScale(d.category))
      .attr("y", innerHeight)  // start from bottom
      .attr("height", 0)
      .attr("width", xScale.bandwidth())
      .attr("fill", palette.primary)
      .call(enter => enter.transition()
        .duration(300)
        .attr("y", d => yScale(d.value))
        .attr("height", d => innerHeight - yScale(d.value))),
    update => update
      .call(update => update.transition()
        .duration(300)
        .attr("y", d => yScale(d.value))
        .attr("height", d => innerHeight - yScale(d.value))),
    exit => exit
      .call(exit => exit.transition()
        .duration(200)
        .attr("height", 0)
        .attr("y", innerHeight)
        .remove())
  );
```

### Key functions

Use key functions for stable updates when data changes:

```javascript
// Without key: elements matched by index (can cause jumpy updates)
.data(data)

// With key: elements matched by unique identifier
.data(data, d => d.id)

// Key from multiple fields
.data(data, d => `${d.category}-${d.year}`)
```

## Scale types

### Linear scale

For continuous numeric data.

```javascript
import { scaleLinear } from "d3-scale";

const yScale = scaleLinear()
  .domain([0, maxValue])  // data space
  .range([innerHeight, 0]);  // pixel space (inverted for SVG)

// Usage
yScale(50);  // returns pixel position
```

### Band scale

For categorical data with equal-width bands (bar charts).

```javascript
import { scaleBand } from "d3-scale";

const xScale = scaleBand()
  .domain(data.map(d => d.category))
  .range([0, innerWidth])
  .padding(0.2);  // 20% gap between bars

// Usage
xScale("Category A");  // returns left edge position
xScale.bandwidth();    // returns bar width
```

### Point scale

For categorical data as points (line charts with categories).

```javascript
import { scalePoint } from "d3-scale";

const xScale = scalePoint()
  .domain(data.map(d => d.label))
  .range([0, innerWidth])
  .padding(0.5);  // half-bandwidth at edges
```

### Time scale

For temporal data.

```javascript
import { scaleTime } from "d3-scale";
import { extent } from "d3-array";

const xScale = scaleTime()
  .domain(extent(data, d => d.date))
  .range([0, innerWidth]);
```

### Ordinal scale (colors)

For mapping categories to colors.

```javascript
import { scaleOrdinal } from "d3-scale";
import { schemeCategory10 } from "d3-scale-chromatic";

const colorScale = scaleOrdinal()
  .domain(["Series A", "Series B", "Series C"])
  .range(schemeCategory10);

// Or with custom colors
const colorScale = scaleOrdinal()
  .domain(categories)
  .range([palette.primary, ...palette.secondary]);
```

### Sqrt scale (bubble sizes)

For encoding values as area (bubble charts).

```javascript
import { scaleSqrt } from "d3-scale";

// Sqrt because area = πr², so we need sqrt for perceptually accurate sizing
const sizeScale = scaleSqrt()
  .domain([0, maxPopulation])
  .range([4, 40]);  // min and max radius
```

## Axis generation

### Basic axis

```javascript
import { axisBottom, axisLeft } from "d3-axis";

// X axis
const xAxis = axisBottom(xScale)
  .ticks(5)
  .tickSizeOuter(0);

svg.append("g")
  .attr("transform", `translate(0,${innerHeight})`)
  .call(xAxis);

// Y axis
const yAxis = axisLeft(yScale)
  .ticks(4)
  .tickFormat(d => `$${d}M`);

svg.append("g")
  .call(yAxis);
```

### Minimal axis (infographic style)

```javascript
// Remove axis line
svg.append("g")
  .call(axisLeft(yScale).tickSize(0))
  .call(g => g.select(".domain").remove());  // remove axis line

// Custom sparse labels (first, last, max only)
const keyValues = [data[0], data[data.length - 1], maxPoint];
keyValues.forEach(d => {
  svg.append("text")
    .attr("x", xScale(d.x))
    .attr("y", yScale(d.y) - 8)
    .text(formatNumber(d.value))
    .style("font-size", "14px")
    .style("font-weight", "600");
});
```

### Time axis formatting

```javascript
import { timeFormat } from "d3-time-format";

const xAxis = axisBottom(xScale)
  .tickFormat(timeFormat("%b %Y"))  // "Jan 2024"
  .ticks(6);
```

## Margin convention

Standard pattern for chart dimensions:

```javascript
const margin = { top: 48, right: 48, bottom: 48, left: 48 };
const innerWidth = width - margin.left - margin.right;
const innerHeight = height - margin.top - margin.bottom;

// Create SVG
const svg = d3.select(containerRef.current)
  .append("svg")
  .attr("width", width)
  .attr("height", height)
  .attr("viewBox", `0 0 ${width} ${height}`)
  .style("font-family", "Inter, system-ui, sans-serif");

// Create chart group (offset by margins)
const chart = svg.append("g")
  .attr("transform", `translate(${margin.left},${margin.top})`);

// Now use innerWidth/innerHeight for scales and chart elements
```

### Responsive with viewBox

```javascript
const svg = d3.select(containerRef.current)
  .append("svg")
  .attr("viewBox", `0 0 ${width} ${height}`)
  .attr("preserveAspectRatio", "xMidYMid meet")
  .style("width", "100%")
  .style("height", "auto");
```

## Common utilities

### Extent (min/max)

```javascript
import { extent, max, min } from "d3-array";

// Get [min, max] for scale domain
const [minVal, maxVal] = extent(data, d => d.value);

// Just max
const maxVal = max(data, d => d.value);

// With padding
const yDomain = [0, max(data, d => d.value) * 1.1];
```

### Number formatting

```javascript
import { format } from "d3-format";

const formatNumber = format(",.0f");     // 1,234
const formatMoney = format("$,.2f");     // $1,234.56
const formatPercent = format(".1%");     // 45.6%
const formatSI = format(".2s");          // 1.2M, 3.4B

// Smart abbreviation
function formatCompact(value) {
  if (Math.abs(value) >= 1e9) return format(".2s")(value).replace("G", "B");
  if (Math.abs(value) >= 1e6) return format(".2s")(value);
  if (Math.abs(value) >= 1e3) return format(",.0f")(value);
  return format(".1f")(value);
}
```

### Path generators

```javascript
import { line, area, curveMonotoneX } from "d3-shape";

// Line
const lineGenerator = line()
  .x(d => xScale(d.date))
  .y(d => yScale(d.value))
  .curve(curveMonotoneX);  // smooth curve

const pathD = lineGenerator(data);

// Area
const areaGenerator = area()
  .x(d => xScale(d.date))
  .y0(innerHeight)
  .y1(d => yScale(d.value))
  .curve(curveMonotoneX);

const areaD = areaGenerator(data);
```

## React integration

### Using refs

```typescript
import { useRef, useEffect } from "react";
import * as d3 from "d3";

function Chart({ data, width, height }) {
  const svgRef = useRef<SVGSVGElement>(null);

  useEffect(() => {
    if (!svgRef.current || !data.length) return;

    const svg = d3.select(svgRef.current);

    // Clear previous content
    svg.selectAll("*").remove();

    // Build chart...
  }, [data, width, height]);

  return <svg ref={svgRef} width={width} height={height} />;
}
```

### Pure SVG approach (recommended for infographics)

```typescript
import { useMemo } from "react";
import { scaleLinear, scaleBand } from "d3-scale";
import { line, curveMonotoneX } from "d3-shape";

function TrendLine({ data, width, height, palette }) {
  const { xScale, yScale, linePath } = useMemo(() => {
    const xScale = scaleBand()
      .domain(data.map(d => d.label))
      .range([0, innerWidth]);

    const yScale = scaleLinear()
      .domain([0, max(data, d => d.value)])
      .range([innerHeight, 0]);

    const lineGenerator = line()
      .x(d => xScale(d.label) + xScale.bandwidth() / 2)
      .y(d => yScale(d.value))
      .curve(curveMonotoneX);

    return { xScale, yScale, linePath: lineGenerator(data) };
  }, [data, innerWidth, innerHeight]);

  return (
    <svg width={width} height={height}>
      <path d={linePath} fill="none" stroke={palette.primary} strokeWidth={3} />
      {/* Labels, annotations, etc. as JSX */}
    </svg>
  );
}
```

The pure SVG approach is cleaner for static infographics. Use D3 selections when you need complex transitions or interactions.
