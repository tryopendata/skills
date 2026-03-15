# Responsive SVG

Making D3 visualizations adapt to container size and device screens.

## Contents

- viewBox and preserveAspectRatio
- Container-based sizing
- Resize event handling
- Aspect ratio strategies
- Mobile-first considerations

## viewBox and preserveAspectRatio

The `viewBox` attribute is the key to responsive SVGs. It defines a coordinate system independent of the actual display size.

### Basic setup

```typescript
const DESIGN_WIDTH = 600;
const DESIGN_HEIGHT = 400;

<svg
  viewBox={`0 0 ${DESIGN_WIDTH} ${DESIGN_HEIGHT}`}
  preserveAspectRatio="xMidYMid meet"
  style={{
    width: "100%",
    height: "auto",
    maxWidth: DESIGN_WIDTH,
  }}
>
  {/* All coordinates use the 600x400 system */}
</svg>
```

### How viewBox works

```
viewBox="minX minY width height"
```

- `minX, minY`: Top-left corner of the viewport (usually 0, 0)
- `width, height`: Coordinate system dimensions

The SVG scales to fit its container while maintaining internal proportions.

### preserveAspectRatio values

| Value | Behavior |
|-------|----------|
| `xMidYMid meet` | Center, scale to fit, show all content (recommended) |
| `xMidYMid slice` | Center, scale to fill, may crop |
| `xMinYMin meet` | Align top-left, scale to fit |
| `none` | Stretch to fill (distorts) |

**Almost always use `xMidYMid meet`.**

## Container-based sizing

### React hook for container dimensions

```typescript
import { useRef, useState, useEffect } from "react";

interface Dimensions {
  width: number;
  height: number;
}

function useContainerDimensions(): [React.RefObject<HTMLDivElement>, Dimensions] {
  const containerRef = useRef<HTMLDivElement>(null);
  const [dimensions, setDimensions] = useState<Dimensions>({ width: 0, height: 0 });

  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;

    const observer = new ResizeObserver((entries) => {
      const { width, height } = entries[0].contentRect;
      setDimensions({ width, height });
    });

    observer.observe(container);
    return () => observer.disconnect();
  }, []);

  return [containerRef, dimensions];
}

// Usage
function ChartWrapper() {
  const [containerRef, { width, height }] = useContainerDimensions();

  return (
    <div ref={containerRef} style={{ width: "100%", height: "400px" }}>
      {width > 0 && <Chart width={width} height={height} />}
    </div>
  );
}
```

### CSS container setup

```css
.chart-container {
  width: 100%;
  max-width: 800px;
  aspect-ratio: 16 / 9;  /* Modern browsers */
}

/* Fallback for older browsers */
.chart-container::before {
  content: "";
  display: block;
  padding-top: 56.25%;  /* 9/16 = 0.5625 */
}
```

## Resize event handling

### Debounced resize

```typescript
import { useCallback, useEffect, useState } from "react";

function useWindowSize() {
  const [size, setSize] = useState({
    width: typeof window !== "undefined" ? window.innerWidth : 0,
    height: typeof window !== "undefined" ? window.innerHeight : 0,
  });

  useEffect(() => {
    let timeoutId: NodeJS.Timeout;

    const handleResize = () => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => {
        setSize({
          width: window.innerWidth,
          height: window.innerHeight,
        });
      }, 150);  // Debounce 150ms
    };

    window.addEventListener("resize", handleResize);
    return () => {
      window.removeEventListener("resize", handleResize);
      clearTimeout(timeoutId);
    };
  }, []);

  return size;
}
```

### With D3 selections

```typescript
import { select } from "d3-selection";

function useResponsiveChart(svgRef: React.RefObject<SVGSVGElement>) {
  useEffect(() => {
    const svg = svgRef.current;
    if (!svg) return;

    const container = svg.parentElement;
    if (!container) return;

    const resize = () => {
      const { width } = container.getBoundingClientRect();
      select(svg)
        .attr("width", width)
        .attr("height", width * 0.625);  // 16:10 ratio
    };

    resize();
    window.addEventListener("resize", resize);
    return () => window.removeEventListener("resize", resize);
  }, [svgRef]);
}
```

## Aspect ratio strategies

### Fixed aspect ratio (recommended)

Design at one size, scale proportionally:

```typescript
const ASPECT_RATIO = 16 / 9;

function ResponsiveChart({ containerWidth }: { containerWidth: number }) {
  const height = containerWidth / ASPECT_RATIO;

  return (
    <svg
      viewBox={`0 0 ${containerWidth} ${height}`}
      style={{ width: "100%", height: "auto" }}
    >
      {/* Chart content */}
    </svg>
  );
}
```

### Responsive margins

Adjust margins based on available width:

```typescript
function getMargins(width: number) {
  if (width < 400) {
    return { top: 32, right: 16, bottom: 40, left: 40 };
  }
  if (width < 600) {
    return { top: 40, right: 32, bottom: 48, left: 48 };
  }
  return { top: 48, right: 48, bottom: 56, left: 64 };
}
```

### Responsive typography

```typescript
function getFontSizes(width: number) {
  const scale = Math.min(1, width / 600);

  return {
    headline: Math.max(20, Math.round(32 * scale)),
    dataLabel: Math.max(10, Math.round(14 * scale)),
    axisLabel: Math.max(9, Math.round(12 * scale)),
  };
}
```

## Mobile-first considerations

### Breakpoint-based rendering

```typescript
type Breakpoint = "mobile" | "tablet" | "desktop";

function getBreakpoint(width: number): Breakpoint {
  if (width < 480) return "mobile";
  if (width < 768) return "tablet";
  return "desktop";
}

function Chart({ width, height, data }: ChartProps) {
  const breakpoint = getBreakpoint(width);

  // Fewer labels on mobile
  const maxLabels = breakpoint === "mobile" ? 4 : breakpoint === "tablet" ? 6 : 10;
  const labelStep = Math.ceil(data.length / maxLabels);

  // Simpler chart on mobile
  const showAnnotations = breakpoint !== "mobile";

  return (
    <svg viewBox={`0 0 ${width} ${height}`}>
      {/* Only show every Nth label */}
      {data.filter((_, i) => i % labelStep === 0).map((d, i) => (
        <text key={i}>{d.label}</text>
      ))}

      {showAnnotations && <Annotations />}
    </svg>
  );
}
```

### Touch-friendly interactions

```typescript
// Larger touch targets on mobile
const pointRadius = breakpoint === "mobile" ? 12 : 6;
const hitAreaRadius = breakpoint === "mobile" ? 24 : 12;

<circle
  r={pointRadius}
  fill={palette.primary}
/>
<circle
  r={hitAreaRadius}
  fill="transparent"
  style={{ cursor: "pointer" }}
  onTouchStart={handleTouch}
  onClick={handleClick}
/>
```

### Horizontal scroll for wide charts

When a chart can't be made narrower:

```typescript
<div style={{ overflowX: "auto", WebkitOverflowScrolling: "touch" }}>
  <svg
    width={Math.max(containerWidth, MIN_CHART_WIDTH)}
    height={CHART_HEIGHT}
  >
    {/* Chart content */}
  </svg>
</div>
```

## Best practices

### Do

- Use `viewBox` for all SVGs
- Design at a fixed size, let it scale
- Test at multiple widths (320px, 768px, 1200px)
- Reduce label density on small screens
- Increase touch target sizes on mobile

### Don't

- Use fixed pixel dimensions without viewBox
- Assume a minimum width
- Show the same level of detail at all sizes
- Forget to debounce resize handlers
- Use hover-only interactions on touch devices

### Performance tip

Avoid re-rendering the entire chart on resize. With viewBox, scaling is handled by the browser:

```typescript
// Good: Let viewBox handle scaling
<svg viewBox="0 0 600 400" style={{ width: "100%" }}>

// Bad: Re-render on every resize
<svg width={containerWidth} height={containerHeight}>
```

If you need to re-render (e.g., for responsive labels), debounce the resize handler and use `useMemo` for expensive computations.
