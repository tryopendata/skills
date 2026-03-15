# Theme Customization

Override default visual styling for any visualization type. All fields are optional and deep-merged onto the default theme.

## ThemeConfig

```typescript
theme?: {
  colors?: {
    categorical?: string[],           // 10-color palette for nominal data
    sequential?: Record<string, string[]>,  // named palettes: "blue", "green", "orange", "purple"
    diverging?: Record<string, string[]>,   // named palettes: "redBlue", "brownTeal"
    background?: string,              // canvas/container background
    text?: string,                    // default text color
    gridline?: string,                // gridline color
    axis?: string,                    // axis line and tick color
  },
  fonts?: {
    family?: string,                  // primary font (default: "Inter, sans-serif")
    mono?: string,                    // monospace font for tabular numbers
  },
  spacing?: {
    padding?: number,                 // chart container padding in px (default: 12)
    chromeGap?: number,               // gap between chrome elements in px (default: 4)
  },
  borderRadius?: number,             // container and tooltip border radius (default: 4)
}
```

**Shorthand:** `theme.colors` can also be a flat `string[]` instead of the nested object. It maps to `colors.categorical`:

```json
{ "theme": { "colors": ["#0d9488", "#94a3b8", "#94a3b8"] } }
```

is equivalent to:

```json
{ "theme": { "colors": { "categorical": ["#0d9488", "#94a3b8", "#94a3b8"] } } }
```

The flat form is more convenient for highlight+gray patterns where you just need to set ordered colors matching your data array.

## Dark Mode

```typescript
darkMode?: "auto" | "force" | "off"
```

| Value | Behavior |
| --- | --- |
| `"off"` | Always light mode (default) |
| `"auto"` | Respect `prefers-color-scheme` system setting |
| `"force"` | Always dark mode |

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
  "type": "bar",
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
