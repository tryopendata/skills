# Advanced Visualization Techniques

Boundary-pushing patterns for creating visualizations that feel closer to art than charts.

## Contents

- Path morphing with Flubber
- Animated gradients for flow effects
- Custom interpolators
- Generative/organic shapes
- Scrollytelling patterns
- GPU-optimized animation
- Creative coding patterns

## Path morphing with Flubber

Standard D3 path interpolation fails when shapes have different numbers of points. [Flubber](https://github.com/veltman/flubber) provides smooth morphing between arbitrary shapes.

### Installation

```bash
npm install flubber
```

### Basic morphing

```typescript
import { interpolate } from "flubber";

// Define two shapes (SVG path strings or coordinate arrays)
const circle = "M100,50 a50,50 0 1,0 100,0 a50,50 0 1,0 -100,0";
const square = "M50,50 L150,50 L150,150 L50,150 Z";

// Create interpolator (0 = circle, 1 = square)
const morpher = interpolate(circle, square, { maxSegmentLength: 10 });

// Use with D3 transition
d3.select("path")
  .transition()
  .duration(800)
  .attrTween("d", () => t => morpher(t));
```

### Shape to circle/rectangle

```typescript
import { toCircle, toRect } from "flubber";

// Morph any shape to a circle
const toCircleInterpolator = toCircle(pathString, cx, cy, radius);

// Morph any shape to a rectangle
const toRectInterpolator = toRect(pathString, x, y, width, height);
```

### Multiple shapes (explode/combine)

```typescript
import { combine, separate } from "flubber";

// Combine multiple shapes into one (implode effect)
const combineInterpolator = combine(
  [shape1, shape2, shape3],  // Array of shapes
  targetShape,               // Final combined shape
  { single: true }
);

// Separate one shape into multiple (explode effect)
const separateInterpolator = separate(
  sourceShape,
  [target1, target2, target3]
);
```

### React component with morphing

```typescript
import { useState, useEffect } from "react";
import { interpolate } from "flubber";

function MorphingShape({ fromPath, toPath, progress }) {
  const [currentPath, setCurrentPath] = useState(fromPath);

  useEffect(() => {
    const morpher = interpolate(fromPath, toPath, { maxSegmentLength: 10 });
    setCurrentPath(morpher(progress));
  }, [fromPath, toPath, progress]);

  return <path d={currentPath} fill="currentColor" />;
}
```

## Animated gradients for flow effects

Create the illusion of flowing data using animated SVG gradients.

### Setup

```typescript
// Create gradient in defs
const defs = svg.append("defs");

const gradient = defs.append("linearGradient")
  .attr("id", "flow-gradient")
  .attr("x1", "0%")
  .attr("y1", "0%")
  .attr("x2", "100%")
  .attr("y2", "0%")
  .attr("spreadMethod", "reflect");  // Key: creates seamless loop

// Create symmetrical color stops (first = last, second = second-to-last)
const colors = ["#2563EB", "#60A5FA", "#93C5FD", "#60A5FA", "#2563EB"];
colors.forEach((color, i) => {
  gradient.append("stop")
    .attr("offset", `${(i / (colors.length - 1)) * 100}%`)
    .attr("stop-color", color);
});
```

### Animation with SVG animate element

```typescript
// Animate x1 from 0% to 100%
gradient.append("animate")
  .attr("attributeName", "x1")
  .attr("values", "0%;100%")
  .attr("dur", "4s")
  .attr("repeatCount", "indefinite");

// Animate x2 simultaneously (maintain 100% offset)
gradient.append("animate")
  .attr("attributeName", "x2")
  .attr("values", "100%;200%")
  .attr("dur", "4s")
  .attr("repeatCount", "indefinite");
```

### Apply to path

```typescript
svg.append("path")
  .attr("d", pathData)
  .attr("fill", "none")
  .attr("stroke", "url(#flow-gradient)")
  .attr("stroke-width", 4);
```

### Safari fallback

Safari has limited `spreadMethod` support. Duplicate color stops manually:

```typescript
// Manually create reflected pattern
const colors = ["#2563EB", "#60A5FA", "#93C5FD"];
const reflected = [...colors, ...colors.slice(0, -1).reverse()];
```

## Custom interpolators

For advanced property tweening beyond D3's defaults.

### attrTween basics

```typescript
// Custom attribute interpolation
selection.transition()
  .attrTween("d", function() {
    const start = this.getAttribute("d");
    const end = newPathData;
    return t => {
      // Custom interpolation logic
      return interpolatePath(start, end)(t);
    };
  });
```

### Color interpolation with HSL

```typescript
import { interpolateHsl } from "d3-interpolate";

selection.transition()
  .attrTween("fill", function() {
    const start = this.getAttribute("fill");
    return interpolateHsl(start, "#F97316");
  });
```

### Multi-property interpolation

```typescript
// Interpolate multiple properties in sync
selection.transition()
  .duration(500)
  .tween("custom", function() {
    const node = this;
    const startX = parseFloat(node.getAttribute("cx"));
    const startR = parseFloat(node.getAttribute("r"));
    const endX = xScale(newData.x);
    const endR = sizeScale(newData.size);

    return t => {
      node.setAttribute("cx", startX + (endX - startX) * t);
      node.setAttribute("r", startR + (endR - startR) * t);
    };
  });
```

### Data-driven interpolation

```typescript
import { interpolateObject } from "d3-interpolate";

// Interpolate entire data objects
const dataInterpolator = interpolateObject(
  { value: 100, x: 0, y: 0 },
  { value: 250, x: 100, y: 50 }
);

selection.transition()
  .tween("data", function() {
    return t => {
      const interpolatedData = dataInterpolator(t);
      // Update visual based on interpolated data
      d3.select(this)
        .attr("cx", xScale(interpolatedData.x))
        .attr("cy", yScale(interpolatedData.y))
        .attr("r", sizeScale(interpolatedData.value));
    };
  });
```

## Generative/organic shapes

Create visualizations that feel alive using algorithmic techniques.

### Noise-based variation

```typescript
// Using simplex noise for organic movement
import { createNoise2D } from "simplex-noise";

const noise = createNoise2D();

function generateOrganicPath(points: number, radius: number, time: number) {
  const path: [number, number][] = [];

  for (let i = 0; i < points; i++) {
    const angle = (i / points) * Math.PI * 2;
    const noiseValue = noise(Math.cos(angle) + time, Math.sin(angle) + time);
    const r = radius + noiseValue * 20;  // Vary radius by noise

    path.push([
      Math.cos(angle) * r + centerX,
      Math.sin(angle) * r + centerY
    ]);
  }

  return path;
}
```

### Parametric shapes

```typescript
// Generate shapes from mathematical functions
function rose(a: number, n: number, d: number) {
  const points: [number, number][] = [];
  const k = n / d;

  for (let theta = 0; theta < Math.PI * 2 * d; theta += 0.01) {
    const r = a * Math.cos(k * theta);
    points.push([
      r * Math.cos(theta),
      r * Math.sin(theta)
    ]);
  }

  return points;
}

// Lissajous curves
function lissajous(a: number, b: number, delta: number) {
  const points: [number, number][] = [];

  for (let t = 0; t < Math.PI * 2; t += 0.01) {
    points.push([
      Math.sin(a * t + delta) * 100,
      Math.sin(b * t) * 100
    ]);
  }

  return points;
}
```

### Particle systems

```typescript
interface Particle {
  x: number;
  y: number;
  vx: number;
  vy: number;
  life: number;
}

function createParticleSystem(count: number) {
  const particles: Particle[] = [];

  for (let i = 0; i < count; i++) {
    particles.push({
      x: Math.random() * width,
      y: Math.random() * height,
      vx: (Math.random() - 0.5) * 2,
      vy: (Math.random() - 0.5) * 2,
      life: Math.random()
    });
  }

  function update() {
    particles.forEach(p => {
      p.x += p.vx;
      p.y += p.vy;
      p.life -= 0.01;

      // Wrap around edges
      if (p.x < 0) p.x = width;
      if (p.x > width) p.x = 0;
      if (p.y < 0) p.y = height;
      if (p.y > height) p.y = 0;

      // Reset dead particles
      if (p.life <= 0) {
        p.life = 1;
        p.x = Math.random() * width;
        p.y = Math.random() * height;
      }
    });
  }

  return { particles, update };
}
```

## Scrollytelling patterns

Reveal visualizations as users scroll, creating narrative-driven experiences.

### Intersection Observer setup

```typescript
function useScrollReveal(threshold = 0.3) {
  const ref = useRef<HTMLDivElement>(null);
  const [isVisible, setIsVisible] = useState(false);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();  // Only trigger once
        }
      },
      { threshold }
    );

    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [threshold]);

  return [ref, isVisible] as const;
}

// Usage
function ScrollChart({ data }) {
  const [ref, isVisible] = useScrollReveal(0.3);

  return (
    <div ref={ref}>
      <Chart data={data} animate={isVisible} />
    </div>
  );
}
```

### Scroll progress tracking

```typescript
function useScrollProgress(ref: RefObject<HTMLElement>) {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const handleScroll = () => {
      const rect = element.getBoundingClientRect();
      const viewportHeight = window.innerHeight;

      // Progress from 0 (element enters viewport) to 1 (element exits)
      const elementTop = rect.top;
      const elementHeight = rect.height;

      // Calculate how far through the element we've scrolled
      const start = viewportHeight;  // Element enters
      const end = -elementHeight;     // Element exits
      const current = elementTop;

      const progress = Math.max(0, Math.min(1,
        (start - current) / (start - end)
      ));

      setProgress(progress);
    };

    window.addEventListener("scroll", handleScroll, { passive: true });
    handleScroll();  // Initial calculation

    return () => window.removeEventListener("scroll", handleScroll);
  }, [ref]);

  return progress;
}
```

### Sequenced narrative reveals

```typescript
function NarrativeChart({ data }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const progress = useScrollProgress(containerRef);

  // Define narrative steps
  const steps = [
    { progress: 0.0, action: "show-axes" },
    { progress: 0.2, action: "reveal-data" },
    { progress: 0.4, action: "highlight-max" },
    { progress: 0.6, action: "show-annotation" },
    { progress: 0.8, action: "reveal-trend" },
  ];

  // Determine current step
  const currentStep = steps.reduce((acc, step) =>
    progress >= step.progress ? step : acc
  , steps[0]);

  return (
    <div ref={containerRef} style={{ minHeight: "200vh" }}>
      <div style={{ position: "sticky", top: "20vh" }}>
        <Chart
          data={data}
          showAxes={progress >= 0.0}
          showData={progress >= 0.2}
          highlightMax={progress >= 0.4}
          showAnnotation={progress >= 0.6}
          showTrend={progress >= 0.8}
        />
      </div>
    </div>
  );
}
```

## GPU-optimized animation

For smooth 60fps animations, only animate properties that don't trigger layout recalculation.

### GPU-accelerated properties

```typescript
// GOOD: GPU-accelerated (compositor thread)
.transition()
.style("transform", "translateX(100px)")
.style("opacity", 0.5);

// BAD: Triggers repaint (main thread)
.transition()
.attr("x", 100)
.attr("fill", "#ff0000");
```

### Transform-based animation

```typescript
// Instead of animating cx/cy, use transform
svg.selectAll("circle")
  .data(data)
  .join("circle")
  .attr("cx", 0)  // Base position
  .attr("cy", 0)
  .attr("r", 5)
  .style("transform", d => `translate(${xScale(d.x)}px, ${yScale(d.y)}px)`)
  .transition()
  .duration(300)
  .style("transform", d => `translate(${xScale(d.newX)}px, ${yScale(d.newY)}px)`);
```

### will-change hint

```css
.animated-element {
  will-change: transform, opacity;
}
```

```typescript
// Apply hint before animation
selection.style("will-change", "transform");

selection.transition()
  .style("transform", "translateX(100px)")
  .on("end", function() {
    // Remove hint after animation
    d3.select(this).style("will-change", "auto");
  });
```

### Reduced motion support

```typescript
// Check user preference
const prefersReducedMotion = window.matchMedia(
  "(prefers-reduced-motion: reduce)"
).matches;

// Adjust animation accordingly
const duration = prefersReducedMotion ? 0 : 400;

selection.transition()
  .duration(duration)
  .attr("opacity", 1);
```

```css
/* CSS fallback */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Creative coding patterns

### Data as texture

```typescript
// Generate a pattern from data values
function dataToPattern(data: number[], width: number, height: number) {
  const canvas = document.createElement("canvas");
  canvas.width = width;
  canvas.height = height;
  const ctx = canvas.getContext("2d")!;

  data.forEach((value, i) => {
    const x = (i % Math.sqrt(data.length)) * (width / Math.sqrt(data.length));
    const y = Math.floor(i / Math.sqrt(data.length)) * (height / Math.sqrt(data.length));
    const brightness = value / Math.max(...data);

    ctx.fillStyle = `rgba(0, 0, 0, ${brightness})`;
    ctx.fillRect(x, y, width / Math.sqrt(data.length), height / Math.sqrt(data.length));
  });

  return canvas.toDataURL();
}
```

### Voronoi-based visualization

```typescript
import { Delaunay } from "d3-delaunay";

function VoronoiChart({ points, width, height }) {
  const delaunay = Delaunay.from(points.map(p => [p.x, p.y]));
  const voronoi = delaunay.voronoi([0, 0, width, height]);

  return (
    <svg width={width} height={height}>
      {points.map((point, i) => (
        <path
          key={i}
          d={voronoi.renderCell(i)}
          fill={colorScale(point.value)}
          stroke="#fff"
          strokeWidth={1}
        />
      ))}
    </svg>
  );
}
```

### Contour visualization

```typescript
import { contours, geoPath } from "d3";

function ContourMap({ values, width, height, thresholds }) {
  const contourGenerator = contours()
    .size([width, height])
    .thresholds(thresholds);

  const contourData = contourGenerator(values);
  const pathGenerator = geoPath();

  return (
    <svg width={width} height={height}>
      {contourData.map((contour, i) => (
        <path
          key={i}
          d={pathGenerator(contour)}
          fill={colorScale(contour.value)}
          fillOpacity={0.3}
          stroke={colorScale(contour.value)}
          strokeWidth={1}
        />
      ))}
    </svg>
  );
}
```

## Related references

- [d3-core-patterns.md](d3-core-patterns.md) - Basic D3 patterns
- [animation-transitions.md](animation-transitions.md) - Standard animation timing
- [infographic-design.md](infographic-design.md) - When to use these techniques (sparingly, with purpose)
