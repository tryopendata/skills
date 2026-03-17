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

When creating multiple logo/icon options, build an HTML preview page alongside the SVGs and auto-open it (`open preview.html` on macOS, `xdg-open` on Linux). This is the single most impactful thing for fast iteration.

The preview page should include:

### Required sections per card

1. **Size ramp** (16, 32, 64px) on a white background with rounded corners, to catch detail that doesn't scale and remain visible when the page is viewed in dark contexts
2. **Dark background row** (16, 32, 64px) using the actual site dark bg color, matching the same three sizes as the light row for symmetry
3. **Fixed-height description** so cards align symmetrically even when descriptions vary in length (`min-height: 2.5em` on the description paragraph)
4. **Realistic context mockups** showing the logo where it will actually live:

| Mockup | What to show | Why |
|--------|-------------|-----|
| Nav bar | Logo (28px) + wordmark, light and dark | Most common placement, tests logo+text pairing |
| Browser tab | 16px favicon in a tab-bar strip | Tests extreme small size legibility |
| App icon | 48-64px in a rounded-rect frame (iOS/Android style) | Tests standalone recognition without wordmark |
| Hero/splash | 120-200px centered on a landing page mockup | Tests whether detail holds up large |

Include at least the nav bar and favicon mockups. Add app icon and hero if the logo will be used in those contexts.

### Interactive features

The template below includes two interactive features (no dependencies, vanilla JS):

- **Click-to-compare:** Clicking a card pins it to a sticky bar at the bottom (up to 4). Click again to unpin. This lets the user evaluate finalists side by side without scrolling.
- **Live reload:** A 3-second polling loop cache-busts all SVG image sources, so edits to SVG files appear in the browser without manual refresh.

### Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Project Name Logo Options</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body { font-family: system-ui, sans-serif; padding: 40px 40px 120px; background: #f8f9fa; }
  h1 { margin-bottom: 8px; font-size: 24px; }
  .sub { color: #666; margin-bottom: 32px; }
  .section { margin-bottom: 48px; }
  .section h2 { font-size: 16px; color: #888; text-transform: uppercase; letter-spacing: 0.05em;
    margin-bottom: 16px; border-bottom: 1px solid #ddd; padding-bottom: 8px; }
  .grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); gap: 24px; }
  .card { background: white; border-radius: 12px; padding: 12px; border: 1px solid #e5e7eb;
    cursor: pointer; transition: outline 0.15s; }
  .card h3 { font-size: 13px; font-weight: 600; margin-bottom: 4px; }
  .card p { font-size: 11px; color: #9ca3af; margin-bottom: 12px; line-height: 1.4;
    min-height: 2.5em; }
  .sizes { display: flex; align-items: flex-end; gap: 16px;
    background: white; border-radius: 8px; padding: 12px; }
  .size-label { font-size: 10px; color: #9ca3af; text-align: center; }
  .size-label img { display: block; margin: 0 auto 4px; }
  .dark-row { background: #0d1117; padding: 12px; border-radius: 8px; margin-top: 8px;
    display: flex; align-items: flex-end; gap: 16px; }
  .dark-row .size-label { color: #666; }

  /* Context mockups */
  .nav-preview { display: flex; align-items: center; gap: 8px; margin-top: 12px;
    padding: 8px 12px; border: 1px solid #e5e7eb; border-radius: 6px; background: #fafafa; }
  .nav-preview img { width: 28px; height: 28px; }
  .nav-preview span { font-size: 16px; font-weight: 700; }
  .nav-dark { background: #0d1117; border-color: #30363d; }
  .nav-dark span { color: #f0f6fc; }
  .favicon-strip { display: flex; align-items: center; gap: 6px; margin-top: 8px;
    padding: 6px 10px; background: #dee1e6; border-radius: 8px 8px 0 0; }
  .favicon-strip .tab { display: flex; align-items: center; gap: 6px; background: white;
    border-radius: 8px 8px 0 0; padding: 6px 12px; font-size: 11px; color: #374151; }
  .favicon-strip .tab img { width: 16px; height: 16px; }

  /* Compare bar */
  #compare-bar { position: fixed; bottom: 0; left: 0; right: 0; background: white;
    border-top: 2px solid #e5e7eb; padding: 16px 40px; display: flex; gap: 24px;
    align-items: center; z-index: 10; min-height: 80px; }
</style>
</head>
<body>
<h1>Project Name Logo Options</h1>
<p class="sub">Click any card to pin it to the comparison bar</p>

<!-- Group variants by concept -->
<div class="section">
  <h2>Concept Group Name</h2>
  <div class="grid">
    <div class="card">
      <h3>01 - Variant Name</h3>
      <p>Short metaphor description. What it evokes.</p>
      <div class="sizes">
        <div class="size-label"><img src="logo.svg" width="16" height="16">16</div>
        <div class="size-label"><img src="logo.svg" width="32" height="32">32</div>
        <div class="size-label"><img src="logo.svg" width="64" height="64">64</div>
      </div>
      <div class="dark-row">
        <div class="size-label"><img src="logo-dark.svg" width="16" height="16">16</div>
        <div class="size-label"><img src="logo-dark.svg" width="32" height="32">32</div>
        <div class="size-label"><img src="logo-dark.svg" width="64" height="64">64</div>
      </div>
      <!-- Context mockups -->
      <div class="favicon-strip">
        <div class="tab"><img src="logo.svg">Project Name</div>
      </div>
      <div class="nav-preview"><img src="logo.svg"><span>BrandName</span></div>
      <div class="nav-preview nav-dark"><img src="logo-dark.svg"><span>BrandName</span></div>
    </div>
  </div>
</div>

<!-- Comparison strip -->
<div id="compare-bar">
  <span style="font-size:12px;color:#9ca3af;white-space:nowrap;">Click cards to compare</span>
</div>

<script>
// Click-to-compare
document.querySelectorAll('.card').forEach(card => {
  card.addEventListener('click', () => {
    const bar = document.getElementById('compare-bar');
    const name = card.querySelector('h3')?.textContent || '';
    const imgSrc = card.querySelector('.sizes img')?.src;
    if (!imgSrc) return;
    const existing = bar.querySelector(`[data-name="${CSS.escape(name)}"]`);
    if (existing) { existing.remove(); card.style.outline = ''; return; }
    if (bar.querySelectorAll('.compare-item').length >= 4) return;
    const item = document.createElement('div');
    item.className = 'compare-item';
    item.dataset.name = name;
    item.style.cssText = 'text-align:center;';
    const thumb = document.createElement('img');
    thumb.src = imgSrc; thumb.width = 48; thumb.height = 48;
    thumb.style.cssText = 'display:block;margin:0 auto 4px;';
    const label = document.createElement('div');
    label.style.cssText = 'font-size:11px;color:#374151;';
    label.textContent = name;
    item.append(thumb, label);
    bar.appendChild(item);
    card.style.outline = '3px solid #3B82F6';
    card.style.outlineOffset = '-3px';
  });
});
// Live reload: poll every 3s, bust image cache
setInterval(() => {
  const t = Date.now();
  document.querySelectorAll('img[src*=".svg"]').forEach(img => {
    const base = img.src.split('?')[0];
    img.src = base + '?t=' + t;
  });
}, 3000);
</script>
</body>
</html>
```

### Checklist

- **Group by concept** (e.g., "Geometric", "Network / Graph", "Abstract") so the user evaluates ideas, not just shapes
- **Description per card** explaining the metaphor and what it conveys
- **Three sizes** (16, 32, 64) on a white-background container to catch detail that doesn't scale
- **Dark background row** with the same three sizes (16, 32, 64) for symmetry
- **Favicon mockup** showing 16px logo in a browser tab strip
- **Nav bar mockup** showing the logo next to the wordmark at realistic size, light and dark
- For colored logos, use the `-dark.svg` variant in dark rows (not CSS filters)
- For monochrome logos, use `style="filter:brightness(0) invert(1)"` in dark rows
- **Comparison strip** with click-to-pin for side-by-side evaluation
- **Live reload** via cache-busting poll (no dependencies)
- **Auto-open** the preview file so the user sees options immediately

## Workflow Tips

1. **Start with shape primitives**, convert to paths only when needed for optimization or compound operations
2. **Use relative coordinates** (`m`, `l`, `c`) when hand-writing paths. Easier to reason about incrementally
3. **Test at multiple sizes**: render at 16px, 24px, 48px, and 200px to verify clean scaling
4. **Keep a monochrome version**: if using color, ensure it also works with a single `currentColor`
5. **Validate the output**: open in a browser, not just a code editor. Check for rendering artifacts
6. **Round coordinates to the grid**: snap to integers on 24x24 canvas for pixel-perfect rendering at 1x
