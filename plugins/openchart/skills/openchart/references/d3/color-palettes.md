# Color Palettes

Color selection, accessibility, and D3 color schemes for data visualization.

## Contents

- Palette types
- Colorblind accessibility
- Semantic colors
- D3 color schemes
- Custom palette design
- Dark mode considerations

## Palette types

### Categorical (qualitative)

For distinct categories with no inherent order.

| Use | Don't Use |
|-----|-----------|
| Different series in line chart | Sequential data (low → high) |
| Bar groups | Ordered rankings |
| Pie/donut slices | Diverging data (neg → pos) |

**Max categories:** 8 distinct colors. More than that, group into "Other" or use small multiples.

```typescript
const categoricalPalette = [
  "#2563EB",  // Blue
  "#DC2626",  // Red
  "#059669",  // Green
  "#D97706",  // Amber
  "#7C3AED",  // Purple
  "#0891B2",  // Cyan
  "#BE185D",  // Pink
  "#4B5563",  // Gray
];
```

### Sequential

For continuous data from low to high.

| Use | Don't Use |
|-----|-----------|
| Heatmaps | Unordered categories |
| Choropleth maps | Diverging data |
| Density | Discrete groups |

```typescript
// Light to dark (single hue)
const sequentialBlue = [
  "#EFF6FF",  // Lightest
  "#BFDBFE",
  "#60A5FA",
  "#2563EB",
  "#1E40AF",  // Darkest
];

// Multi-hue (better perceptual uniformity)
const sequentialViridis = d3.schemeViridis;
```

### Diverging

For data with a meaningful midpoint (zero, average, baseline).

| Use | Don't Use |
|-----|-----------|
| Positive/negative change | Always-positive data |
| Above/below average | Unordered categories |
| Correlation matrices | Single-direction scales |

```typescript
// Red-Blue diverging (good for positive/negative)
const divergingRdBu = [
  "#B91C1C",  // Strong negative
  "#EF4444",
  "#FCA5A5",
  "#E5E7EB",  // Neutral
  "#93C5FD",
  "#3B82F6",
  "#1D4ED8",  // Strong positive
];
```

## Colorblind accessibility

~8% of men have red-green color blindness (deuteranopia/protanopia).

### Safe combinations

| Good | Avoid |
|------|-------|
| Blue + Orange | Red + Green |
| Blue + Yellow | Red + Brown |
| Purple + Yellow | Green + Brown |
| Blue + Red (with shape/pattern) | Adjacent hues only |

### Recommended palette (colorblind-safe)

```typescript
const colorblindSafe = [
  "#0077BB",  // Blue
  "#EE7733",  // Orange
  "#009988",  // Teal
  "#CC3311",  // Red
  "#33BBEE",  // Cyan
  "#EE3377",  // Magenta
  "#BBBBBB",  // Gray
];
```

### Testing

- [Coblis Color Blindness Simulator](https://www.color-blindness.com/coblis-color-blindness-simulator/)
- [Sim Daltonism](https://michelf.ca/projects/sim-daltonism/) (macOS)
- Chrome DevTools > Rendering > Emulate vision deficiencies

### Beyond color

Don't rely on color alone:
- Add patterns/textures to chart elements
- Use different shapes for data points
- Include direct labels

```typescript
// Line chart with different dash patterns
const dashPatterns = [
  "",           // Solid
  "4,4",        // Dashed
  "1,3",        // Dotted
  "8,4,2,4",    // Dash-dot
];

<path
  strokeDasharray={dashPatterns[seriesIndex]}
/>
```

## Semantic colors

Assign consistent meaning to colors.

```typescript
interface SemanticPalette {
  positive: string;   // Growth, success, profit
  negative: string;   // Decline, error, loss
  neutral: string;    // Unchanged, baseline
  highlight: string;  // Call attention
}

const semanticColors: SemanticPalette = {
  positive: "#059669",  // Green
  negative: "#DC2626",  // Red
  neutral: "#6B7280",   // Gray
  highlight: "#F59E0B", // Amber
};
```

### Automatic application

```typescript
function getValueColor(value: number, baseline = 0): string {
  if (value > baseline) return semanticColors.positive;
  if (value < baseline) return semanticColors.negative;
  return semanticColors.neutral;
}

// Usage
<rect fill={getValueColor(change)} />
```

## D3 color schemes

### Categorical

```typescript
import { schemeCategory10, schemeSet2, schemeTableau10 } from "d3-scale-chromatic";

// Most popular
const colors = schemeCategory10;      // 10 colors
const colors = schemeSet2;            // 8 colors, pastel
const colors = schemeTableau10;       // 10 colors, professional
```

### Sequential

```typescript
import {
  interpolateBlues,
  interpolateViridis,
  interpolatePlasma,
} from "d3-scale-chromatic";

// For scaleSequential
const colorScale = scaleSequential()
  .domain([0, maxValue])
  .interpolator(interpolateBlues);

// Usage
fill={colorScale(value)}
```

### Diverging

```typescript
import {
  interpolateRdBu,
  interpolatePiYG,
  interpolateBrBG,
} from "d3-scale-chromatic";

const colorScale = scaleSequential()
  .domain([-maxAbs, maxAbs])  // Symmetric around zero
  .interpolator(interpolateRdBu);
```

### Creating discrete scales from continuous

```typescript
import { quantize } from "d3-interpolate";
import { interpolateBlues } from "d3-scale-chromatic";

// Get 5 discrete colors from continuous scale
const discreteBlues = quantize(interpolateBlues, 5);
// ["#eff6ff", "#bfdbfe", "#60a5fa", "#2563eb", "#1e40af"]
```

## Custom palette design

### Starting from a brand color

```typescript
import { hsl, rgb } from "d3-color";

function generatePalette(baseColor: string, count: number): string[] {
  const base = hsl(baseColor);
  const palette: string[] = [];

  for (let i = 0; i < count; i++) {
    const hue = (base.h + (i * 360 / count)) % 360;
    palette.push(hsl(hue, base.s, base.l).formatHex());
  }

  return palette;
}
```

### Generating shades

```typescript
function generateShades(color: string, steps: number): string[] {
  const base = hsl(color);
  const shades: string[] = [];

  for (let i = 0; i < steps; i++) {
    const lightness = 0.95 - (i * 0.7 / (steps - 1));  // 95% to 25%
    shades.push(hsl(base.h, base.s, lightness).formatHex());
  }

  return shades;
}
```

## OpenData palette system

### 6 preset palettes

| Palette | Primary | Secondary | Positive | Negative | Use Case |
|---------|---------|-----------|----------|----------|----------|
| **Ocean** | #2563EB | #60A5FA | #10B981 | #EF4444 | General purpose, trust |
| **Sunset** | #F97316 | #FBBF24 | #22C55E | #DC2626 | Growth stories, energy |
| **Violet** | #8B5CF6 | #A78BFA | #14B8A6 | #F43F5E | Tech, analytical |
| **Slate** | #475569 | #94A3B8 | #059669 | #B91C1C | Serious, institutional |
| **Forest** | #166534 | #4ADE80 | #0D9488 | #DC2626 | Environmental, sustainability |
| **Midnight** | #1E3A5F | #3B82F6 | #34D399 | #FB7185 | Premium, editorial |

### Full palette definitions

```typescript
interface ChartPalette {
  id: string;
  name: string;
  primary: string;
  secondary: string;
  positive: string;
  negative: string;
  background: string;
  text: string;
  textMuted: string;
}

export const PALETTES: Record<string, ChartPalette> = {
  ocean: {
    id: "ocean",
    name: "Ocean",
    primary: "#2563EB",
    secondary: "#60A5FA",
    positive: "#10B981",
    negative: "#EF4444",
    background: "#FFFFFF",
    text: "#1A1A1A",
    textMuted: "#9CA3AF",
  },
  sunset: {
    id: "sunset",
    name: "Sunset",
    primary: "#F97316",
    secondary: "#FBBF24",
    positive: "#22C55E",
    negative: "#DC2626",
    background: "#FFFFFF",
    text: "#1A1A1A",
    textMuted: "#9CA3AF",
  },
  violet: {
    id: "violet",
    name: "Violet",
    primary: "#8B5CF6",
    secondary: "#A78BFA",
    positive: "#14B8A6",
    negative: "#F43F5E",
    background: "#FFFFFF",
    text: "#1A1A1A",
    textMuted: "#9CA3AF",
  },
  slate: {
    id: "slate",
    name: "Slate",
    primary: "#475569",
    secondary: "#94A3B8",
    positive: "#059669",
    negative: "#B91C1C",
    background: "#FFFFFF",
    text: "#1A1A1A",
    textMuted: "#9CA3AF",
  },
  forest: {
    id: "forest",
    name: "Forest",
    primary: "#166534",
    secondary: "#4ADE80",
    positive: "#0D9488",
    negative: "#DC2626",
    background: "#FFFFFF",
    text: "#1A1A1A",
    textMuted: "#9CA3AF",
  },
  midnight: {
    id: "midnight",
    name: "Midnight",
    primary: "#1E3A5F",
    secondary: "#3B82F6",
    positive: "#34D399",
    negative: "#FB7185",
    background: "#FFFFFF",
    text: "#1A1A1A",
    textMuted: "#9CA3AF",
  },
};
```

### Usage patterns

```typescript
// Get series color (primary first, then secondary)
function getSeriesColor(palette: ChartPalette, index: number): string {
  if (index === 0) return palette.primary;
  return palette.secondary;  // Only 2 colors in simplified palette
}

// Auto-select semantic colors based on data
function getValueColor(
  value: number,
  baseline: number,
  palette: ChartPalette
): string {
  if (value > baseline) return palette.positive;
  if (value < baseline) return palette.negative;
  return palette.textMuted;
}
```

### Palette selection guidelines

| Content Type | Recommended Palette |
|--------------|---------------------|
| Financial data | Ocean, Slate |
| Growth stories | Sunset, Forest |
| Tech/Analytics | Violet, Midnight |
| Environmental | Forest |
| Serious/institutional | Slate |
| Editorial/premium | Midnight |

### Contrast-aware watermark

```typescript
function getWatermarkVariant(backgroundColor: string): "light" | "dark" {
  const bgLuminance = luminance(backgroundColor);
  return bgLuminance > 0.5 ? "dark" : "light";
}
```

## Dark mode

### Adjustments needed

| Element | Light Mode | Dark Mode |
|---------|------------|-----------|
| Background | #FFFFFF | #1A1A1A |
| Text | #1A1A1A | #F5F5F5 |
| Muted text | #666666 | #9CA3AF |
| Borders | #E5E7EB | #374151 |
| Primary colors | Saturated | Slightly desaturated |

### Desaturating colors

```typescript
import { hsl } from "d3-color";

function desaturateForDark(color: string, amount = 0.2): string {
  const c = hsl(color);
  c.s = Math.max(0, c.s - amount);
  return c.formatHex();
}
```

### Theme-aware palette

```typescript
const lightPalette: ChartPalette = {
  background: "#FFFFFF",
  text: "#1A1A1A",
  textMuted: "#666666",
  primary: "#2563EB",
  // ...
};

const darkPalette: ChartPalette = {
  background: "#1A1A1A",
  text: "#F5F5F5",
  textMuted: "#9CA3AF",
  primary: "#3B82F6",  // Slightly lighter blue
  // ...
};

function usePalette(theme: "light" | "dark"): ChartPalette {
  return theme === "dark" ? darkPalette : lightPalette;
}
```

## Contrast checking

Ensure text is readable on colored backgrounds:

```typescript
import { rgb } from "d3-color";

function luminance(color: string): number {
  const c = rgb(color);
  const [r, g, b] = [c.r, c.g, c.b].map(v => {
    v /= 255;
    return v <= 0.03928 ? v / 12.92 : Math.pow((v + 0.055) / 1.055, 2.4);
  });
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}

function contrastRatio(color1: string, color2: string): number {
  const l1 = luminance(color1);
  const l2 = luminance(color2);
  const lighter = Math.max(l1, l2);
  const darker = Math.min(l1, l2);
  return (lighter + 0.05) / (darker + 0.05);
}

// WCAG requires 4.5:1 for normal text, 3:1 for large text
function getTextColor(background: string): string {
  return contrastRatio(background, "#FFFFFF") > 4.5 ? "#FFFFFF" : "#1A1A1A";
}
```
