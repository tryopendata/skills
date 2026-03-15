# Accessibility and Common Pitfalls

## Common Pitfalls

### 1. Missing xmlns attribute

**Problem:** SVG files used outside HTML (as `<img src>`, in CSS `url()`, or as standalone files) must have the namespace.

```xml
<!-- Breaks when loaded as image or in CSS -->
<svg viewBox="0 0 24 24">...</svg>

<!-- Works everywhere -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">...</svg>
```

When inlined directly in HTML5, `xmlns` is technically optional because the HTML parser infers it. But always include it for portability.

### 2. Hardcoded dimensions without viewBox

**Problem:** Setting `width="24" height="24"` without `viewBox` locks the SVG to exactly 24px. It won't scale.

```xml
<!-- Can't scale -->
<svg width="24" height="24">...</svg>

<!-- Scales to any container size -->
<svg viewBox="0 0 24 24">...</svg>

<!-- Fixed default size, but still scalable via CSS -->
<svg width="24" height="24" viewBox="0 0 24 24">...</svg>
```

**Rule:** Always include `viewBox`. Add `width`/`height` only when you want a default intrinsic size.

### 3. stroke-width scales with viewBox

**Problem:** `stroke-width` is in viewBox units, not pixels. Changing the viewBox or scaling the SVG changes the apparent stroke width.

A `stroke-width="2"` on a `viewBox="0 0 24 24"` SVG rendered at 48px will appear as 4px thick. At 12px, it appears as 1px thick. This is usually desirable, but if you need a fixed-pixel stroke regardless of scaling:

```xml
<!-- Non-scaling stroke (CSS) -->
<style>
  path { vector-effect: non-scaling-stroke; }
</style>
```

Or use `vector-effect="non-scaling-stroke"` as an attribute. But this is rarely what you want for icons. It's more useful for maps and technical drawings.

### 4. fill="none" on groups overriding children

**Problem:** Setting `fill="none"` on a `<g>` group applies to all children, even if those children should have fills.

```xml
<!-- Bug: the rect inherits fill="none" from the group -->
<g fill="none">
  <rect x="2" y="2" width="20" height="20" fill="blue" />  <!-- This works (explicit override) -->
  <circle cx="12" cy="12" r="5" />                           <!-- This is invisible! -->
</g>
```

**Fix:** Either set `fill` on individual elements, or set it on the root `<svg>` and override only where needed.

### 5. Missing width/height causes 0x0 in some contexts

**Problem:** SVGs without `width`/`height` attributes can render as 0x0 in:
- Flexbox containers (Safari)
- Absolutely positioned elements
- `<img>` tags without CSS sizing
- Email clients

**Fix:** For `<img>` usage, always provide `width` and `height` attributes (or CSS). For inline SVGs, `viewBox` alone is usually fine since the SVG takes its container's width.

### 6. viewBox mismatch clips content

**Problem:** If your content extends beyond the viewBox boundaries, it gets clipped.

```xml
<!-- Circle at r=12 extends from 0 to 24, but content at x=-2 is clipped -->
<svg viewBox="0 0 24 24">
  <circle cx="12" cy="12" r="14" />  <!-- Extends beyond viewBox, gets clipped -->
</svg>
```

**Fix:** Either adjust the viewBox to contain all content, or add `overflow="visible"` (but this can cause layout issues).

### 7. Using xlink:href (deprecated)

```xml
<!-- Old (deprecated) -->
<use xlink:href="#icon" />
<image xlink:href="photo.jpg" />

<!-- New -->
<use href="#icon" />
<image href="photo.jpg" />
```

The `xlink:` prefix has been deprecated since SVG 2. Modern browsers support plain `href`. If you need to support very old browsers, include both.

### 8. Browser rendering differences

| Issue | Browsers affected | Workaround |
|-------|-------------------|------------|
| SVG in flexbox renders at 0x0 | Safari | Add explicit `width`/`height` |
| `<use>` with external file doesn't work cross-origin | Safari, older Firefox | Inline the sprite or use same-origin |
| `transform-origin` default differs | Firefox vs Chrome (historically) | Always set `transform-origin` explicitly |
| CSS `filter: drop-shadow()` clips at viewBox | Some browsers | Add padding to viewBox |
| `<text>` font rendering varies | All browsers | Convert to paths for exact rendering |
| `preserveAspectRatio` interpretation | Edge cases | Test with `xMidYMid meet` (default) |

### 9. Color inheritance gotchas

```xml
<!-- Problem: inline fill overrides CSS -->
<svg>
  <path fill="#000000" d="..." />  <!-- This ignores any CSS `fill` rule -->
</svg>

<!-- Fix: remove inline fill, use currentColor or CSS -->
<svg fill="currentColor">
  <path d="..." />
</svg>
```

Inline attributes have higher specificity than external CSS (unless the CSS uses `!important`). Design tools export with inline fills. Strip them if you want CSS control.

### 10. Forgetting stroke extends beyond the path

Strokes are centered on the path by default. A `stroke-width="2"` extends 1 unit on each side of the path. If your path touches the viewBox edge, the stroke will be clipped:

```xml
<!-- Bug: stroke extends beyond viewBox -->
<svg viewBox="0 0 24 24">
  <rect x="0" y="0" width="24" height="24" stroke="black" stroke-width="2" fill="none" />
  <!-- The outer half of the stroke is clipped -->
</svg>

<!-- Fix: inset the shape by half the stroke width -->
<svg viewBox="0 0 24 24">
  <rect x="1" y="1" width="22" height="22" stroke="black" stroke-width="2" fill="none" />
</svg>
```

This is why icon design guidelines specify 1-2px padding from the viewBox edge.

### 11. Colored SVGs invisible on dark backgrounds

**Problem:** Logos with hardcoded dark colors (like `#1E3A5F` for edges) look great on white but disappear on dark backgrounds. CSS `filter: brightness(0) invert(1)` only works for monochrome SVGs -- it mangles multi-color logos.

**Fix:** Create separate `-dark.svg` variants with lighter structural colors. Keep the colored fills the same (they already pop on dark) but lighten edges, rings, and connection lines.

```
logo-brand.svg        →  edges: #1E3A5F, rings: #1E3A5F, centers: white
logo-brand-dark.svg   →  edges: #4B8BBE, rings: #3B6B8A, centers: #E2E8F0
```

See `references/logo-techniques.md` for the full dark mode color mapping table.

## Accessibility Quick Reference

See the advanced-techniques reference for full accessibility patterns. Summary:

| SVG role | Implementation |
|----------|---------------|
| Decorative (next to text) | `aria-hidden="true"` |
| Informative (standalone icon) | `role="img"` + `<title>` + `aria-labelledby` |
| Complex (illustration) | `role="img"` + `<title>` + `<desc>` + `aria-labelledby` |
| Inside a button | Button gets `aria-label`, SVG gets `aria-hidden="true"` |
| Focusable but shouldn't be | Add `focusable="false"` (IE/Edge legacy issue) |
