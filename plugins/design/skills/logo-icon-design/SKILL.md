---
name: logo-icon-design
description: >
  Design and generate professional logos, icons, and brand marks as SVG.
  Use this skill whenever the user mentions: logo, icon, pictogram, brand mark,
  wordmark, lettermark, favicon, app icon, monogram, symbol, sigil, badge, seal,
  emblem, or asks to "create a visual identity", "make a logo", "design an icon",
  "generate a symbol for", "create a brand mark". Also trigger when user asks to
  redesign or improve an existing logo/icon. Works in Claude Chat (SVG artifact),
  Claude Desktop, and Claude Code (SVG + PNG file output).
---

# Logo & Icon Design Skill

Produces professional logos, icons, and brand marks as SVG.
Production quality — never generic AI aesthetics.

---

## Step 1 — Detect environment

Before generating anything, identify the rendering context:

| Signal | Environment | Action |
|---|---|---|
| No filesystem access | Claude.ai Chat | Inline SVG via artifact/widget |
| bash_tool available | Claude Desktop / Code | .svg file + .png export |

---

## Step 2 — Logo brief (REQUIRED before drawing)

Ask these questions if not specified, in ONE grouped response:

1. **Name / text** to include? (or pure symbol?)
2. **Type**: wordmark / lettermark / pictorial / combined / abstract / icon
3. **Sector / tone**: luxury, tech, artisan, corporate, playful?
4. **Colors**: existing constraints or full freedom?
5. **Primary use**: web, app, print, favicon?

If context is ambiguous → ask ONE question: "Which project?"

---

## Step 3 — Creative direction (validate before SVG)

Propose in 3-4 lines:
- **Concept**: the central idea behind the shape
- **Style**: geometric / organic / typographic / brutalist / minimalist
- **Palette**: max 2-3 colors with hex codes
- **What makes it memorable**: the distinctive detail

Wait for validation or adjustment before coding the SVG.

---

## Step 4 — SVG Generation

### SVG rules for logos/icons

- Viewport: square preferred (100x100 or 500x500)
- Viewbox: always defined, e.g. viewBox="0 0 100 100"
- Units: no absolute px — coordinates relative to viewBox
- Colors: CSS custom properties in defs + style
- Text: always converted to path OR embedded font-face
- Export: no external dependencies (self-contained)

### Base SVG template

```svg
<svg xmlns="http://www.w3.org/2000/svg"
     viewBox="0 0 100 100"
     width="500" height="500"
     role="img" aria-label="[logo name]">
  <defs>
    <style>
      :root {
        --primary: #0A0F1E;
        --accent:  #4A9EFF;
        --light:   #FFFFFF;
      }
    </style>
  </defs>
  <!-- graphic elements here -->
</svg>
```

### Quality rules

- **Shapes**: prefer path with Bézier curves for fluidity
- **Proportions**: 8px grid / multiples of 8 for consistency
- **Stroke widths**: consistent across all elements
- **Legibility**: test mentally at 16px (favicon) AND 500px (print)
- **Variations**: plan light AND dark background versions if requested

---

## Step 5 — Output by environment

### Claude.ai Chat — Inline SVG artifact

Display SVG directly as a visual widget.
Then offer: "Want a dark background / color variant / file export?"

### Claude Desktop / Claude Code — Files

Save to project directory.

PNG export:
```bash
pip install cairosvg --break-system-packages -q
python3 -c "import cairosvg; cairosvg.svg2png(url='logo.svg', write_to='logo.png', output_width=1000)"
```

Deliver: SVG file + PNG 1000px + preview in chat.

---

## Logo types — quick reference

See references/logo-types.md for detailed SVG patterns by type.

| Type | Use case |
|---|---|
| Wordmark | Stylized brand name |
| Lettermark | 1-3 initials |
| Pictorial | Standalone recognizable icon |
| Combined | Icon + text |
| Abstract | Non-figurative shape |
| UI Icon | Interface, app, favicon (24x24/48x48) |

---

## Anti-patterns to always avoid

- Generic purple gradient on white circle
- Lightbulb for "idea", gear for "settings", rocket for "startup"
- Montserrat Bold all-caps + blue geometric icon
- Drop shadow on logo
- SVG text not converted to path

---

## Systematic iteration

After each delivery, always offer:
- Color variants (inverted, monochrome, single color)
- Proportion variants (horizontal / square / vertical)
- Optimized 32x32 favicon version
- Simple CSS animation if relevant
