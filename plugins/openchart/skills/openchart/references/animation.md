# Animation

Entrance animations for charts and tables. Off by default. Set `animation: true` on any chart or table spec to enable entrance animation with sensible defaults. Animation is a top-level spec property alongside `responsive`, `theme`, and `darkMode`.

The implementation follows Vega's enter/update/exit conceptual model. Currently only entrance (`enter`) animations are supported. Update and exit phases are reserved for v2.

All animations are pure CSS (keyframes, clip-path, opacity). The system automatically respects `prefers-reduced-motion`, disabling all animations when the user has requested reduced motion.

## Type Reference

```typescript
type AnimationSpec = boolean | AnimationConfig;

interface AnimationConfig {
  enter?: AnimationPhaseConfig | boolean;
  update?: AnimationPhaseConfig | boolean;  // Reserved for v2
  exit?: AnimationPhaseConfig | boolean;    // Reserved for v2
  annotationDelay?: number;                 // Default: 200ms
}

interface AnimationPhaseConfig {
  duration?: number;                        // Default: 500ms
  ease?: AnimationEase;                     // Default: 'smooth'
  stagger?: AnimationStagger | boolean;     // Default: true
}

type AnimationEase = 'smooth' | 'snappy';

interface AnimationStagger {
  delay?: number;                           // Default: 80ms
  order?: 'index' | 'value' | 'reverse';   // Default: 'index'
}
```

## Easing Presets

Two named presets that map to CSS `linear()` timing functions. No bounce or overshoot - data visualizations should feel precise, not playful.

| Preset | Feel | When to use |
| --- | --- | --- |
| `smooth` | Decelerating ease-out, clean deceleration | Default. All chart types, editorial, financial. |
| `snappy` | Fast attack, gentle settle | Small elements (points, ticks). Dense scatter plots. |

Easing uses CSS `linear()` which requires Chrome 113+, Firefox 112+, Safari 17.2+. Older browsers get a graceful fallback to CSS `ease`.

## Per-Mark-Type Defaults

Each mark type gets an appropriate entrance animation when `animation: true` is set. These are not configurable per-mark in the spec (only duration, ease, and stagger can be overridden).

| Mark type | Animation style | Detail |
| --- | --- | --- |
| `bar`/`rect` (vertical) | Clip-path reveal bottom-to-top + fade | Bars grow upward from baseline with smooth easing |
| `bar`/`rect` (horizontal) | Clip-path reveal left-to-right + fade | Bars grow rightward from axis with smooth easing |
| Stacked bars/columns | Sequential segment reveal | Segments chain left-to-right (or bottom-to-top) so the full bar animates as one fluid unit |
| `line` | Clip-path reveal left-to-right | Line draws progressively across the chart |
| `area` | Clip-path reveal left-to-right + fade | Same as line, fill fades in simultaneously |
| `arc` | Fade in | Clean opacity fade per slice |
| `point` | Fade in | Points pop in individually. On line/area charts, delayed to follow the line draw. |
| `text` | Fade + subtle slide up | Small upward motion with opacity |
| `rule`/`tick` | Fade in | Simple opacity fade |
| Annotations | Delayed fade | Fade in after data marks complete, controlled by `annotationDelay` |
| Data labels | Delayed fade | Fade in at 70% of animation duration |

## Stagger Behavior

Stagger adds a sequential delay between elements so they animate in one after another instead of all at once. Enabled by default with 80ms delay between elements.

**Auto-clamping:** To prevent slow animations on charts with many elements, the effective delay is clamped: `effectiveDelay = min(delay, 2000 / count)`. A chart with 100 bars and 80ms stagger would take 8 seconds without clamping. The clamp caps total stagger time at ~2 seconds.

**Stagger order:**

| Order | Behavior |
| --- | --- |
| `index` | DOM order. Bars animate top-to-bottom (horizontal) or left-to-right (vertical columns). Default. |
| `value` | By primary encoding value, ascending. Smallest values animate first. |
| `reverse` | Reverse DOM order. |

## Usage Examples

### Minimal

```json
{
  "mark": "bar",
  "data": [...],
  "encoding": { ... },
  "animation": true
}
```

Enables entrance animation with all defaults: 500ms duration, 80ms stagger, smooth easing and mark-appropriate animation style.

### Custom Enter Config

```json
{
  "animation": {
    "enter": {
      "duration": 800,
      "ease": "smooth",
      "stagger": { "delay": 50, "order": "value" }
    }
  }
}
```

### Disable on Mobile

Animation replays on each mount. Disable at compact breakpoint for mobile:

```json
{
  "animation": true,
  "overrides": {
    "compact": { "animation": false }
  }
}
```

### Disable Stagger

All elements animate simultaneously:

```json
{
  "animation": {
    "enter": { "stagger": false }
  }
}
```

### Custom Annotation Delay

Control how long after marks finish before annotations fade in:

```json
{
  "animation": {
    "enter": true,
    "annotationDelay": 500
  }
}
```

## Table Animations

Tables support the same `animation` property as charts. When enabled, tables animate on first render only (sort, search, pagination, and resize do not re-trigger animation).

**What animates:**

| Element | Animation | Sequencing |
| --- | --- | --- |
| Chrome (title/subtitle) | Fade + slide up | Immediate |
| Table header | Fade in | Immediate |
| Rows | Slide up (no opacity) | Staggered by row index |
| Text cells | Fade in | Synced with row |
| Heatmap/category backgrounds | Fade in | Delayed after row appears |
| Inline bar fills | Clip-path grow left-to-right | Delayed after row appears |
| Sparkline lines | Clip-path draw left-to-right | Delayed after row appears |
| Sparkline dots/labels | Fade in | After line finishes drawing |
| Search/pagination | Fade in | Immediate |

**Design notes:**
- Row animation is transform-only (slide up, no opacity) so it doesn't compound with cell-level opacity fades.
- Bar fill animation uses clip-path only (no opacity) to preserve the resting `opacity: 0.15` that gives bars their subtle look.
- Sparkline clip-path targets the inner SVG element to avoid clipping the absolutely-positioned dots and labels.
- Stagger is auto-clamped by row count, same as charts (max ~2s total stagger time).

```json
{
  "type": "table",
  "data": [...],
  "columns": [...],
  "animation": true
}
```

Custom config works the same as charts:

```json
{
  "type": "table",
  "animation": { "enter": { "duration": 800, "ease": "snappy" } }
}
```

## Integration Gotchas

### `chart.update()` cancels in-progress animations

Calling `chart.update(newSpec)` triggers an internal `render()` which runs `cancelAnimations()` before re-rendering. This removes the `oc-animate` class and kills any running CSS entrance animation. If your React wrapper changes the spec after mount (even to remove the `animation` field), the animation dies mid-flight.

**Critical rule:** Do not change the spec object during the animation window (~2.5s after mount). If you need to strip `animation` later (e.g., to prevent replay on theme toggle), use a `useRef` flag, not `useState`. State changes trigger re-renders which trigger `chart.update()`.

### Scroll-into-view (lazy loading)

The most common pattern for blog/article animations: charts animate when scrolled into the viewport. Use `IntersectionObserver` (or a `useInView` hook) to control when the chart component mounts. The animation fires on first `createChart()` call automatically.

**Key settings:**
- `rootMargin: "50px"` - mounts the chart just before viewport entry, giving it a frame to render before the user sees it. Too large (e.g., 400px) and the animation plays off-screen.
- `threshold: 0` - triggers immediately on intersection.
- `triggerOnce: true` - chart stays mounted after first intersection.

```tsx
function AnimatedChart({ spec }) {
  const { ref, isVisible } = useInView({ rootMargin: "50px", threshold: 0 });
  const hasAnimatedRef = useRef(false);

  // Include animation only on the first mount. The ref prevents replay when
  // the component re-renders (e.g., theme toggle destroys/recreates the chart).
  const chartSpec = useMemo(() => {
    if (hasAnimatedRef.current) return spec;
    hasAnimatedRef.current = true;
    return { ...spec, animation: true };
  }, [spec]);

  if (!isVisible) return <div ref={ref} style={{ minHeight: 200 }} />;
  return <Chart spec={chartSpec} />;
}
```

### Theme toggle replays entrance animation

In class-based dark mode apps (Astro, Next.js), the typical wrapper pattern maps the DOM theme class to `darkMode: "force"` or `"off"`. Since `darkMode` is in the React `Chart` component's mount effect dependency array, toggling the theme destroys and recreates the chart instance. A fresh `createChart()` resets `isFirstRender = true`, so if `animation` is still in the spec, the entrance replays.

**Fix:** Use a `useRef` to track whether animation has already played. Include `darkMode` (or equivalent) in the `useMemo` deps so it recomputes on theme toggle, but the ref ensures animation is excluded on the second computation:

```tsx
const hasAnimatedRef = useRef(false);

const chartSpec = useMemo(() => {
  if (hasAnimatedRef.current) return spec;
  hasAnimatedRef.current = true;
  return { ...spec, animation: true };
  // darkMode triggers recomputation on theme toggle, but the ref
  // ensures animation is only included on the first evaluation.
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, [spec, darkMode]);
```

**Why `useRef` and not `useState`:** Setting state triggers a re-render, which changes the spec, which calls `chart.update()`, which calls `cancelAnimations()`. The animation gets killed on the very next frame. A ref flip is invisible to React's render cycle.

## Anti-Patterns

| Mistake | Fix |
| --- | --- |
| Long stagger delay on charts with many elements | Let auto-clamping handle it, or set an explicit lower delay. |
| Animating charts that update frequently | Entrance animation replays on each mount. Disable for live-updating dashboards. |
| Setting duration over 1500ms | Animations should feel snappy. Keep total time (including stagger) under 2.5s. |
| Low stagger delay on scatter plots | Points need enough delay between them to appear individually. Use 40-80ms. |
| Using `useState` to track animation completion | State changes trigger spec updates which cancel animations. Use `useRef` instead. |
| Large `rootMargin` with lazy-loaded animated charts | Animation plays off-screen. Use 50px or less for scroll-triggered animations. |

## Browser Support

- **Easing via CSS `linear()`:** Chrome 113+, Firefox 112+, Safari 17.2+
- **Older browsers:** Graceful fallback to CSS `ease` (smooth deceleration)
- **`prefers-reduced-motion`:** All animations disabled automatically. No additional configuration needed.
- **Frameworks:** Animation works identically across React, Vue, Svelte, and vanilla JS. No framework-specific setup needed.
