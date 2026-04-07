# Logo Design Techniques

## Principles of Clean, Scalable Logos

1. **Simplicity scales.** A logo must read clearly at 16px favicon size and 200px hero size. Eliminate detail that disappears at small sizes.
2. **Geometric construction.** Build from circles, rectangles, and triangles. Organic curves should still derive from geometric foundations.
3. **Consistent stroke weight.** If using strokes, keep them uniform unless intentional contrast is the design concept.
4. **Limited color palette.** 1-3 colors max. A good logo works in single-color (monochrome) form first, then gets color added.
5. **No raster effects.** Avoid filters, blur, drop shadows in the SVG itself. These don't scale cleanly and add file size.

## Typography in SVGs

### When to use `<text>`

- Internal tools, dashboards, or prototypes where the font is guaranteed
- Dynamic text that changes (user names, labels)
- SVGs that need to be searchable/indexable
- When file size matters (text is tiny compared to outlined paths)

### When to convert text to paths

- Logo wordmarks distributed as standalone files
- Any SVG that must render identically without font dependencies
- Icons containing letterforms (like a "B" for bold icon)
- Print/brand assets

**Trade-off:** Outlined text bloats file size significantly. A single word can go from ~200 bytes as `<text>` to 5-10KB as paths. Only outline when portability is required.

## Negative Space Techniques

Negative space creates shapes through absence rather than presence. This is how you create logos where two meanings coexist (like the FedEx arrow).

### Using fill-rule="evenodd"

The simplest way. Overlapping subpaths within a single `<path>` automatically create holes:

```xml
<!-- Circle with square hole (badge icon) -->
<path fill-rule="evenodd" d="
  M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z
  M 8 8 h 8 v 8 h -8 Z
" fill="currentColor" />
```

### Using clip-path

Cut one shape out of another:

```xml
<defs>
  <clipPath id="cut-hole">
    <path d="M 0 0 h 24 v 24 h -24 Z M 8 8 h 8 v 8 h -8 Z" fill-rule="evenodd" />
  </clipPath>
</defs>
<circle cx="12" cy="12" r="10" clip-path="url(#cut-hole)" fill="currentColor" />
```

### Using mask with white/black

White areas of a mask are visible, black areas are hidden:

```xml
<defs>
  <mask id="knockout">
    <rect width="24" height="24" fill="white" />
    <rect x="8" y="8" width="8" height="8" fill="black" />
  </mask>
</defs>
<circle cx="12" cy="12" r="10" mask="url(#knockout)" fill="currentColor" />
```

## Combining Geometric Shapes

### Layered composition

Stack shapes using document order (later elements render on top):

```xml
<!-- Shield logo -->
<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <!-- Shield body -->
  <path d="M 12 2 L 4 6 v 6 c 0 5 3.5 9.5 8 11 c 4.5 -1.5 8 -6 8 -11 V 6 Z" fill="#3B82F6" />
  <!-- Inner checkmark -->
  <path d="M 8 12 l 3 3 l 5 -6" stroke="white" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round" />
</svg>
```

### Compound path (single path, multiple subpaths)

Multiple `M` commands in one path create subpaths. Combined with `fill-rule="evenodd"`, overlapping areas become transparent:

```xml
<!-- Ring with notch -->
<path fill-rule="evenodd" d="
  M 12 1 A 11 11 0 1 1 12 23 A 11 11 0 1 1 12 1 Z
  M 12 5 A 7 7 0 1 1 12 19 A 7 7 0 1 1 12 5 Z
" fill="currentColor" />
```

## Creating Symmetrical Designs

### Mirror technique

Design half the shape, then mirror it. For horizontal symmetry, design the right half and negate x-offsets for the left:

```xml
<!-- Symmetrical leaf/wing using relative commands -->
<path d="
  M 12 4
  C 16 4 20 8 20 12
  C 20 16 16 20 12 20
  C 8 20 4 16 4 12
  C 4 8 8 4 12 4 Z
" />
```

### Using transform for symmetry

```xml
<g>
  <!-- Left wing -->
  <path d="M 12 8 Q 6 4 4 12 Q 6 16 12 16 Z" fill="currentColor" />
  <!-- Right wing (mirrored) -->
  <path d="M 12 8 Q 18 4 20 12 Q 18 16 12 16 Z" fill="currentColor" />
</g>
```

### Rotational symmetry with transform

```xml
<!-- 3-part rotational logo -->
<g fill="currentColor">
  <path d="M 12 4 L 14 10 L 12 12 Z" />
  <path d="M 12 4 L 14 10 L 12 12 Z" transform="rotate(120 12 12)" />
  <path d="M 12 4 L 14 10 L 12 12 Z" transform="rotate(240 12 12)" />
</g>
```

## Logo Ideation: Exploring Metaphors

Don't fixate on one concept. Before writing any SVG code, brainstorm 8-10 visual metaphors that represent the product. Then pick the 3-4 strongest and create variants of each.

### Metaphor brainstorming template

For a product, ask: "What does this product *do*, and what real-world objects share that quality?"

| Product quality | Possible metaphors |
|---|---|
| Connects things | Network graph, bridge, chain links, constellation |
| Organizes data | Stacked layers, filing cabinet, grid, table rows |
| Makes things accessible | Open book, unlocked padlock, broken circle, doorway |
| Aggregates sources | Funnel, magnet, venn overlap, tributary rivers |
| Discovers/searches | Compass, telescope, magnifying glass, radar |
| Stores/preserves | Cube, vault, crystal, container |
| Transforms/processes | Prism, gear, filter, pipeline |
| Grows/scales | Tree, rising bars, expanding circles, staircase |

### Common logo archetypes that work at small sizes

| Archetype | Why it works at 16px | Example |
|---|---|---|
| **Single geometric shape** (circle, hexagon, diamond) | One bold silhouette, no detail to lose | Stripe, Spotify |
| **Letterform** (stylized first letter) | Instantly recognizable, scales perfectly | Medium, Facebook |
| **Overlapping shapes** (venn, stacked) | 2-3 elements max, reads as a unit | Mastercard, Olympics |
| **Isometric object** (cube, box, stack) | 3 flat faces = 3 colors, very readable | Figma files icon |
| **Broken/open shape** (arc with gap, open bracket) | "Open" implied by the gap | OpenAI, open-source logos |
| **Abstract mark** (swoosh, asterisk, spark) | Pure shape, no literal meaning needed | Nike, Slack |

### What to avoid at small sizes

- Thin lines or strokes under 1.5px (on 32x32 viewBox)
- More than 6-7 distinct elements
- Text or letterforms with serifs
- Gradients with more than 2 stops (muddy at small sizes)
- Details that only appear above 48px

## Isometric and 3D Techniques

Isometric projection creates a sense of depth using three visible faces. No perspective distortion, so it scales cleanly.

### Basic isometric cube

Three parallelogram faces. The key angles: left face leans right, right face leans left, top face is a diamond.

```xml
<!-- viewBox 0 0 32 32 -->
<!-- Top face (lightest) -->
<path d="M 16 4 L 28 11 L 16 18 L 4 11 Z" fill="#38BDF8" />
<!-- Left face (medium) -->
<path d="M 4 11 L 16 18 L 16 28 L 4 21 Z" fill="#2563EB" />
<!-- Right face (darkest) -->
<path d="M 16 18 L 28 11 L 28 21 L 16 28 Z" fill="#1E3A5F" />
```

**Color convention:** Top = lightest (lit from above), left = medium, right = darkest. This creates convincing depth with flat colors.

### Stacked floating layers

Multiple diamond shapes floating above a base, suggesting data layers lifting out of a container.

```xml
<!-- Base box -->
<path d="M 4 18 L 16 23 L 28 18 L 16 13 Z" fill="#2563EB" opacity="0.25" />
<path d="M 4 18 L 16 23 L 16 30 L 4 25 Z" fill="#2563EB" />
<path d="M 16 23 L 28 18 L 28 25 L 16 30 Z" fill="#1E3A5F" />
<!-- Floating layer 1 -->
<path d="M 4 13 L 16 18 L 28 13 L 16 8 Z" fill="#10B981" />
<!-- Floating layer 2 -->
<path d="M 4 9 L 16 14 L 28 9 L 16 4 Z" fill="#38BDF8" />
```

### Spatial budgeting for multi-element compositions

Before coding, plan the vertical distribution to avoid clipping:

```
viewBox height: 32
Top padding:     2  (y=0 to y=2)
Layer 2:         4  (y=2 to y=6, diamond spans 4 units tall)
Gap:             2
Layer 1:         4  (y=8 to y=12)
Gap:             2
Box top:         5  (y=14 to y=19)
Box sides:       7  (y=19 to y=26)
Bottom padding:  2  (y=26 to y=32, but box extends to ~y=30)
```

Sketch this budget on paper first. Adjusting after the fact is tedious because moving one element means moving everything.

## Dark Mode Variants for Colored Logos

Logos with hardcoded colors need separate dark-mode SVG files. `currentColor` logos can use CSS `filter: brightness(0) invert(1)` but colored logos cannot.

### Color mapping for dark variants

| Element | Light mode | Dark mode | Why |
|---|---|---|---|
| Connection lines | `#1E3A5F` (dark navy) | `#4B8BBE` (medium blue) | Navy disappears on dark backgrounds |
| Node outer rings | `#1E3A5F` | `#3B6B8A` (steel blue) | Needs contrast against dark bg |
| White fills | `#FFFFFF` | `#E2E8F0` (off-white) | Pure white is harsh on dark bg |
| Colored fills | Same hex values | Same or slightly brighter | Colors already pop on dark |

### File naming convention

```
logo-color-blue.svg          # Light mode
logo-color-blue-dark.svg     # Dark mode variant
```

### Node ring + fill pattern (illustrated icon style)

A popular technique for colored logos: each node has a dark outer ring with a colored inner fill. Creates depth and reads well at small sizes.

```xml
<!-- Light mode node -->
<circle cx="12" cy="8" r="3" fill="#1E3A5F" />   <!-- outer ring -->
<circle cx="12" cy="8" r="2" fill="#38BDF8" />   <!-- inner fill -->

<!-- Dark mode equivalent -->
<circle cx="12" cy="8" r="3" fill="#3B6B8A" />   <!-- lighter ring -->
<circle cx="12" cy="8" r="2" fill="#38BDF8" />   <!-- same fill -->
```

## Logo File Checklist

Before shipping a logo SVG:

- [ ] Works at 16px (favicon), 32px, 64px, and 200px+
- [ ] Works in monochrome (single color)
- [ ] Works on both light and dark backgrounds (use `currentColor` or provide dark variants for colored logos)
- [ ] Dark variants created for any logo with hardcoded colors (see dark mode section above)
- [ ] No content clipping at viewBox edges (check all coordinates are within bounds)
- [ ] No embedded fonts (text converted to paths)
- [ ] No editor metadata or hidden layers
- [ ] `viewBox` is tight to the artwork (no excess whitespace)
- [ ] `xmlns` attribute present
- [ ] File is optimized (see optimization reference)
