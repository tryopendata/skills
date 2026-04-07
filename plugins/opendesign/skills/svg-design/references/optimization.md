# SVG Optimization

## Consolidate Styles to Root Element

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

Install via npm: `npm install -g svgo`

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

## Sprites

**Pros:** Single HTTP request for all icons, browser caches the file, consistent styling.

**Cons:** Downloads all icons even if page uses one. No tree-shaking. External sprite files don't work with `<use>` in Safari for cross-origin requests.

## When to Use `<g>` Groups

Groups are free (no rendering cost) but add DOM complexity. Use them when:

- Applying a shared `transform` to multiple elements
- Applying a shared `opacity`, `clip-path`, `mask`, or `filter`
- Logically grouping elements for readability
- Adding event handlers to a collection of shapes

Don't use them just for organization in distributed icons (Lucide prohibits them entirely).
