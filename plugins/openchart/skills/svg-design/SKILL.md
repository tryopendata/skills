---
name: svg-design
description: Generates and edits SVG logos, icons, and graphics. Use when creating SVG files, designing logos or icons, writing path data, optimizing SVGs, building icon systems, animating SVG elements, or modifying existing vector graphics. Covers path commands, shape primitives, styling, accessibility, gradients, masks, sprites, optimization, and animation (CSS keyframes, GPU acceleration, staggering, easing, SVG-specific techniques).
---

# SVG Creation and Editing

**Core principle:** SVGs are code. Write them by hand like you'd write any markup: clean, minimal, semantically meaningful. Every element and attribute should earn its place.

## Topic Routing

| Task | Load reference |
|------|---------------|
| Writing or editing path `d` data | [references/path-commands.md](references/path-commands.md) |
| Choosing shapes vs paths, styling | [references/shapes-and-styling.md](references/shapes-and-styling.md) |
| Logo design, typography, negative space | [references/logo-techniques.md](references/logo-techniques.md) |
| Icon design, grid systems, pixel alignment | [references/icon-design.md](references/icon-design.md) |
| Gradients, masks, clips, filters, transforms | [references/advanced-techniques.md](references/advanced-techniques.md) |
| Animation (CSS keyframes, stagger, GPU, easing, SVG-specific) | [references/animation.md](references/animation.md) |
| Optimization, sprites, defs/use/symbol | [references/optimization.md](references/optimization.md) |
| Accessibility, common pitfalls | [references/accessibility-and-pitfalls.md](references/accessibility-and-pitfalls.md) |
| Editing workflow, combining shapes | [references/editing-workflow.md](references/editing-workflow.md) |

## SVG Skeleton

Always start from this structure:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
  <!-- content -->
</svg>
```

Adjust `viewBox` to match the design canvas. Omit `width`/`height` attributes to let the SVG scale with its container (add them only when a fixed size is needed).

## Canvas Size Conventions

| Size | Use case | Notes |
|------|----------|-------|
| `0 0 16 16` | Micro icons, favicons | Heroicons micro, GitHub Octicons |
| `0 0 20 20` | Small UI icons, form elements | Heroicons mini |
| `0 0 24 24` | Standard icons (most common) | Lucide, Heroicons outline, Material |
| `0 0 32 32` | Medium icons, navigation | Phosphor uses 256x256 internally |
| `0 0 48 48` | Large display icons | App icons, illustrations |
| Custom | Logos, illustrations | Match the natural aspect ratio |

**Default to 24x24** unless there's a reason not to. It's the industry standard.

## Shape vs Path Decision

| Use shape primitive when... | Use `<path>` when... |
|---|---|
| The shape is a basic geometric form | The shape has curves, complex outlines |
| Readability matters (a circle should look like `<circle>`) | You need to minimize element count |
| You need to animate individual properties (r, cx, cy) | Combining multiple shapes into one element |
| The shape will be programmatically modified | Exporting from design tools (paths are universal) |

## Styling Defaults

Set these on the root `<svg>` element to avoid repetition on children:

| Attribute | Icon default | Logo default | Why |
|-----------|-------------|--------------|-----|
| `fill` | `none` | varies | Icons are typically stroked, logos are filled |
| `stroke` | `currentColor` | `none` or `currentColor` | Inherits text color from parent |
| `stroke-width` | `2` (on 24x24) | varies | Consistent weight across icons |
| `stroke-linecap` | `round` | `round` or `butt` | Rounded ends look cleaner at small sizes |
| `stroke-linejoin` | `round` | `round` or `miter` | Prevents sharp spikes at joins |

**`currentColor` is your friend.** It lets the SVG inherit whatever color the parent element has, making icons themeable with zero extra CSS.

## Quick Path Reference

| Command | Name | Parameters | What it does |
|---------|------|-----------|--------------|
| `M`/`m` | Move to | `x y` | Move cursor (no drawing) |
| `L`/`l` | Line to | `x y` | Straight line |
| `H`/`h` | Horizontal line | `x` | Horizontal line |
| `V`/`v` | Vertical line | `y` | Vertical line |
| `C`/`c` | Cubic bezier | `x1 y1 x2 y2 x y` | Curve with 2 control points |
| `S`/`s` | Smooth cubic | `x2 y2 x y` | Continues previous curve smoothly |
| `Q`/`q` | Quadratic bezier | `x1 y1 x y` | Curve with 1 control point |
| `T`/`t` | Smooth quadratic | `x y` | Continues previous quadratic |
| `A`/`a` | Arc | `rx ry rotation large-arc sweep x y` | Elliptical arc |
| `Z`/`z` | Close path | (none) | Close back to last M |

Uppercase = absolute coordinates. Lowercase = relative (offset from current point).

## Logo Design Process

When creating logos (not icons), follow this process:

1. **Explore multiple metaphors, not multiple layouts of one metaphor.** If asked for 10 logo options, don't make 10 network graphs. Make 3 network graphs, 2 isometric shapes, 2 abstract symbols, etc. Conceptual diversity matters more than layout variations.
2. **Start with what the product *does*, not what it *looks like*.** A data platform could be: a network graph, stacked layers, a prism, a compass, an open book, overlapping circles, a rising chart, a cube, a broken ring. List 8-10 metaphors before drawing anything.
3. **Build an HTML preview page immediately.** Show every variant at 16px, 32px, and 64px on both light and dark backgrounds, plus a nav-bar mockup with the wordmark. This accelerates feedback cycles dramatically.
4. **For colored logos, always create a `-dark.svg` variant.** Dark navy edges (#1E3A5F) that look great on white disappear on dark backgrounds. Dark variants need lighter edges (#4B8BBE), lighter rings (#3B6B8A), and off-white centers (#E2E8F0).
5. **Plan your vertical budget before drawing.** On a 32x32 canvas with 3 stacked elements, you have ~30 usable units. Sketch the vertical distribution first (e.g., box=12, gap=2, layer=5, gap=2, layer=5) to avoid clipping at viewBox edges.

## Anti-Patterns

| Don't | Do instead |
|-------|-----------|
| Hardcode `width="24" height="24"` without `viewBox` | Use `viewBox` always; add width/height only if needed |
| Set `fill="none"` on a `<g>` group | Set fill on individual elements or the root `<svg>` |
| Use `px` units inside SVG | SVG coordinates are unitless; they map to viewBox |
| Include editor metadata (`<sodipodi:*>`, `<inkscape:*>`) | Strip all editor cruft |
| Use `<text>` for logo wordmarks in distributed SVGs | Convert text to paths for portability |
| Nest transforms three levels deep | Flatten transforms into path coordinates |
| Use `xlink:href` | Use `href` (xlink is deprecated) |
| Forget `xmlns` on standalone SVG files | Always include `xmlns="http://www.w3.org/2000/svg"` |
| Use decimal precision beyond 2-3 places for icons | Round to 2 decimals for icons, 3 max for complex art |

## Cross-References

- **Design aesthetics**: If the `ce` plugin is installed, `Skill(ce:design)` covers broader visual design decisions
- **Mermaid diagrams**: If the `ce` plugin is installed, `Skill(ce:visualizing-with-mermaid)` covers diagram-specific visualizations (not SVG)
