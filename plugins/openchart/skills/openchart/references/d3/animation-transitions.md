# Animation and Transitions

Motion design for data visualization storytelling.

## Contents

- D3 transition basics
- Timing and easing
- Staggered entrance effects
- Data update animations
- Scroll-triggered reveals
- Performance considerations

## D3 transition basics

### Simple transition

```typescript
import { select } from "d3-selection";
import { transition } from "d3-transition";

// Apply transition to a selection
select(element)
  .transition()
  .duration(300)
  .attr("opacity", 1)
  .attr("y", finalY);
```

### Chaining transitions

```typescript
select(element)
  .transition()
  .duration(200)
  .attr("opacity", 1)
  .transition()  // Chains after previous completes
  .duration(300)
  .attr("y", finalY);
```

### Named transitions

```typescript
// Named transitions can run in parallel
select(element)
  .transition("fade")
  .duration(200)
  .attr("opacity", 1);

select(element)
  .transition("move")  // Different name = parallel
  .duration(300)
  .attr("y", finalY);
```

## Timing and easing

### Duration guidelines

| Animation Type | Duration | Use Case |
|----------------|----------|----------|
| Micro-interaction | 100-200ms | Hover, focus states |
| Element transition | 200-400ms | Enter/exit, state change |
| Coordinated group | 400-600ms | Multiple elements together |
| Page-level | 600-1000ms | Major view changes |

### Easing functions

```typescript
import {
  easeCubicOut,    // Fast start, slow end (recommended for entrances)
  easeCubicIn,     // Slow start, fast end (for exits)
  easeCubicInOut,  // Smooth both ends
  easeLinear,      // Constant speed
  easeElasticOut,  // Bouncy (use sparingly)
  easeBackOut,     // Slight overshoot
} from "d3-ease";

select(element)
  .transition()
  .duration(300)
  .ease(easeCubicOut)  // Apply easing
  .attr("y", finalY);
```

### When to use each easing

| Easing | Effect | Use For |
|--------|--------|---------|
| `easeCubicOut` | Decelerates | Element entrances, settling into position |
| `easeCubicIn` | Accelerates | Element exits, disappearing |
| `easeCubicInOut` | Smooth | State changes, morphing |
| `easeLinear` | Constant | Progress bars, continuous motion |
| `easeBackOut` | Overshoot | Playful entrances (use sparingly) |

## Staggered entrance effects

Staggering creates visual hierarchy and guides attention.

### Index-based delay

```typescript
svg.selectAll("rect")
  .data(data)
  .join("rect")
  .attr("y", innerHeight)  // Start at bottom
  .attr("height", 0)       // Start collapsed
  .transition()
  .duration(400)
  .delay((_, i) => i * 50)  // 50ms between each bar
  .ease(easeCubicOut)
  .attr("y", d => yScale(d.value))
  .attr("height", d => innerHeight - yScale(d.value));
```

### Value-based delay

```typescript
// Larger values animate first (for rankings)
.delay((d, i) => {
  const rank = data.length - i - 1;  // Reverse: biggest first
  return rank * 40;
})
```

### Group-based stagger

```typescript
// Stagger by category, then by item within category
.delay((d, i) => {
  const categoryIndex = categories.indexOf(d.category);
  const itemInCategory = data.filter(
    (x, j) => x.category === d.category && j < i
  ).length;
  return categoryIndex * 200 + itemInCategory * 50;
})
```

### React implementation

```typescript
import { useEffect, useState } from "react";

function StaggeredBars({ data, width, height, palette }: Props) {
  const [animationProgress, setAnimationProgress] = useState(0);

  useEffect(() => {
    // Trigger animation on mount
    const timeout = setTimeout(() => setAnimationProgress(1), 50);
    return () => clearTimeout(timeout);
  }, []);

  return (
    <svg viewBox={`0 0 ${width} ${height}`}>
      {data.map((d, i) => {
        const targetHeight = innerHeight - yScale(d.value);
        const delay = i * 50;
        const currentHeight = animationProgress * targetHeight;

        return (
          <rect
            key={d.id}
            x={xScale(d.category)}
            y={innerHeight - currentHeight}
            width={xScale.bandwidth()}
            height={currentHeight}
            fill={palette.primary}
            style={{
              transition: `all 400ms ${delay}ms cubic-bezier(0.33, 1, 0.68, 1)`,
            }}
          />
        );
      })}
    </svg>
  );
}
```

## Data update animations

### Smooth transitions on data change

```typescript
import { useEffect, useRef } from "react";
import { select } from "d3-selection";
import { transition } from "d3-transition";

function AnimatedBars({ data, width, height, palette }: Props) {
  const svgRef = useRef<SVGSVGElement>(null);

  useEffect(() => {
    if (!svgRef.current) return;

    const svg = select(svgRef.current);

    svg.selectAll("rect")
      .data(data, d => d.id)
      .join(
        enter => enter
          .append("rect")
          .attr("x", d => xScale(d.category))
          .attr("y", innerHeight)
          .attr("width", xScale.bandwidth())
          .attr("height", 0)
          .attr("fill", palette.primary)
          .call(enter => enter
            .transition()
            .duration(400)
            .attr("y", d => yScale(d.value))
            .attr("height", d => innerHeight - yScale(d.value))
          ),
        update => update
          .call(update => update
            .transition()
            .duration(300)
            .attr("y", d => yScale(d.value))
            .attr("height", d => innerHeight - yScale(d.value))
          ),
        exit => exit
          .call(exit => exit
            .transition()
            .duration(200)
            .attr("height", 0)
            .attr("y", innerHeight)
            .remove()
          )
      );
  }, [data, xScale, yScale, innerHeight, palette]);

  return <svg ref={svgRef} width={width} height={height} />;
}
```

### Interpolating between states

```typescript
import { interpolate } from "d3-interpolate";

// Interpolate complex attributes
const pathInterpolator = interpolate(oldPath, newPath);

element
  .transition()
  .duration(500)
  .attrTween("d", () => t => pathInterpolator(t));
```

## Scroll-triggered reveals

### Intersection Observer approach

```typescript
import { useEffect, useRef, useState } from "react";

function useOnScreen(threshold = 0.3) {
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

function ScrollRevealChart({ data, ...props }: ChartProps) {
  const [ref, isVisible] = useOnScreen(0.3);

  return (
    <div ref={ref}>
      <Chart
        data={data}
        {...props}
        animationProgress={isVisible ? 1 : 0}
      />
    </div>
  );
}
```

### Scroll position-based animation

```typescript
import { useEffect, useState } from "react";

function useScrollProgress(elementRef: React.RefObject<HTMLElement>) {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    const element = elementRef.current;
    if (!element) return;

    const handleScroll = () => {
      const rect = element.getBoundingClientRect();
      const windowHeight = window.innerHeight;

      // Progress from 0 (element enters) to 1 (element exits)
      const start = windowHeight;
      const end = -rect.height;
      const current = rect.top;

      const progress = Math.max(0, Math.min(1, (start - current) / (start - end)));
      setProgress(progress);
    };

    window.addEventListener("scroll", handleScroll, { passive: true });
    handleScroll();
    return () => window.removeEventListener("scroll", handleScroll);
  }, [elementRef]);

  return progress;
}
```

## Motion principles for storytelling

### Sequence reveals to guide attention

```typescript
// Reveal elements in narrative order
const sequence = [
  { selector: ".headline", delay: 0 },
  { selector: ".chart-area", delay: 200 },
  { selector: ".annotation-1", delay: 600 },
  { selector: ".annotation-2", delay: 800 },
  { selector: ".source", delay: 1000 },
];

sequence.forEach(({ selector, delay }) => {
  select(selector)
    .style("opacity", 0)
    .transition()
    .delay(delay)
    .duration(400)
    .style("opacity", 1);
});
```

### Highlight changes

When data updates, emphasize what changed:

```typescript
// Flash changed values
svg.selectAll("rect")
  .data(newData, d => d.id)
  .join("rect")
  .filter(d => {
    const old = oldDataMap.get(d.id);
    return old && old.value !== d.value;
  })
  .attr("fill", palette.highlight)  // Temporary highlight
  .transition()
  .delay(500)
  .duration(300)
  .attr("fill", palette.primary);   // Return to normal
```

### Motion should have meaning

| Good Motion | Bad Motion |
|-------------|------------|
| Bars grow to show magnitude | Spinning entrance |
| Lines draw to show progression | Bouncing for no reason |
| Fade in for reveals | Constant animation |
| Highlight changes | Motion that distracts |

## Performance considerations

### Avoid layout thrashing

```typescript
// Bad: Multiple reads and writes
elements.forEach(el => {
  const rect = el.getBoundingClientRect();  // Read
  el.style.transform = `translateY(${rect.top}px)`;  // Write
});

// Good: Batch reads, then writes
const rects = elements.map(el => el.getBoundingClientRect());  // All reads
elements.forEach((el, i) => {
  el.style.transform = `translateY(${rects[i].top}px)`;  // All writes
});
```

### Use CSS transforms

```typescript
// Good: GPU-accelerated
.transition()
.style("transform", `translateY(${y}px)`)

// Less efficient: Triggers layout
.transition()
.attr("y", y)
```

### Reduce animated elements

```typescript
// Too many: Animating every data point
data.forEach((d, i) => {
  animate(d, i * 10);  // 1000 animations
});

// Better: Animate groups or representative elements
const groups = groupBy(data, d => d.category);
groups.forEach((group, i) => {
  animateGroup(group, i * 100);  // 10 animations
});
```

### Disable motion for reduced motion preference

```typescript
// CSS
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

// JavaScript
const prefersReducedMotion = window.matchMedia(
  "(prefers-reduced-motion: reduce)"
).matches;

const duration = prefersReducedMotion ? 0 : 400;
```

## GPU-accelerated animation

For smooth 60fps animations, only animate properties that don't trigger layout recalculation.

### GPU-accelerated vs CPU-bound properties

| GPU-Accelerated (Compositor) | CPU-Bound (Repaint) |
|------------------------------|---------------------|
| `transform` | `x`, `y`, `cx`, `cy` |
| `opacity` | `fill`, `stroke` |
| | `width`, `height`, `r` |

### Transform-based animation

```typescript
// Instead of animating position attributes, use transform
svg.selectAll("circle")
  .data(data)
  .join("circle")
  .attr("cx", 0)
  .attr("cy", 0)
  .attr("r", 5)
  .style("transform", d => `translate(${xScale(d.x)}px, ${yScale(d.y)}px)`)
  .transition()
  .duration(300)
  .style("transform", d => `translate(${newX}px, ${newY}px)`);
```

### will-change hint

```typescript
// Apply before animation starts
selection.style("will-change", "transform, opacity");

selection.transition()
  .style("transform", "translateX(100px)")
  .on("end", function() {
    // Remove after animation completes
    d3.select(this).style("will-change", "auto");
  });
```

## Scrollytelling implementation

For narrative-driven visualizations that reveal as users scroll.

### Full scrollytelling component

```typescript
import { useRef, useEffect, useState, ReactNode } from "react";

interface ScrollyStep {
  content: ReactNode;
  onEnter?: () => void;
}

function Scrollytelling({ steps, stickyChart }: {
  steps: ScrollyStep[];
  stickyChart: ReactNode;
}) {
  const [activeStep, setActiveStep] = useState(0);

  return (
    <div style={{ position: "relative" }}>
      {/* Sticky visualization */}
      <div style={{
        position: "sticky",
        top: "10vh",
        height: "80vh",
        display: "flex",
        alignItems: "center",
        justifyContent: "center"
      }}>
        {stickyChart}
      </div>

      {/* Scrollable steps */}
      <div style={{ position: "relative", marginTop: "-80vh" }}>
        {steps.map((step, i) => (
          <ScrollyStep
            key={i}
            index={i}
            onActivate={() => {
              setActiveStep(i);
              step.onEnter?.();
            }}
          >
            {step.content}
          </ScrollyStep>
        ))}
      </div>
    </div>
  );
}

function ScrollyStep({
  children,
  index,
  onActivate
}: {
  children: ReactNode;
  index: number;
  onActivate: () => void;
}) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) onActivate();
      },
      { threshold: 0.5, rootMargin: "-20% 0px -20% 0px" }
    );

    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, [onActivate]);

  return (
    <div ref={ref} style={{ minHeight: "80vh", padding: "40vh 0" }}>
      {children}
    </div>
  );
}
```

### Step-based chart state

```typescript
function StoryChart({ activeStep, data }) {
  const chartState = {
    0: { showAxes: true, showData: false, highlightMax: false },
    1: { showAxes: true, showData: true, highlightMax: false },
    2: { showAxes: true, showData: true, highlightMax: true },
    3: { showAxes: true, showData: true, highlightMax: true, showTrend: true },
  }[activeStep] ?? {};

  return (
    <svg>
      {chartState.showAxes && <Axes />}
      {chartState.showData && <DataBars data={data} />}
      {chartState.highlightMax && <MaxAnnotation data={data} />}
      {chartState.showTrend && <TrendLine data={data} />}
    </svg>
  );
}
```

## Related references

- [advanced-techniques.md](advanced-techniques.md) - Path morphing, animated gradients
- [infographic-design.md](infographic-design.md) - When motion serves the story
