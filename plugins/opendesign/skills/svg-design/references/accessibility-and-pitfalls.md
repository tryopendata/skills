# Accessibility and Common Pitfalls

## Accessibility Quick Reference

| SVG role | Implementation |
|----------|---------------|
| Decorative (next to text) | `aria-hidden="true"` on svg |
| Informative (standalone icon) | `role="img"` + `<title>` + `aria-labelledby` |
| Complex (illustration) | `role="img"` + `<title>` + `<desc>` + `aria-labelledby` |
| Inside a button | Button gets `aria-label`, SVG gets `aria-hidden="true"` |
| Focusable but shouldn't be | Add `focusable="false"` (IE/Edge legacy issue) |

`<title>` must be the **first child** of its parent element. Use `role="img"` to tell screen readers to treat the SVG as a single image.

## Common Pitfalls

### stroke-width scales with viewBox

`stroke-width` is in viewBox units, not pixels. A `stroke-width="2"` on a `viewBox="0 0 24 24"` SVG rendered at 48px appears as 4px thick. Use `vector-effect="non-scaling-stroke"` only for maps/technical drawings where you need fixed-pixel strokes.

### fill="none" on groups overrides children

Setting `fill="none"` on a `<g>` group makes all children without explicit `fill` invisible. Set fill on individual elements or the root `<svg>`, not on groups.

### Missing width/height causes 0x0 in some contexts

SVGs without `width`/`height` attributes render as 0x0 in:
- Flexbox containers (Safari)
- Absolutely positioned elements
- `<img>` tags without CSS sizing
- Email clients

For `<img>` usage, always provide `width` and `height` attributes.

### viewBox mismatch clips content

Content extending beyond viewBox boundaries gets clipped. Either adjust the viewBox to contain all content, or add `overflow="visible"` (but this can cause layout issues).

### Stroke extends beyond the path

Strokes are centered on the path. A `stroke-width="2"` extends 1 unit on each side. If your path touches the viewBox edge, the stroke gets clipped. Inset shapes by half the stroke width. This is why icon guidelines specify 1-2px padding from the viewBox edge.

### Browser rendering differences

| Issue | Browsers affected | Workaround |
|-------|-------------------|------------|
| SVG in flexbox renders at 0x0 | Safari | Add explicit `width`/`height` |
| `<use>` with external file doesn't work cross-origin | Safari, older Firefox | Inline the sprite or use same-origin |
| `transform-origin` default differs | Firefox vs Chrome (historically) | Always set `transform-origin` explicitly |
| CSS `filter: drop-shadow()` clips at viewBox | Some browsers | Add padding to viewBox |
| `<text>` font rendering varies | All browsers | Convert to paths for exact rendering |

### Colored SVGs invisible on dark backgrounds

Logos with hardcoded dark colors (like `#1E3A5F` for edges) disappear on dark backgrounds. CSS `filter: brightness(0) invert(1)` only works for monochrome SVGs. Create separate `-dark.svg` variants with lighter structural colors. Keep colored fills the same; lighten edges, rings, and connection lines.

```
logo-brand.svg        ->  edges: #1E3A5F, rings: #1E3A5F, centers: white
logo-brand-dark.svg   ->  edges: #4B8BBE, rings: #3B6B8A, centers: #E2E8F0
```
