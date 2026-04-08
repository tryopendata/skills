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

### Step 1: Mine the domain

The biggest source of generic logos is skipping this step. Before thinking about shapes, list 10-15 physical objects, tools, environments, textures, and actions that are *specific to this product's world*:

- A marine logistics company: knots, anchors, currents, hull cross-sections, container stacks, signal flags, wake patterns, bollards, cargo nets
- A code editor: cursors, brackets, indentation, diff markers, tree structures, merge arrows, syntax highlighting bands, terminal prompts
- A bakery: wheat stalks, dough scoring patterns, rolling pins, oven arches, braided loaves, flour dusting, banneton spiral imprints

These domain objects become the raw material. If you can't list 10, you don't know the domain well enough yet.

### Step 2: Semantic branching (literal -> abstract -> unexpected)

For each promising domain object, push through three levels of abstraction:

| Level | Example (coffee roaster) | What it produces |
|---|---|---|
| **Literal** | A coffee bean shape | Recognizable but generic, likely overused |
| **Abstract** | The bean's center crease as a single curved line | Distinctive, still evocative |
| **Unexpected** | The crease becomes a sound wave (roasting = transformation) | Unique, carries a second meaning |

The best logos live at level 2-3. Level 1 is where cliches come from. Always push at least to level 2.

### Step 3: Check against industry cliches

Before finalizing concepts, reject any that match common overused motifs:

| Industry | Overused motifs to avoid |
|---|---|
| Tech/SaaS | Hexagons, circuit boards, generic nodes-and-edges, globe with lines |
| Finance | Bar charts going up, shield shapes, dollar signs, generic buildings |
| Health/wellness | Hearts, crosses, leaves, human silhouettes with arms up |
| Education | Open books, graduation caps, lightbulbs, apples |
| Food/restaurant | Fork-and-knife, chef hats, plates, steam swirls |
| Creative/design | Pencils, color wheels, paintbrushes, eye symbols |
| Environment/green | Leaves, globes, trees, recycling arrows |

If a concept matches this list, push it to level 2-3 abstraction or replace it entirely.

### Step 4: Guarantee category diversity

Every logo set of 5+ options must include concepts from different *structural categories*, not just different metaphors rendered the same way:

| Category | What it means | Forces you to... |
|---|---|---|
| **Typographic/wordmark** | The brand name IS the logo, styled distinctively | Explore letterform design, ligatures, custom strokes |
| **Symbolic icon** | A recognizable object or scene, simplified | Think about what single image represents the brand |
| **Abstract geometric** | Non-representational shapes, patterns, or compositions | Work with pure form, color, and spatial relationships |
| **Letterform + metaphor** | A letter that doubles as a visual concept | Find where typography and meaning intersect |
| **Negative space / dual-read** | Two meanings coexist in one mark (FedEx arrow style) | Think about figure-ground relationships |

Include at least 3 of these 5 categories. If all your concepts are symbolic icons, the set lacks structural variety regardless of how different the symbols are.

### Step 5: SCAMPER one concept to push it further

Pick your most promising concept and run it through these lenses to generate unexpected variants:

- **Substitute**: Swap the expected symbol for something from a different domain
- **Combine**: Merge two unrelated visual ideas into one mark
- **Adapt**: Borrow visual language from another industry entirely
- **Modify**: Push scale, weight, or proportion to an extreme
- **Put to other use**: Use typography as illustration, or vice versa
- **Eliminate**: Strip to the absolute minimum recognizable form
- **Reverse**: Flip figure/ground, invert the concept, use negative space

You don't need all 7. Even running 2-3 lenses on one concept often produces the most distinctive option in the set.

### Rendering techniques that work at small sizes

These are techniques for *how* to render a concept, not concepts themselves. Pair with domain-specific metaphors:

| Technique | Why it works at 16px | Example |
|---|---|---|
| **Single bold silhouette** | One shape, no detail to lose | Stripe, Spotify |
| **Stylized letterform** | Instantly recognizable, scales perfectly | Medium, Facebook |
| **Overlapping shapes** (2-3 max) | Reads as a unit | Mastercard, Olympics |
| **Isometric projection** | 3 flat faces = 3 colors, very readable | Figma files icon |
| **Broken/open shape** (gap implies meaning) | The absence carries the concept | OpenAI |
| **Abstract mark** | Pure shape, no literal meaning needed | Nike, Slack |
| **Negative space cutout** | Two meanings coexist | FedEx arrow, NBC peacock |
| **Contained symbol** (shape inside a frame) | Frame provides structure at small sizes | Instagram, App Store |

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
