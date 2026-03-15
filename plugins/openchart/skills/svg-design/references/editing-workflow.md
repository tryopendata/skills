# SVG Editing Workflow

## Modifying Existing SVGs

### Changing colors

**Single color icon:** Change the `fill` or `stroke` on the root `<svg>` element.

```xml
<!-- Before: hardcoded color -->
<svg viewBox="0 0 24 24">
  <path fill="#000000" d="..." />
</svg>

<!-- After: themeable -->
<svg viewBox="0 0 24 24" fill="currentColor">
  <path d="..." />
</svg>
```

**Multi-color icon:** Change `fill`/`stroke` on individual elements:

```xml
<svg viewBox="0 0 24 24">
  <circle cx="12" cy="12" r="10" fill="#3B82F6" />  <!-- Change this hex -->
  <path d="..." fill="white" />                       <!-- And this one -->
</svg>
```

**Gradient swap:** Replace the gradient stops in `<defs>`:

```xml
<defs>
  <linearGradient id="g1">
    <stop offset="0%" stop-color="#NEW_COLOR_1" />
    <stop offset="100%" stop-color="#NEW_COLOR_2" />
  </linearGradient>
</defs>
```

### Scaling an SVG

**To change the default rendered size**, adjust `width`/`height` attributes:

```xml
<!-- Render at 32px by default instead of 24px -->
<svg width="32" height="32" viewBox="0 0 24 24">
```

**To change the coordinate system** (e.g., converting a 24x24 icon to work on a 16x16 grid), you need to either:
- Scale all coordinates by the ratio (16/24 = 0.667)
- Wrap content in a group with `transform="scale(0.667)"`
- Or just change the viewBox and let the browser scale it (easiest)

```xml
<!-- Easiest: just change the viewBox -->
<!-- The paths stay the same, browser handles the math -->
<svg viewBox="0 0 24 24" width="16" height="16">
```

### Combining multiple SVGs

Merge SVGs by placing their content into a single `<svg>` element. Adjust positions using `transform="translate(x, y)"` or by wrapping in `<g>` groups.

```xml
<!-- Icon + text logo composition -->
<svg viewBox="0 0 200 40" xmlns="http://www.w3.org/2000/svg">
  <!-- Icon (scaled down from 24x24 to 32x32 area, positioned at left) -->
  <g transform="translate(4, 4) scale(1.33)">
    <!-- paste icon paths here -->
  </g>
  <!-- Wordmark text -->
  <text x="44" y="28" font-family="Inter" font-size="20" font-weight="700" fill="currentColor">
    BrandName
  </text>
</svg>
```

### Flipping/mirroring

```xml
<!-- Horizontal flip (mirror) -->
<g transform="scale(-1, 1) translate(-24, 0)">
  <!-- original paths -->
</g>

<!-- Vertical flip -->
<g transform="scale(1, -1) translate(0, -24)">
  <!-- original paths -->
</g>
```

The `translate` compensates for the flip moving the content off-canvas. The translate values should match the viewBox dimensions.

### Rotating

```xml
<!-- Rotate 90 degrees clockwise around center -->
<g transform="rotate(90 12 12)">
  <!-- paths -->
</g>
```

## Conceptual Boolean Operations

Design tools have union, subtract, intersect, and exclude. In raw SVG, you achieve these with compound paths and fill rules.

### Union (combine two shapes into one)

**Approach:** Merge both shapes' path data into a single `<path>`. Overlapping areas remain filled.

```xml
<!-- Two overlapping circles as separate elements -->
<circle cx="10" cy="12" r="8" />
<circle cx="14" cy="12" r="8" />

<!-- Unioned: single path containing both -->
<path d="
  M 10 4 A 8 8 0 1 1 10 20 A 8 8 0 1 1 10 4 Z
  M 14 4 A 8 8 0 1 1 14 20 A 8 8 0 1 1 14 4 Z
" />
```

With the default `fill-rule="nonzero"`, overlapping same-direction subpaths just fill. This gives you a union.

For a true outline-only union (merged contour with no interior lines), you'd need to calculate the actual merged path. This is computationally complex. In practice, for hand-written SVGs, you either:
- Accept the overlapping paths (they render the same when filled)
- Manually trace the combined outline

### Subtract (cut one shape out of another)

**Approach:** Use `fill-rule="evenodd"` with overlapping subpaths. The intersection becomes transparent.

```xml
<!-- Circle with rectangular cutout -->
<path fill-rule="evenodd" d="
  M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z
  M 8 8 h 8 v 8 h -8 Z
" />
```

Alternatively, use `<mask>` for non-path shapes:

```xml
<defs>
  <mask id="subtract">
    <rect width="24" height="24" fill="white" />
    <rect x="8" y="8" width="8" height="8" fill="black" />
  </mask>
</defs>
<circle cx="12" cy="12" r="10" mask="url(#subtract)" />
```

### Intersect (keep only the overlap)

**Approach:** Use `<clipPath>` with one shape clipping the other.

```xml
<defs>
  <clipPath id="clip-circle">
    <circle cx="14" cy="12" r="8" />
  </clipPath>
</defs>
<!-- Only the part of this circle that overlaps with the clip circle is visible -->
<circle cx="10" cy="12" r="8" clip-path="url(#clip-circle)" />
```

### Exclude (XOR: only non-overlapping areas)

**Approach:** Use `fill-rule="evenodd"` with both shapes in a single path. Where they overlap, the fill cancels out.

```xml
<!-- Two overlapping circles with XOR behavior -->
<path fill-rule="evenodd" d="
  M 10 4 A 8 8 0 1 1 10 20 A 8 8 0 1 1 10 4 Z
  M 14 4 A 8 8 0 1 1 14 20 A 8 8 0 1 1 14 4 Z
" />
```

The key difference from union: with `evenodd`, the overlapping region is transparent. With `nonzero` (default), it's filled.

## Converting Between Formats

### Stroked icon to filled icon

Convert a stroked path to a filled shape by "outlining" the stroke. This means creating two paths that trace the outer and inner edges of the stroke, then closing them.

For a simple straight line:

```xml
<!-- Stroked -->
<line x1="4" y1="12" x2="20" y2="12" stroke="currentColor" stroke-width="2" />

<!-- Equivalent filled (2px wide horizontal bar) -->
<rect x="4" y="11" width="16" height="2" fill="currentColor" rx="1" />
```

For curves, this is much harder by hand. Design tools handle this via "Outline Stroke" or "Expand" commands.

### Filled icon to stroked icon

Go the other direction by finding the centerline of filled shapes. This is generally not feasible by hand for complex shapes. Works for simple geometric shapes:

```xml
<!-- Filled square -->
<rect x="4" y="4" width="16" height="16" fill="currentColor" />

<!-- Stroked equivalent -->
<rect x="5" y="5" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none" />
```

Note the inset: the stroked version is smaller because the stroke extends outward from the path.

## Multi-Variant Preview Page

When creating multiple logo/icon options, build an HTML preview page alongside the SVGs. This is the single most impactful thing for fast iteration.

```html
<!DOCTYPE html>
<html>
<head>
<style>
  .card { background: white; border-radius: 12px; padding: 20px; border: 1px solid #e5e7eb; }
  .sizes { display: flex; align-items: flex-end; gap: 16px; }
  .dark-row { background: #0d1117; padding: 16px; border-radius: 8px; margin-top: 12px; }
  .nav-preview { display: flex; align-items: center; gap: 8px; margin-top: 12px; padding: 8px 12px; }
  .nav-dark { background: #0d1117; }
</style>
</head>
<body>
  <div class="card">
    <h3>Variant Name</h3>
    <!-- Show at multiple sizes -->
    <div class="sizes">
      <div><img src="logo.svg" width="16" height="16">16</div>
      <div><img src="logo.svg" width="32" height="32">32</div>
      <div><img src="logo.svg" width="64" height="64">64</div>
    </div>
    <!-- Dark background row (use dark variant for colored logos) -->
    <div class="dark-row">
      <img src="logo-dark.svg" width="64" height="64">
    </div>
    <!-- Nav bar mockup -->
    <div class="nav-preview">
      <img src="logo.svg" width="28" height="28">
      <span><span style="color:#2563EB">Open</span>Data</span>
    </div>
  </div>
</body>
</html>
```

Key things to include in every preview:
- **Three sizes** (16, 32, 64) to catch detail that doesn't scale
- **Dark background row** using the actual site dark bg color
- **Nav bar mockup** showing the logo next to the wordmark at realistic size
- For colored logos, use the `-dark.svg` variant in the dark rows (not CSS filters)
- For monochrome logos, use `style="filter:brightness(0) invert(1)"` in dark rows

## Workflow Tips

1. **Start with shape primitives**, convert to paths only when needed for optimization or compound operations
2. **Use relative coordinates** (`m`, `l`, `c`) when hand-writing paths. They're easier to reason about incrementally
3. **Test at multiple sizes**: render your SVG at 16px, 24px, 48px, and 200px to verify it scales cleanly
4. **Keep a monochrome version**: if your icon/logo uses color, make sure it also works with a single `currentColor`
5. **Validate the output**: open in a browser, not just a code editor. Check for rendering artifacts
6. **Round coordinates to the grid**: snap to integers on 24x24 canvas for pixel-perfect rendering at 1x
