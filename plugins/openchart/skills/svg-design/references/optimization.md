# SVG Optimization

## Manual Optimization Checklist

Before using any tooling, apply these by hand:

### 1. Remove editor metadata

Strip everything from design tools:

```xml
<!-- Remove all of these -->
<sodipodi:namedview .../>
<inkscape:whatever .../>
<metadata>...</metadata>
<!-- Remove Illustrator comments -->
<!-- Generator: Adobe Illustrator 27.0.0, SVG Export Plug-In . -->
<!-- Remove data-name attributes from Figma -->
<path data-name="Layer 1" .../>
```

Also remove: `xmlns:sodipodi`, `xmlns:inkscape`, `xmlns:dc`, `xmlns:cc`, `xmlns:rdf`, `xmlns:serif`, `xml:space="preserve"`, `version="1.1"`, `id` attributes (unless referenced by `url(#id)`).

### 2. Remove unnecessary attributes

```xml
<!-- Before -->
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
  version="1.1" xml:space="preserve" x="0px" y="0px"
  width="24px" height="24px" viewBox="0 0 24 24"
  enable-background="new 0 0 24 24">

<!-- After -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
```

**Safe to remove** (usually):
- `version="1.1"` (browsers don't need it)
- `xml:space="preserve"` (only needed for whitespace-sensitive text)
- `x="0"`, `y="0"` on root svg (defaults)
- `xmlns:xlink` (deprecated; use `href` instead of `xlink:href`)
- `enable-background` (legacy)
- `id` on elements not referenced elsewhere
- `class` if no CSS targets it

### 3. Minimize decimal precision

Icons work fine with 1-2 decimal places. Complex illustrations can use 2-3.

```xml
<!-- Before -->
<path d="M 12.000000 2.345678 L 19.876543 21.234567" />

<!-- After -->
<path d="M 12 2.35 L 19.88 21.23" />
```

### 4. Use shorthand path commands

```xml
<!-- Before (absolute) -->
<path d="M 10 10 L 10 20 L 20 20 L 20 10 Z" />

<!-- After (relative + shorthand) -->
<path d="M 10 10 v 10 h 10 v -10 Z" />
```

### 5. Remove default attribute values

```xml
<!-- Defaults you can remove -->
fill="black"              <!-- fill defaults to black -->
fill-opacity="1"          <!-- defaults to 1 -->
stroke-opacity="1"        <!-- defaults to 1 -->
stroke-dashoffset="0"     <!-- defaults to 0 -->
stroke-miterlimit="4"     <!-- defaults to 4 -->
opacity="1"               <!-- defaults to 1 -->
fill-rule="nonzero"       <!-- defaults to nonzero -->
clip-rule="nonzero"       <!-- defaults to nonzero -->
```

### 6. Consolidate styles to root element

```xml
<!-- Before: repeated on every element -->
<path stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" fill="none" d="..." />
<path stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" fill="none" d="..." />

<!-- After: set once on root -->
<svg stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" fill="none" ...>
  <path d="..." />
  <path d="..." />
</svg>
```

## SVGO (Automated Optimization)

SVGO is the standard tool. Install via npm: `npm install -g svgo`

### Basic usage

```bash
svgo input.svg -o output.svg
svgo input.svg                  # Overwrite in place
svgo -f ./icons/                # Process entire folder
```

### Recommended config for icons

```js
// svgo.config.mjs
export default {
  plugins: [
    {
      name: 'preset-default',
      params: {
        overrides: {
          removeViewBox: false,          // CRITICAL: never remove viewBox
          cleanupIds: false,             // Keep IDs if using defs/use
        },
      },
    },
    'removeXMLNS',                       // Only if inlining in HTML
    {
      name: 'convertPathData',
      params: {
        floatPrecision: 2,              // 2 decimals for icons
      },
    },
  ],
};
```

### Dangerous SVGO defaults to disable

| Plugin | Risk | Fix |
|--------|------|-----|
| `removeViewBox` | Breaks responsive scaling | Set `removeViewBox: false` |
| `cleanupIds` | Breaks gradient/mask/clipPath references | Disable if using defs |
| `removeHiddenElems` | Can remove elements that are revealed via CSS/JS | Disable for animated SVGs |
| `collapseGroups` | Removes groups that may carry important transforms | Review output |

## defs, use, and symbol

### `<defs>` (Definitions)

Container for reusable elements. Content inside `<defs>` is not rendered directly.

```xml
<defs>
  <linearGradient id="brand-gradient">
    <stop offset="0%" stop-color="#3B82F6" />
    <stop offset="100%" stop-color="#8B5CF6" />
  </linearGradient>
</defs>
<!-- Reference it -->
<circle fill="url(#brand-gradient)" cx="12" cy="12" r="10" />
```

**Put in defs:** Gradients, patterns, masks, clipPaths, filters, symbols.

### `<use>` (Instance)

Clones a defined element. The clone inherits styles but can override some.

```xml
<defs>
  <circle id="dot" r="3" />
</defs>
<use href="#dot" x="6" y="12" fill="red" />
<use href="#dot" x="12" y="12" fill="green" />
<use href="#dot" x="18" y="12" fill="blue" />
```

**Important:** `<use>` creates a shadow DOM clone. You cannot style individual children of the cloned element with CSS. You can only override attributes set on the `<use>` element itself.

### `<symbol>` (Reusable icon template)

Like `<defs>` + `<g>` but with its own `viewBox`. Ideal for icon sprites.

```xml
<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
  <symbol id="icon-home" viewBox="0 0 24 24">
    <path d="M 3 12 L 12 3 L 21 12 v 9 H 15 v -6 H 9 v 6 H 3 Z" />
  </symbol>
  <symbol id="icon-user" viewBox="0 0 24 24">
    <circle cx="12" cy="8" r="5" />
    <path d="M 3 21 C 3 16 7 13 12 13 S 21 16 21 21" />
  </symbol>
</svg>
```

**Using symbols from the sprite:**

```html
<!-- Same-page sprite -->
<svg width="24" height="24"><use href="#icon-home" /></svg>

<!-- External sprite file -->
<svg width="24" height="24"><use href="/icons.svg#icon-home" /></svg>
```

### Sprite file structure

```xml
<!-- icons.svg -->
<svg xmlns="http://www.w3.org/2000/svg">
  <defs>
    <symbol id="icon-home" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <path d="..." />
    </symbol>
    <symbol id="icon-settings" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
      <path d="..." />
    </symbol>
    <!-- more icons -->
  </defs>
</svg>
```

**Pros of sprites:** Single HTTP request for all icons, browser caches the file, consistent styling.
**Cons:** Downloads all icons even if page uses one. No tree-shaking. External sprite files don't work with `<use>` in Safari for cross-origin requests.

## When to use `<g>` groups

Groups are free (no rendering cost) but add DOM complexity. Use them when:

- Applying a shared `transform` to multiple elements
- Applying a shared `opacity`, `clip-path`, `mask`, or `filter`
- Logically grouping elements for readability
- Adding event handlers to a collection of shapes

Don't use them just for organization in distributed icons (Lucide prohibits them entirely).

```xml
<!-- Good: shared transform -->
<g transform="rotate(45 12 12)">
  <line x1="12" y1="5" x2="12" y2="19" />
  <line x1="5" y1="12" x2="19" y2="12" />
</g>

<!-- Bad: unnecessary wrapper -->
<g>
  <path d="..." />
</g>
```
