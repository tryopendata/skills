# Responsive Charts

Charts are responsive by default (`responsive: true`). The engine uses a ResizeObserver to detect container size changes and recompiles the full layout at the new dimensions. No CSS media queries are involved - everything is computed at compile time based on container width and height.

## Width Breakpoints

Three breakpoints based on container width:

| Breakpoint | Width | Typical context |
| --- | --- | --- |
| `compact` | < 400px | Mobile, small embeds, sidebar widgets |
| `medium` | 400-700px | Tablet, half-width layouts |
| `full` | > 700px | Desktop, full-width |

## Height Classes

Three height classes based on container height:

| Height Class | Height | Typical context |
| --- | --- | --- |
| `cramped` | < 200px | Dashboard widgets, thumbnails, ultra-short embeds |
| `short` | 200-350px | Embedded panels, short containers |
| `normal` | > 350px | Standard containers |

## What the Engine Auto-Handles

The layout strategy combines width breakpoints and height classes to adjust these properties automatically:

### Width-Based Adaptation

| Property | Compact | Medium | Full |
| --- | --- | --- | --- |
| Data labels | Hidden | Important only | All shown |
| Legend position | Top | Top | Right |
| Annotations | Tooltip-only | Inline | Inline |
| Axis tick density | Minimal (3) | Reduced (5) | Full (8) |

### Height-Based Adaptation

| Property | Cramped (< 200px) | Short (200-350px) | Normal (> 350px) |
| --- | --- | --- | --- |
| Chrome | Hidden entirely | Compact (title only) | Full |
| Legend | Hidden | Max 15% of height | Unlimited |
| Data labels | Hidden | (uses width strategy) | (uses width strategy) |
| Annotations | Tooltip-only | (uses width strategy) | (uses width strategy) |

### Continuous Scaling

These properties scale continuously rather than stepping at breakpoints:

| Property | Behavior |
| --- | --- |
| **Padding** | Full at >= 500px (min dimension), scales down to 50% at <= 200px, floor of 4px |
| **Chrome font sizes** | Full at >= 500px wide, scales down to 72% at <= 250px wide (min 10px) |
| **Axis tick density** | Further reduced when chart area is very small, independent of breakpoint |

Font scaling only applies to default theme sizes. Explicit `style: { fontSize: 18 }` overrides on chrome elements are respected and not scaled.

### Auto-Rotation of Axis Labels

Column charts with band scales auto-rotate x-axis labels to -45 degrees when labels would overlap (average label width > 85% of bandwidth). This is automatic and only triggers when no explicit `tickAngle` is set. It handles the common case of column charts with many categories at narrow widths.

### Legend Overflow Protection

When there are many series:
- **Top-positioned legends** truncate after 2 rows (configurable via `legend.maxRows`), showing "+N more"
- **Right-positioned legends** cap at 40% of chart area height, truncating with "+N more"
- **Cramped height** hides the legend entirely

These are defaults. Explicit spec values (like `legend: { position: "top" }` or `labels: { density: "all" }`) override the responsive strategy.

## Breakpoint Overrides

Use the `overrides` field to provide different chrome, labels, legend, or annotations per breakpoint. Overrides are shallow-merged into the base spec at compile time when the container matches.

```js
{
  mark: "line",
  data: [...],
  encoding: { ... },
  chrome: {
    title: "Inflation hit a 40-year high in June 2022",
    subtitle: "CPI year-over-year % change vs federal funds rate, 2010-2026"
  },
  overrides: {
    compact: {
      chrome: {
        title: "Inflation hit a 40-year high",
        subtitle: "CPI YoY % change vs fed funds rate"
      },
      legend: { show: false },
      labels: { density: "none" }
    },
    medium: {
      chrome: {
        title: "Inflation hit a 40-year high in June 2022"
      }
    }
  }
}
```

**Override fields:** `chrome`, `labels`, `legend`, `annotations`. Each is shallow-merged (not deep-merged) with the base spec value. Annotations are replaced entirely, not merged.

**Priority:** If no override exists for the current breakpoint, the base spec is used unchanged.

## Legend Hiding

Set `legend: { show: false }` to suppress the legend entirely. Useful for bar charts where the y-axis already labels each category, making the legend redundant:

```js
{
  mark: "bar",
  encoding: {
    x: { field: "value", type: "quantitative" },
    y: { field: "platform", type: "nominal" },
    color: { field: "platform", type: "nominal" }
  },
  legend: { show: false }
}
```

You can also hide the legend only at compact widths via overrides:

```js
overrides: {
  compact: { legend: { show: false } }
}
```

## Chrome Compression

The engine automatically compresses chrome at small heights:

- **Normal** (> 350px): Full chrome - title, subtitle, source, byline, footer
- **Short** (200-350px): Compact mode - title only, subtitle and bottom chrome hidden
- **Cramped** (< 200px): Chrome hidden entirely to maximize chart area

Additionally, if the computed chart area is smaller than 60x40px even after height-based compression, the engine automatically strips chrome as a fallback (stepping from full to compact to hidden until the chart area is usable).

## Chrome Text Wrapping

Long chrome text (titles, subtitles, source) automatically wraps to multiple lines based on container width. The engine uses word-boundary wrapping with the same character-width heuristics used for layout computation.

- Text wraps at word boundaries when it exceeds the available width (container width minus padding)
- Each line becomes a separate `<tspan>` element in the SVG
- The layout system reserves vertical space for wrapped lines

While wrapping works, shorter text is still preferable. Wrapped titles lose the punchy single-line impact that makes charts scannable. Use overrides to provide shorter text at compact widths rather than relying on wrapping.

## What the Engine Does NOT Automatically Do

- Switch chart types (e.g., column to bar) at small sizes
- Reduce data point count for dense series

## Designing for Multiple Sizes

### Write short titles first

Titles should fit at compact width (~340px usable after padding). Aim for titles under ~25 characters. If a title needs more context at full width, put the detail in the subtitle:

```js
// Good - fits at all sizes
chrome: {
  title: "Inflation hit a 40-year high",
  subtitle: "CPI year-over-year % change vs fed funds rate, 2010-2026"
}

// Bad - truncates at compact
chrome: {
  title: "Inflation hit a 40-year high in June 2022"
}
```

### Choose chart types that work narrow

Horizontal bar charts handle narrow containers better than column charts when you have many categories, because category labels sit on the y-axis where they have room to breathe:

```js
// Good for narrow containers - 7 categories read fine
{ mark: "bar", encoding: { x: { field: "value" }, y: { field: "category" } } }

// Works now with auto-rotation, but bar is still better for 7+ categories
{ mark: "bar", encoding: { x: { field: "category" }, y: { field: "value" } } }
```

Column charts work fine with few categories (2-5) or when x-axis labels are short. With many categories, labels auto-rotate to -45 degrees to avoid overlap.

### Shorten axis labels

Long category labels on the x-axis of column charts auto-rotate at narrow widths. Use abbreviations when possible for cleaner results:

```js
// Good
{ quarter: "Q1 '24" }

// Acceptable - auto-rotates at narrow widths
{ quarter: "Q1 2024" }
```

For temporal x-axes, the engine reduces tick count automatically via `axisLabelDensity`. You rarely need to intervene.

### Mind the legend

The legend auto-positions to "top" at compact/medium and "right" at full. For bar charts where the y-axis already labels each category, hide the legend with `legend: { show: false }` to reclaim vertical space. You can also hide it only at compact widths via `overrides: { compact: { legend: { show: false } } }`.

When color encoding is needed for emphasis (e.g., highlight one bar in red, rest gray), keep series names short to reduce legend clutter.

With 8+ series, the legend truncates to 2 rows at top position and shows "+N more". At cramped heights, the legend is hidden entirely.

### Reduce annotation density

At compact, annotations switch to tooltip-only automatically. But explicitly placed annotations (`type: "text"`) may still overlap if positioned for full-width layouts. For charts that will render at compact:

- Use fewer annotations
- Prefer `type: "refline"` (adapts better) over `type: "text"` (fixed position)
- Keep annotation label text short

### Test at 375px

The most common mobile viewport is 375px (iPhone). Before publishing, verify your chart renders cleanly at this width. The three-second test applies double at mobile: if the title clips or labels overlap, the chart fails.

### Test at short heights

If your chart will appear in a dashboard widget or constrained panel, also test at 200-300px height. The engine will automatically compress chrome and adapt, but verify the result looks right.

## Spec Properties That Affect Responsiveness

| Property | Effect |
| --- | --- |
| `responsive: true` (default) | Enables ResizeObserver recompilation |
| `responsive: false` | Renders once at initial container size |
| `legend: { show: false }` | Hides legend entirely (saves space) |
| `legend: { position: "top" }` | Overrides responsive legend placement |
| `labels: { density: "none" }` | Forces labels off regardless of breakpoint |
| `labels: { density: "all" }` | Forces labels on regardless of breakpoint (may overlap at compact) |
| `encoding.x.axis.tickCount` | Overrides responsive tick reduction |
| `encoding.x.axis.tickAngle` | Overrides auto-rotation (set explicitly to prevent or force rotation) |
| `encoding.x.scale.nice: false` | Prevents padding at axis ends (saves space) |
| `overrides: { compact: { ... } }` | Breakpoint-conditional spec values |

## Known Limitations

1. **Overrides are shallow-merged** - You can override chrome, labels, legend, and annotations per breakpoint, but not encoding or data. Chart type and data are fixed across breakpoints.
2. **Text wrapping is heuristic** - Word wrapping uses estimated character widths (~0.55x font size). Actual rendered widths may differ slightly from estimates.
3. **Height overrides are not per-height-class** - The `overrides` field keys on width breakpoints (compact/medium/full), not height classes. Height adaptation is automatic only.
4. **Auto-rotation is band scales only** - Continuous and temporal scales use tick thinning, not rotation.
