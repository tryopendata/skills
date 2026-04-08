# SVG Editing Workflow

## Flipping/Mirroring

The `translate` compensates for the flip moving content off-canvas. Values should match the viewBox dimensions.

```xml
<!-- Horizontal flip -->
<g transform="scale(-1, 1) translate(-24, 0)">...</g>

<!-- Vertical flip -->
<g transform="scale(1, -1) translate(0, -24)">...</g>
```

## Combining Multiple SVGs

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

## Boolean Operations as Compound Paths

Design tools have union, subtract, intersect, and exclude. In raw SVG, achieve these with compound paths and fill rules.

### Union (combine two shapes)

Merge both shapes' path data into a single `<path>`. With the default `fill-rule="nonzero"`, overlapping same-direction subpaths just fill.

For a true outline-only union (merged contour), you'd need to calculate the actual merged path. In practice, either accept overlapping paths (they render the same when filled) or manually trace the combined outline.

### Subtract (cut one shape out of another)

Use `fill-rule="evenodd"` with overlapping subpaths. The intersection becomes transparent:

```xml
<!-- Circle with rectangular cutout -->
<path fill-rule="evenodd" d="
  M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z
  M 8 8 h 8 v 8 h -8 Z
" />
```

Alternatively, use `<mask>` for non-path shapes.

### Intersect (keep only the overlap)

Use `<clipPath>` with one shape clipping the other:

```xml
<defs>
  <clipPath id="clip-circle">
    <circle cx="14" cy="12" r="8" />
  </clipPath>
</defs>
<circle cx="10" cy="12" r="8" clip-path="url(#clip-circle)" />
```

### Exclude (XOR: only non-overlapping areas)

Use `fill-rule="evenodd"` with both shapes in a single path. Where they overlap, the fill cancels out. The key difference from union: with `evenodd`, the overlapping region is transparent. With `nonzero` (default), it's filled.

## Multi-Variant Preview Page

When creating multiple logo/icon options, use the data-driven preview system. This separates the static HTML scaffold from the variant data, so iterating on designs only requires updating a small data file.

### Setup

1. **Copy `assets/preview.html`** to the project directory using `cp` with the absolute path from where this skill was loaded (e.g., `cp /path/to/this/skill/assets/preview.html ./preview.html`). Do not read it into context or modify it.
2. **Write `variants.js`** in the same directory with the variant data (format below).
3. **Auto-open** with `open preview.html` (macOS) or `xdg-open preview.html` (Linux).

### `variants.js` Format

```js
window.VARIANTS = {
  projectName: "Acme",
  brandName: "Acme",
  concepts: [
    {
      name: "Geometric",
      variants: [
        {
          id: "01",
          name: "Prism",
          description: "Light refracting through a triangular prism. Suggests transformation.",
          light: "logo-prism.svg",
          dark: "logo-prism-dark.svg"
        },
        {
          id: "02",
          name: "Hexagon Stack",
          description: "Layered hexagons suggesting modularity and structure.",
          light: "logo-hex.svg"
        }
      ]
    }
  ]
};
```

| Field | Required | Notes |
|-------|----------|-------|
| `projectName` | Yes | Used in page title and heading |
| `brandName` | Yes | Shown in nav bar mockups |
| `concepts[].name` | Yes | Section heading (group by concept, not layout) |
| `concepts[].variants[].id` | Yes | Short identifier, e.g. "01", "02" |
| `concepts[].variants[].name` | Yes | Variant name shown on card |
| `concepts[].variants[].description` | Yes | Metaphor explanation |
| `concepts[].variants[].light` | Yes | Path to light-background SVG |
| `concepts[].variants[].dark` | No | Path to dark-background SVG. Omit for monochrome (auto-applies `filter:brightness(0) invert(1)`) |

### What the preview renders per card

The scaffold in `assets/preview.html` reads `variants.js` and generates:
- **Size ramp** (16, 32, 64px) on white background
- **Dark background row** (16, 32, 64px) using the dark SVG or monochrome filter
- **Favicon mockup** (16px in a browser tab strip)
- **Nav bar mockup** (logo + brand name, light and dark)
- **Click-to-compare** bar (pin up to 4 cards for side-by-side evaluation)
- **Live reload** (3s poll cache-busts SVG images and reloads `variants.js`, no manual refresh needed)

### Iteration workflow

| Change type | What to update | Then |
|-------------|---------------|------|
| Edit an SVG (colors, shapes, paths) | Just the `.svg` file | Nothing. Live reload picks it up in 3s. |
| Add/remove/rename a variant | `variants.js` | Nothing. Live reload picks it up in 3s. |
| Replace a variant (new SVG + new filename) | The `.svg` file + `variants.js` | Nothing. Both auto-reload. |
| Add a new concept group | `variants.js` | Nothing. Live reload picks it up in 3s. |
| Change project or brand name | `variants.js` | Nothing. Live reload picks it up in 3s. |

**Do NOT rewrite `preview.html`.** It is a static scaffold copied from the skill assets. All project-specific data lives in `variants.js`.

### Checklist

- **Copy `assets/preview.html`** to the project directory (do not modify it)
- **Write `variants.js`** with all variant data
- **Group by concept** (e.g., "Geometric", "Network / Graph", "Abstract")
- **Description per card** explaining the metaphor and what it conveys
- For colored logos, provide a `-dark.svg` variant (not CSS filters)
- For monochrome logos, omit the `dark` field (auto-applies invert filter)
- **Auto-open** the preview so the user sees options immediately
- On subsequent iterations, **only edit `variants.js`** and SVG files. Live reload handles the rest.

## Workflow Tips

1. **Start with shape primitives**, convert to paths only when needed for optimization or compound operations
2. **Use relative coordinates** (`m`, `l`, `c`) when hand-writing paths. Easier to reason about incrementally
3. **Test at multiple sizes**: render at 16px, 24px, 48px, and 200px to verify clean scaling
4. **Keep a monochrome version**: if using color, ensure it also works with a single `currentColor`
5. **Validate the output**: open in a browser, not just a code editor. Check for rendering artifacts
6. **Round coordinates to the grid**: snap to integers on 24x24 canvas for pixel-perfect rendering at 1x
