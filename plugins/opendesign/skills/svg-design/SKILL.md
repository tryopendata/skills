---
name: svg-design
description: Generates and edits SVG logos, icons, and graphics. Use when creating SVG files, designing logos or icons, writing path data, optimizing SVGs, building icon systems, animating SVG elements, or modifying existing vector graphics. Covers path commands, shape primitives, styling, accessibility, gradients, masks, sprites, optimization, and animation (CSS keyframes, GPU acceleration, staggering, easing, SVG-specific techniques).
---

# SVG Creation and Editing

**Core principle:** SVGs are code. Write them by hand like you'd write any markup: clean, minimal, semantically meaningful. Every element and attribute should earn its place.

## Topic Routing

| Task | Load reference |
|------|---------------|
| Arc flag combinations, common path shapes | [references/path-patterns.md](references/path-patterns.md) |
| Logo design, typography, negative space | [references/logo-techniques.md](references/logo-techniques.md) |
| Icon design, grid systems, pixel alignment | [references/icon-design.md](references/icon-design.md) |
| Gradients, masks, clips, filters, transforms (design decisions) | [references/advanced-techniques.md](references/advanced-techniques.md) |
| Animation (CSS keyframes, stagger, GPU, easing, SVG-specific) | [references/animation.md](references/animation.md) |
| Optimization, sprites, SVGO config | [references/optimization.md](references/optimization.md) |
| Accessibility, browser pitfalls | [references/accessibility-and-pitfalls.md](references/accessibility-and-pitfalls.md) |
| Editing workflow, boolean operations, combining SVGs | [references/editing-workflow.md](references/editing-workflow.md) |

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

## Stroke-Width Relative to ViewBox

`stroke-width` is in viewBox units, not pixels. Always set relative to your canvas dimensions:

| viewBox | Typical stroke-width | Visual result |
|---------|---------------------|---------------|
| 16x16 | 1.5 | ~9.4% of canvas |
| 24x24 | 2 | ~8.3% of canvas |
| 32x32 | 2-2.5 | ~6.3-7.8% of canvas |
| 48x48 | 3 | ~6.3% of canvas |
| 256x256 | 16 | ~6.3% of canvas |

## When to Convert Shapes to Paths

**Convert when:**
- Combining multiple shapes into a single compound path (boolean operations)
- You need the shape as part of a larger path composition
- Distributing to environments that only support `<path>`
- Optimizing for minimal DOM elements

**Keep as shapes when:**
- Code readability matters (a `<circle>` is self-documenting)
- You need to animate specific properties (animating `r` on a circle is cleaner than animating path data)
- The SVG will be programmatically modified

### Rounded rect-to-path template

The arc parameters here are error-prone to derive. Use this as a reference:

```xml
<!-- <rect x="2" y="2" width="20" height="20" rx="3" /> becomes: -->
<path d="M 5 2 h 14 a 3 3 0 0 1 3 3 v 14 a 3 3 0 0 1 -3 3 h -14 a 3 3 0 0 1 -3 -3 v -14 a 3 3 0 0 1 3 -3 Z" />
```

## fill-rule: evenodd vs nonzero

Use `evenodd` when you have compound shapes with holes. It's simpler because you don't need to worry about winding direction:

```xml
<!-- Donut using evenodd (direction doesn't matter) -->
<path fill-rule="evenodd" d="
  M 12 2 A 10 10 0 1 1 12 22 A 10 10 0 1 1 12 2 Z
  M 12 7 A 5 5 0 1 1 12 17 A 5 5 0 1 1 12 7 Z
" fill="black" />
```

With `nonzero` (default), the inner circle must wind in the opposite direction to create the hole.

## Logo Design Process

When creating logos (not icons), follow this process:

0. **Clarify direction when the brief is vague.** If the user's request lacks specifics about style, mood, or visual direction (e.g., "make me a logo for X"), use AskUserQuestion to present design direction options before starting. Don't ask open-ended questions that put the burden on the user. Instead, offer curated choices like a designer presenting mood boards. Tailor the options to the user's domain. Example:

    ```
    AskUserQuestion({
      questions: [
        {
          question: "What visual personality fits your brand?",
          header: "Mood",
          multiSelect: false,
          options: [
            { label: "Bold & geometric", description: "Clean shapes, strong lines, confident. Think Stripe, Figma." },
            { label: "Organic & crafted", description: "Hand-drawn feel, natural curves, warmth. Think Mailchimp, Basecamp." },
            { label: "Minimal & typographic", description: "Wordmark-driven, restrained, sophisticated. Think Glossier, Everlane." },
            { label: "Playful & dynamic", description: "Energetic, colorful, movement. Think Slack, Discord." }
          ]
        },
        {
          question: "What should the logo emphasize?",
          header: "Focus",
          multiSelect: false,
          options: [
            { label: "What we do", description: "Visual metaphor tied to your product's core function" },
            { label: "How it feels", description: "Abstract mark that conveys emotion or energy" },
            { label: "Who we are", description: "Letterform or wordmark built from the brand name" }
          ]
        }
      ]
    })
    ```

    Adapt the mood options to the user's domain. A law firm gets different moods than a gaming startup. Skip this step if the user already provided clear direction (e.g., "minimalist geometric logo in blue").

1. **Explore multiple metaphors, not multiple layouts of one metaphor.** Conceptual diversity matters more than layout variations. Follow the full ideation process in [references/logo-techniques.md](references/logo-techniques.md), which covers domain-specific brainstorming, category diversity requirements, and cliche avoidance.
2. **Guarantee structural variety.** Every logo set must span multiple *categories* of approach, not just multiple metaphors within the same style. Include at least one from each column when presenting 5+ options: a typographic/wordmark approach, a symbolic icon, an abstract geometric mark, and a letterform-meets-metaphor hybrid. See the category diversity table in [references/logo-techniques.md](references/logo-techniques.md).
3. **Set up the preview page.** Copy this skill's `assets/preview.html` to the project directory using `cp` with the absolute path from where this skill was loaded (do not read or modify the file). Write `variants.js` with variant data per the format in [references/editing-workflow.md](references/editing-workflow.md). Open with `open preview.html` (macOS) or `xdg-open preview.html` (Linux). On subsequent iterations, only edit `variants.js` and SVG files. The preview auto-reloads both every 3 seconds.
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
