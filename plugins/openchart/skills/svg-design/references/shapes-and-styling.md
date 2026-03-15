# Shape Primitives and Styling

## Shape Elements

### rect

```xml
<rect x="2" y="2" width="20" height="20" rx="3" ry="3" />
```

| Attribute | Purpose |
|-----------|---------|
| `x`, `y` | Top-left corner position |
| `width`, `height` | Dimensions |
| `rx`, `ry` | Corner radius (if only `rx` is set, `ry` matches it) |

**Use for:** Rectangles, squares, rounded rectangles, cards, containers.

### circle

```xml
<circle cx="12" cy="12" r="10" />
```

| Attribute | Purpose |
|-----------|---------|
| `cx`, `cy` | Center point |
| `r` | Radius |

**Use for:** Dots, bullets, circular icons, rings (with stroke, no fill).

### ellipse

```xml
<ellipse cx="12" cy="12" rx="10" ry="6" />
```

| Attribute | Purpose |
|-----------|---------|
| `cx`, `cy` | Center point |
| `rx` | Horizontal radius |
| `ry` | Vertical radius |

**Use for:** Oval shapes, shadows, perspective circles.

### line

```xml
<line x1="2" y1="12" x2="22" y2="12" />
```

| Attribute | Purpose |
|-----------|---------|
| `x1`, `y1` | Start point |
| `x2`, `y2` | End point |

**Use for:** Dividers, crosshairs, simple strokes. Requires `stroke` to be visible.

### polyline

```xml
<polyline points="4,4 12,20 20,4" />
```

Open shape made of connected line segments. Does not auto-close.

**Use for:** Checkmarks, zigzags, open line charts.

### polygon

```xml
<polygon points="12,2 22,22 2,22" />
```

Same as polyline but automatically closes the shape.

**Use for:** Triangles, stars, hexagons, any regular polygon.

## When to Convert Shapes to Paths

**Convert when:**
- Combining multiple shapes into a single compound path (boolean operations)
- You need the shape as part of a larger path composition
- Distributing to environments that only support `<path>` (some icon systems)
- Optimizing for minimal DOM elements

**Keep as shapes when:**
- Code readability matters (a `<circle>` is self-documenting)
- You need to animate specific properties (animating `r` on a circle is cleaner than animating path data)
- The SVG will be programmatically generated or modified

## Shape-to-Path Conversions

```xml
<!-- Circle to path -->
<!-- <circle cx="12" cy="12" r="10" /> becomes: -->
<path d="M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z" />

<!-- Rect to path -->
<!-- <rect x="2" y="2" width="20" height="20" /> becomes: -->
<path d="M 2 2 h 20 v 20 h -20 Z" />

<!-- Rounded rect to path -->
<!-- <rect x="2" y="2" width="20" height="20" rx="3" /> becomes: -->
<path d="M 5 2 h 14 a 3 3 0 0 1 3 3 v 14 a 3 3 0 0 1 -3 3 h -14 a 3 3 0 0 1 -3 -3 v -14 a 3 3 0 0 1 3 -3 Z" />

<!-- Line to path -->
<!-- <line x1="2" y1="12" x2="22" y2="12" /> becomes: -->
<path d="M 2 12 L 22 12" />
```

## Styling Reference

### fill

The interior color of a shape.

```xml
<rect fill="#3B82F6" />           <!-- Hex color -->
<rect fill="rgb(59, 130, 246)" /> <!-- RGB -->
<rect fill="currentColor" />      <!-- Inherits text color -->
<rect fill="none" />              <!-- Transparent (for stroke-only shapes) -->
<rect fill="url(#myGradient)" />  <!-- Reference a gradient -->
```

### stroke

The outline color. Line elements (`<line>`, `<polyline>`) need this to be visible.

```xml
<circle stroke="currentColor" stroke-width="2" />
```

### stroke-width

Width of the stroke. Scales with the viewBox coordinate system, not pixels.

**Important:** On a 24x24 viewBox, `stroke-width="2"` means 2 units out of 24, which is ~8.3% of the canvas width. If you change the viewBox to 48x48, a `stroke-width="2"` will appear half as thick. Always set stroke-width relative to your viewBox dimensions.

| viewBox | Typical stroke-width | Visual result |
|---------|---------------------|---------------|
| 16x16 | 1.5 | ~9.4% of canvas |
| 24x24 | 2 | ~8.3% of canvas |
| 32x32 | 2-2.5 | ~6.3-7.8% of canvas |
| 48x48 | 3 | ~6.3% of canvas |
| 256x256 | 16 | ~6.3% of canvas |

### stroke-linecap

How line endpoints are drawn.

| Value | Effect | Use when |
|-------|--------|----------|
| `butt` | Flat end, flush with endpoint | Technical diagrams, precise alignment |
| `round` | Semicircle at endpoint | Icons, soft aesthetic (most icon sets use this) |
| `square` | Flat end, extends past endpoint by half stroke-width | Rare; when you need butt alignment but round overshoot |

### stroke-linejoin

How corners where two lines meet are drawn.

| Value | Effect | Use when |
|-------|--------|----------|
| `miter` | Sharp pointed corner | Technical, geometric designs |
| `round` | Rounded corner | Icons, soft aesthetic |
| `bevel` | Flat corner (cuts off the point) | Preventing sharp spikes on tight angles |

`miter` has a gotcha: on very acute angles, the miter can extend very far. Use `stroke-miterlimit` to cap it, or just use `round`.

### fill-rule

Determines what counts as "inside" a path for filling purposes. Only matters for self-intersecting or compound paths.

**`nonzero` (default):** Counts path direction. A point is inside if the total winding number is non-zero. Same-direction subpaths add up; opposite-direction subpaths cancel out.

**`evenodd`:** Counts crossings regardless of direction. A point is inside if a ray from it crosses an odd number of path edges.

```xml
<!-- Donut using evenodd (direction doesn't matter) -->
<path fill-rule="evenodd" d="
  M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z
  M 12 7 A 5 5 0 1 1 12 17 A 5 5 0 1 1 12 7 Z
" fill="black" />

<!-- Donut using nonzero (inner circle must wind opposite direction) -->
<path d="
  M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z
  M 12 7 A 5 5 0 1 0 12 17 A 5 5 0 1 0 12 7 Z
" fill="black" />
```

**Rule of thumb:** Use `evenodd` when you have compound shapes with holes. It's simpler because you don't have to worry about winding direction.

### opacity and fill-opacity / stroke-opacity

```xml
<rect opacity="0.5" />              <!-- Everything at 50% -->
<rect fill-opacity="0.5" />         <!-- Fill at 50%, stroke at 100% -->
<rect stroke-opacity="0.5" />       <!-- Stroke at 50%, fill at 100% -->
```

### currentColor

A CSS keyword that resolves to the computed `color` property of the element. This is how icons inherit their color from surrounding text:

```html
<button style="color: #3B82F6;">
  <svg fill="currentColor" viewBox="0 0 24 24">
    <!-- This icon will be #3B82F6 -->
  </svg>
  Save
</button>
```

Set `fill="currentColor"` for filled icons, `stroke="currentColor"` for outlined icons, on the root `<svg>` element.
