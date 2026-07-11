# Theme Customization

Override default visual styling for any visualization type. All fields are optional and deep-merged onto the default theme. For the full `ThemeConfig` shape (every field, every default), load `ThemeConfig` and `ThemeColors` from `index.d.ts`. This file covers behavioral notes and editorial guidance.

## Shorthand: flat color array

`theme.colors` can be a flat `string[]` instead of the nested `{ categorical, ... }` object. It maps to `colors.categorical`:

```json
{ "theme": { "colors": ["#0d9488", "#94a3b8", "#94a3b8"] } }
```

is equivalent to:

```json
{ "theme": { "colors": { "categorical": ["#0d9488", "#94a3b8", "#94a3b8"] } } }
```

The flat form is more convenient for highlight+gray patterns where you just need to set ordered colors matching your data array.

## Default Categorical Palette

The default `colors.categorical` palette is an **OKLCH cyan-led** sequence designed to read as vibrant on both light and dark backgrounds. It's **mode-agnostic**: the same hex values are used in light and dark mode. The dark-mode adapter explicitly preserves the categorical palette across modes (it does not desaturate cyan into teal, the way a naive contrast-equivalence adapter would).

If you override `theme.colors.categorical`, your custom palette is also passed through unchanged across modes -- pick colors that work in both. For line strokes specifically, the engine applies a small per-color light-mode darkening (`adaptForLightLineStroke`) to saturated colors so they meet contrast on white backgrounds; achromatic (low-saturation) colors and already-dark colors pass through unchanged.

## Series Strategy

`theme.seriesStrategy?: 'palette' | 'accent-neutral'` (default `'palette'`) controls categorical color assignment by series count:

- `'palette'` -- full categorical palette, always (the default).
- `'accent-neutral'` -- 1 series gets the accent (first palette color) only; 2-4 series get accent for the first plus neutral grays for the rest; 5+ fall back to the full palette. The grays are surface-aware: darkest-first on light backgrounds, brightest-first on dark.

This automates the highlight+gray convention without hand-building a color array, but it always accents the *first* series -- if the protagonist isn't first in the data, sort it first or use the manual array (see color-strategy.md).

## Light/Dark Token Pairs

Every color field in `ThemeConfig` (including chrome element `color`) accepts a `TokenValue`: a plain string, or `{ "light": "#fff1e5", "dark": "#1a1311" }`. With a pair, dark mode uses your explicit dark value instead of deriving one algorithmically. Plain strings behave as before.

## Presets

Three named `ThemeConfig` presets are exported from `@opendata-ai/openchart-core`: `editorial` (the default look), `essay` (serif titles, warm background, generous spacing), `wire` (monospace, dense, tight chrome). Pass one as the `theme` prop/option, optionally spreading your own overrides on top.

## Dark Mode

`darkMode?: 'auto' | 'force' | 'off'` (default `'off'`):

| Value | Behavior |
| --- | --- |
| `"off"` | Always light mode |
| `"auto"` | Respect `prefers-color-scheme` system setting |
| `"force"` | Always dark mode |

**Class-based dark-mode apps:** `'auto'` only checks `prefers-color-scheme`. If your app toggles dark mode by toggling a CSS class (Astro, Next.js, Tailwind), observe the DOM class change and map it to `'force'`/`'off'` yourself — `'auto'` will not pick that up.

**What dark mode adapts:** background, text/secondary text, gridlines, axis lines, chrome text colors, and (where contrast equivalence helps) chrome accent colors. **What it does not adapt:** the categorical palette — those colors render identically in both modes by design.

## Theme Provider (React)

Wrap multiple visualizations to share a theme:

```tsx
import { VizThemeProvider } from '@opendata-ai/openchart-react';

<VizThemeProvider theme={myTheme} darkMode="auto">
  <Chart spec={spec1} />
  <Chart spec={spec2} />
  <DataTable spec={tableSpec} />
</VizThemeProvider>
```

## Example: Custom Theme

```json
{
  "mark": "bar",
  "data": [{ "cat": "A", "val": 10 }, { "cat": "B", "val": 20 }],
  "encoding": {
    "x": { "field": "val", "type": "quantitative" },
    "y": { "field": "cat", "type": "nominal" }
  },
  "theme": {
    "colors": {
      "categorical": ["#264653", "#2a9d8f", "#e9c46a", "#f4a261", "#e76f51"],
      "background": "#fafafa",
      "text": "#264653",
      "gridline": "#e0e0e0"
    },
    "fonts": {
      "family": "IBM Plex Sans, sans-serif"
    },
    "spacing": {
      "padding": 16
    },
    "borderRadius": 8
  },
  "darkMode": "auto"
}
```
