# Advanced SVG Techniques

## Gradient Tips for Logos

- Use gradients sparingly. A logo should work in flat/monochrome first.
- Prefer 2-3 stops max. Complex gradients don't scale down well.
- Always provide a flat-color fallback version of gradient logos.
- Use `gradientUnits="userSpaceOnUse"` when applying the same gradient across multiple shapes to get a unified gradient rather than per-shape gradients.

## Clip-path vs Mask

| Feature | clip-path | mask |
|---------|-----------|------|
| Transparency | Binary (in or out) | Gradient (luminance-based) |
| Performance | Faster | Slower |
| Use case | Hard edges, shape cutouts | Fade effects, soft edges |
| Defined with | Shapes/paths | Any elements (including gradients) |

## When to Use Filters in Logos

Almost never. Filters are raster operations that:
- Don't scale cleanly at different sizes
- Increase rendering cost
- Add complexity to the SVG

If you need a shadow or glow on a logo, consider:
- Using a slightly offset duplicate shape with reduced opacity
- Applying the shadow via CSS `filter: drop-shadow()` on the container instead
- Creating the shadow effect with actual vector shapes

## SVG Transform vs CSS Transform

| Aspect | SVG `transform` attribute | CSS `transform` property |
|--------|--------------------------|-------------------------|
| Default origin | `0 0` (top-left of SVG canvas) | `50% 50%` (center of element) |
| Origin control | Via `rotate(angle, cx, cy)` parameter | Via `transform-origin` property |
| Priority | Lower (CSS overrides) | Higher |
| Animation | Harder (SMIL or JS) | Easier (CSS transitions/keyframes) |
| Units | SVG user units only | Supports px, %, em, etc. |

**When baking transforms into path data is better:** For distributed icons, flatten transforms into the path coordinates. This avoids rendering differences and makes the SVG simpler. Design tools do this on export. For dynamic/animated SVGs, keep transforms as attributes or CSS.
