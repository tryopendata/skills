# Advanced SVG Techniques

## Gradients

### Linear Gradient

Transitions color along a straight line. Defined in `<defs>`, referenced via `url(#id)`.

```xml
<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <linearGradient id="grad1" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#3B82F6" />
      <stop offset="100%" stop-color="#8B5CF6" />
    </linearGradient>
  </defs>
  <circle cx="12" cy="12" r="10" fill="url(#grad1)" />
</svg>
```

| Attribute | Purpose |
|-----------|---------|
| `x1`, `y1` | Start point of gradient line (percentages or coordinates) |
| `x2`, `y2` | End point of gradient line |
| `gradientUnits` | `objectBoundingBox` (default, relative to shape) or `userSpaceOnUse` (absolute to viewBox) |
| `gradientTransform` | Transform the gradient independently of the shape |

**Direction shortcuts:**

| Direction | x1 y1 x2 y2 |
|-----------|-------------|
| Left to right | 0% 0% 100% 0% |
| Top to bottom | 0% 0% 0% 100% |
| Diagonal (TL to BR) | 0% 0% 100% 100% |
| Diagonal (BL to TR) | 0% 100% 100% 0% |

### Radial Gradient

Transitions color from a center point outward.

```xml
<defs>
  <radialGradient id="grad2" cx="50%" cy="50%" r="50%">
    <stop offset="0%" stop-color="#FBBF24" />
    <stop offset="100%" stop-color="#F59E0B" />
  </radialGradient>
</defs>
<circle cx="12" cy="12" r="10" fill="url(#grad2)" />
```

| Attribute | Purpose |
|-----------|---------|
| `cx`, `cy` | Center of the gradient |
| `r` | Radius of the gradient |
| `fx`, `fy` | Focal point (where the gradient starts, can differ from center for off-center glow) |

### Multi-stop gradients

```xml
<linearGradient id="rainbow">
  <stop offset="0%" stop-color="#EF4444" />
  <stop offset="25%" stop-color="#F59E0B" />
  <stop offset="50%" stop-color="#10B981" />
  <stop offset="75%" stop-color="#3B82F6" />
  <stop offset="100%" stop-color="#8B5CF6" />
</linearGradient>
```

### Gradient tips for logos

- Use gradients sparingly. A logo should work in flat/monochrome first.
- Prefer 2-3 stops max. Complex gradients don't scale down well.
- Always provide a flat-color fallback version of gradient logos.
- Use `gradientUnits="userSpaceOnUse"` when applying the same gradient across multiple shapes to get a unified gradient rather than per-shape gradients.

## Masks

Masks use luminance (brightness) to control visibility. White = fully visible, black = fully hidden, gray = partially transparent.

```xml
<defs>
  <mask id="fade-mask">
    <rect width="24" height="24" fill="white" />
    <!-- Black areas will be invisible -->
    <circle cx="12" cy="12" r="5" fill="black" />
  </mask>
</defs>
<!-- Square with circular hole -->
<rect width="24" height="24" fill="#3B82F6" mask="url(#fade-mask)" />
```

### Gradient mask (fade effect)

```xml
<defs>
  <linearGradient id="fade-grad">
    <stop offset="0%" stop-color="white" />
    <stop offset="100%" stop-color="black" />
  </linearGradient>
  <mask id="fade-out">
    <rect width="24" height="24" fill="url(#fade-grad)" />
  </mask>
</defs>
<rect width="24" height="24" fill="#3B82F6" mask="url(#fade-out)" />
```

## Clip Paths

Binary clipping: inside the clip shape = visible, outside = invisible. No partial transparency.

```xml
<defs>
  <clipPath id="circle-clip">
    <circle cx="12" cy="12" r="10" />
  </clipPath>
</defs>
<!-- Square clipped to circle shape -->
<rect width="24" height="24" fill="#3B82F6" clip-path="url(#circle-clip)" />
```

### Clip-path vs mask

| Feature | clip-path | mask |
|---------|-----------|------|
| Transparency | Binary (in or out) | Gradient (luminance-based) |
| Performance | Faster | Slower |
| Use case | Hard edges, shape cutouts | Fade effects, soft edges |
| Defined with | Shapes/paths | Any elements (including gradients) |

## Filters

Filters apply raster effects. Use sparingly in logos/icons since they don't scale as cleanly as vector shapes.

### Drop shadow

```xml
<defs>
  <filter id="shadow" x="-20%" y="-20%" width="140%" height="140%">
    <feDropShadow dx="1" dy="1" stdDeviation="1" flood-color="#00000040" />
  </filter>
</defs>
<circle cx="12" cy="12" r="8" fill="#3B82F6" filter="url(#shadow)" />
```

### Blur

```xml
<defs>
  <filter id="blur">
    <feGaussianBlur stdDeviation="2" />
  </filter>
</defs>
<circle cx="12" cy="12" r="8" fill="#3B82F6" filter="url(#blur)" />
```

### When to use filters in logos

Almost never. Filters are raster operations that:
- Don't scale cleanly at different sizes
- Increase rendering cost
- Add complexity to the SVG

If you need a shadow or glow on a logo, consider:
- Using a slightly offset duplicate shape with reduced opacity
- Applying the shadow via CSS `filter: drop-shadow()` on the container instead
- Creating the shadow effect with actual vector shapes

## Transforms

### SVG transform attribute

Applied directly to elements. Values are in the SVG coordinate system.

```xml
<!-- Rotate 45 degrees around center of 24x24 canvas -->
<rect x="8" y="8" width="8" height="8" transform="rotate(45 12 12)" />

<!-- Scale to half size -->
<g transform="scale(0.5)">...</g>

<!-- Translate (move) -->
<circle cx="0" cy="0" r="5" transform="translate(12 12)" />

<!-- Multiple transforms (applied right-to-left) -->
<rect transform="translate(12 12) rotate(45) translate(-4 -4)" width="8" height="8" />
```

**Transform functions:**

| Function | Syntax | Notes |
|----------|--------|-------|
| `translate` | `translate(tx, ty)` | Move element. ty defaults to 0 |
| `rotate` | `rotate(angle, cx, cy)` | Rotate. cx,cy default to 0,0 |
| `scale` | `scale(sx, sy)` | Scale. sy defaults to sx |
| `skewX` | `skewX(angle)` | Horizontal skew |
| `skewY` | `skewY(angle)` | Vertical skew |
| `matrix` | `matrix(a,b,c,d,e,f)` | Full 2D transform matrix |

### SVG transform vs CSS transform

| Aspect | SVG `transform` attribute | CSS `transform` property |
|--------|--------------------------|-------------------------|
| Default origin | `0 0` (top-left of SVG canvas) | `50% 50%` (center of element) |
| Origin control | Via `rotate(angle, cx, cy)` parameter | Via `transform-origin` property |
| Priority | Lower (CSS overrides) | Higher |
| Animation | Harder (SMIL or JS) | Easier (CSS transitions/keyframes) |
| Units | SVG user units only | Supports px, %, em, etc. |

**When baking transforms into path data is better:** For distributed icons, flatten transforms into the path coordinates. This avoids rendering differences and makes the SVG simpler. Design tools do this on export. For dynamic/animated SVGs, keep transforms as attributes or CSS.

## Animation

For comprehensive animation guidance (CSS keyframes, GPU acceleration, staggering, easing curves, SVG-specific techniques like viewBox clipping and box occlusion), see [animation.md](animation.md).

## Accessibility

### Decorative SVGs (no semantic meaning)

```xml
<svg aria-hidden="true" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <!-- decorative icon -->
</svg>
```

### Informative SVGs (convey meaning)

```xml
<svg role="img" aria-labelledby="icon-title" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <title id="icon-title">Settings</title>
  <!-- icon paths -->
</svg>
```

### SVGs with description

```xml
<svg role="img" aria-labelledby="logo-title logo-desc" viewBox="0 0 200 50" xmlns="http://www.w3.org/2000/svg">
  <title id="logo-title">Acme Corp</title>
  <desc id="logo-desc">Company logo featuring a stylized mountain peak</desc>
  <!-- logo paths -->
</svg>
```

### Pattern summary

| Context | Pattern |
|---------|---------|
| Decorative icon next to text label | `aria-hidden="true"` on svg |
| Standalone icon (no text label) | `role="img"` + `<title>` + `aria-labelledby` |
| Complex illustration | `role="img"` + `<title>` + `<desc>` + `aria-labelledby` |
| Interactive icon (button) | Wrap in `<button>` with `aria-label`, SVG gets `aria-hidden="true"` |

### Key rules

- `<title>` must be the **first child** of its parent element
- Use `aria-labelledby` (not `aria-label`) when `<title>`/`<desc>` exist
- `role="img"` tells screen readers to treat the SVG as a single image
- Focusable SVGs need `focusable="false"` on the `<svg>` element if they shouldn't receive focus
