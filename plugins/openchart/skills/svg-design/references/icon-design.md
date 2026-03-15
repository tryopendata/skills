# Icon Design Principles

## Industry Conventions by Library

| Library | viewBox | Stroke width | Linecap | Linejoin | Fill | Sizes |
|---------|---------|-------------|---------|----------|------|-------|
| **Lucide** | 0 0 24 24 | 2 | round | round | none | 24 |
| **Heroicons outline** | 0 0 24 24 | 1.5 | round | round | none | 24 |
| **Heroicons solid** | 0 0 24 24 | n/a | n/a | n/a | #0F172A | 24 |
| **Heroicons mini** | 0 0 20 20 | n/a | n/a | n/a | #0F172A | 20 |
| **Heroicons micro** | 0 0 16 16 | n/a | n/a | n/a | #0F172A | 16 |
| **Material** | 0 0 24 24 | 2 | n/a | n/a | varies | 24 |
| **Phosphor** | 0 0 256 256 | ~16 | round | round | none | 256 (scaled) |

### Standard SVG Skeleton (Lucide convention, most widely adopted)

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24"
  viewBox="0 0 24 24" fill="none" stroke="currentColor"
  stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
  <!-- icon content -->
</svg>
```

### Heroicons outline convention

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24"
  viewBox="0 0 24 24" fill="none" stroke="currentColor"
  stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round">
  <!-- icon content -->
</svg>
```

### Heroicons solid/mini/micro convention

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24"
  viewBox="0 0 24 24" fill="currentColor">
  <!-- filled paths, often with fill-rule="evenodd" -->
</svg>
```

## Pixel-Perfect Alignment

Icons at small sizes (16-24px) render on actual device pixels. Misalignment causes blurry edges.

### Rules for sharp rendering

1. **Snap to integer coordinates.** Points at `x=12` render sharply. Points at `x=12.5` may blur.
2. **Account for stroke width.** A stroke-width of 2 on a line at `y=12` extends 1px above and 1px below (from y=11 to y=13). The line center should be on an integer.
3. **Odd stroke widths need half-pixel positioning.** A stroke-width of 1 centered on `y=12` extends from 11.5 to 12.5, which doesn't align to pixels. Center it on `y=12.5` instead.
4. **Use whole-number dimensions.** Icon containers should be integer widths/heights.

### Stroke alignment cheat sheet

| Stroke width | Position line center at | Why |
|-------------|------------------------|-----|
| 1 | x.5 (half pixel) | 0.5 + 0.5 = fills exactly 1 pixel |
| 1.5 | integer | 0.75 on each side, rounds to 1px each |
| 2 | integer | 1 + 1 = fills exactly 2 pixels |
| 3 | x.5 (half pixel) | 1.5 + 1.5 = fills exactly 3 pixels |

## Consistent Stroke Weights

Within an icon set, every icon should use the same stroke width. This creates visual cohesion.

**For a 24x24 grid:**
- Lucide: `stroke-width="2"` everywhere
- Heroicons: `stroke-width="1.5"` everywhere
- Never mix weights within a single icon unless it's a deliberate design choice (like a bold accent)

**For filled icons:** Use uniform path widths. A "line" in a filled icon is a rectangle or path with consistent width. Measure your paths to ensure uniformity.

## Optical Balance and Visual Weight

Not all shapes have equal visual weight at the same physical size. Compensate:

| Shape | Adjustment needed |
|-------|------------------|
| Circle | Slightly larger than a square to look the same size |
| Triangle | Needs to be taller than a square to match visual weight |
| Horizontal line | Looks lighter than vertical line of same dimensions |
| Detailed icon | Looks heavier than simple icon at same size |

### The blur test

Squint at your icon or apply a gaussian blur. If it looks significantly darker/lighter than neighboring icons, adjust the visual weight. A set of icons should look roughly equally "dense" when blurred.

### Optical centering

Mathematical center != visual center. A "play" triangle centered mathematically looks too far left. Shift it slightly right to look visually centered. An arrow pointing right needs to be shifted a few units right.

**General rule:** Shift directional shapes ~1-2 units (on 24x24 grid) in their "pointing" direction.

## Icon Grid System

Material Design defines a grid system that most icon sets loosely follow:

### The 24x24 grid zones

```
+------------------------+
|  1px padding (all sides)|
|  +------------------+  |
|  |   20x20 live     |  |
|  |    area          |  |
|  |                  |  |
|  |   Content goes   |  |
|  |   here           |  |
|  +------------------+  |
+------------------------+
```

- **Trim area:** Full 24x24 canvas. Nothing should extend beyond this.
- **Live area:** 20x20 (2px padding on each side). All icon content should fit within this zone.
- **Padding:** 2px minimum on all sides. Some icons (circles, squares) may use the full live area; others should have more breathing room.

### Keyline shapes (Material Design)

These are reference shapes that define the "bounding box" of different icon types:

| Shape | Dimensions (within 20x20 live area) | Use for |
|-------|-------------------------------------|---------|
| Circle | 20px diameter | Round icons (globe, user avatar) |
| Square | 18x18 | Square icons (file, card) |
| Vertical rectangle | 16x20 | Tall icons (document, phone) |
| Horizontal rectangle | 20x16 | Wide icons (landscape, video) |

The keylines ensure different-shaped icons occupy similar visual space.

### Corner radius

| Element size | Corner radius |
|-------------|---------------|
| >= 8px | 2px |
| < 8px | 1px |
| Interior corners | 0px (square) |

## Allowed SVG Elements (Lucide convention)

Lucide is the most restrictive and produces the cleanest output:

**Allowed:**
- `<path>` with `d` attribute
- `<line>` with coordinate attributes
- `<polyline>` with `points`
- `<polygon>` with `points`
- `<circle>` with `cx`, `cy`, `r`
- `<ellipse>` with `cx`, `cy`, `rx`, `ry`
- `<rect>` with `x`, `y`, `width`, `height`, `rx`

**Prohibited:**
- `<g>` groups (flatten everything)
- `transform` attributes (bake transforms into coordinates)
- `<use>` references
- `<defs>` definitions
- `<filter>` effects
- Inline `fill` or `stroke` on individual elements (set on root `<svg>`)
- `<text>` elements
- `style` attributes or `<style>` blocks

This strictness ensures maximum compatibility and the smallest possible file size. When building an icon set, follow these constraints. For standalone icons or logos, you can be more flexible.

## Naming Conventions

| Convention | Example |
|-----------|---------|
| Lowercase kebab-case | `arrow-up-right.svg` |
| Describe appearance, not function | `arrow-up` not `scroll-to-top` |
| Group variants | `chevron-left`, `chevron-right`, `chevron-up` |
| Size elements largest-to-smallest | `circle-dot` (circle is bigger than dot) |
| No numerals (unless the icon shows a number) | `calendar` not `calendar1` |
