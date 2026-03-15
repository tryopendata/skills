# Color Strategy

Color is a narrative tool, not decoration. The right color strategy directs attention to the story.

## Three Strategies

| Strategy | Use when | How |
| --- | --- | --- |
| **Highlight + gray** | One series or data point matters most | Key element in brand/accent color, everything else gray. This is the editorial default. |
| **Sequential** | Magnitude or intensity matters | Single hue from light (low value, near background) to saturated (high value). For heatmaps, choropleths. |
| **Categorical** | Must distinguish 2-5 groups equally | Different hues with equal visual weight. No hue implies "more" or "less." |

## Highlight + Gray is the Default

Most editorial charts use this approach because it forces a story. When one element is colored and the rest are gray, the reader's eye goes directly to the signal. If everything is colored, nothing stands out.

Use categorical palettes only when the story genuinely requires distinguishing multiple groups equally. If one group is the protagonist, highlight it and gray the rest.

**Implementation:** Set `theme.colors` to an array where the protagonist gets a saturated color and every other entry is `"#94a3b8"` (slate-400). The order matches the data array order.

```json
{
  "theme": {
    "colors": ["#1b7fa3", "#94a3b8", "#94a3b8", "#94a3b8", "#94a3b8"]
  }
}
```

The first data row gets `#1b7fa3` (teal-blue, the default palette primary), the rest get gray. Sort data so the protagonist is first (or last, depending on chart type and visual weight).

## Decision Table: Story to Color Strategy

When choosing colors, find the row that best describes your data story:

| Data story | Strategy | theme.colors |
| --- | --- | --- |
| One group stands out from the rest | Highlight + gray | `["#1b7fa3", "#94a3b8", "#94a3b8", "#94a3b8"]` |
| Comparing 2-3 groups equally | Categorical | `["#1b7fa3", "#c44e52", "#6a9f58"]` |
| Showing intensity or magnitude gradient | Sequential | Use `encoding.color` with `type: "quantitative"` |
| Above/below a meaningful threshold | Diverging | `["#c44e52", "#e8e8e8", "#1b7fa3"]` |
| Positive vs. negative change | Semantic red/green | `["#c44e52", "#6a9f58"]` |
| One protagonist + one antagonist | Two-color highlight | `["#1b7fa3", "#c44e52", "#94a3b8"]` |
| No particular group matters more | Default palette | Omit `theme.colors` entirely |

## How theme.colors Maps to Series

`theme.colors` (flat array shorthand) sets the categorical palette. The engine assigns colors by order of unique values encountered in the `color` encoding field's data:

- First unique value in data -> `colors[0]`
- Second unique value -> `colors[1]`
- Third -> `colors[2]`, etc.

To highlight a specific series, sort data so the protagonist appears first, then set `theme.colors` to `["#accent", "#94a3b8", "#94a3b8", ...]`.

**Caveat:** "first unique value" depends on chart type and data order. For bar charts, the first data row renders as the topmost bar. For line charts, the first unique value in the `color` field (based on row order) gets `colors[0]`. If data comes from an external source and row order isn't controlled, explicitly set enough entries in `theme.colors` to cover all series, placing the accent color at the correct index position.

## Double-Encoding: Color That Reinforces Position

When a quantitative axis already tells the story (e.g., poverty rate on x-axis), **use color to reinforce the same variable**. Bucket the continuous dimension into 3-4 ordinal tiers and map them to a cool-to-warm gradient. This makes the pattern legible at a glance even before the reader processes axis values.

This is not the same as categorical color (where hues are arbitrary). Here, the color progression has semantic meaning: blue = low, red = high. The reader sees the gradient and understands the narrative without reading a single number.

When to use: scatter plots, bubble charts, or any chart where a quantitative dimension is the primary story. Don't leave dots monochrome when the data has a strong gradient to show.

**Implementation:** Bucket the continuous variable into ordinal tiers in your data before passing to the spec, then map those tiers to a cool-to-warm gradient:

```json
{
  "encoding": {
    "x": { "field": "poverty_rate", "type": "quantitative" },
    "y": { "field": "graduation_rate", "type": "quantitative" },
    "color": { "field": "poverty_tier", "type": "ordinal" }
  },
  "theme": {
    "colors": ["#1b7fa3", "#d47215", "#c44e52"]
  }
}
```

Where `poverty_tier` is a derived field with values like "Low", "Medium", "High" bucketed from the continuous `poverty_rate`.

## How Many Colors

| Count | Guidance |
| --- | --- |
| 1 (+ gray) | Ideal for most editorial charts. Forces focus. |
| 2-3 | Good for direct comparison between specific groups |
| 4-5 | Maximum for categorical. Beyond this, hues become hard to distinguish. |
| 6+ | Regroup into "Other", use highlight+gray, or switch to small multiples |

## Color Ramps

**Sequential**: light to dark within a single hue. Light values sit near the background (low data values), dark values carry visual weight (high data values). Good for: heatmaps, choropleths, single-variable intensity.

**Diverging**: two complementary hues meeting at a meaningful midpoint (often zero, or a target value). The midpoint should be semantically meaningful, not just the mathematical center. Good for: positive/negative change, above/below target, deviation from average.

## Accessibility

Color vision deficiency affects ~8% of men and ~0.5% of women (predominantly red-green).

| Rule | Why |
| --- | --- |
| Never encode meaning through hue alone | Pair color with labels, patterns, or position |
| Avoid red-green as the only differentiator | Use blue-orange or other colorblind-safe pairs |
| Test against deuteranopia simulation | Catches the most common deficiency |
| Ensure sufficient contrast ratios | WCAG AA minimum: 4.5:1 for text, 3:1 for large text/graphics |
| Provide text labels on data points | Labels work regardless of color perception |

## Semantic Conventions

These associations are culturally common in Western contexts but not universal:

| Color | Convention | Caveat |
| --- | --- | --- |
| Red | Loss, decline, danger, negative change | In Chinese and some Asian cultures, red signals prosperity |
| Green | Growth, positive change, success | Don't rely on red-green contrast alone |
| Blue | Neutral, primary, trustworthy | Safe default when no semantic meaning needed |
| Gray | Context, background, de-emphasized | The workhorse of editorial charts |

Always provide context beyond color. A red bar with a "-12%" label communicates decline through both channels.

## Dark Mode

When supporting dark mode:
- Lighten desaturated colors so they remain visible against dark backgrounds
- Maintain WCAG contrast ratios (recalculate against dark background)
- Sequential ramps may need inversion (dark-to-light instead of light-to-dark)
- Gray context elements need higher lightness to remain visible
- Test the full palette in both modes before shipping

## Ready-Made Palettes

Copy-paste-ready `theme.colors` arrays for common scenarios. All colors come from or complement the default categorical palette.

| Scenario | theme.colors |
| --- | --- |
| Highlight + gray (1 accent) | `["#1b7fa3", "#94a3b8", "#94a3b8", "#94a3b8", "#94a3b8"]` |
| Two-group comparison | `["#1b7fa3", "#c44e52"]` |
| Three-group comparison | `["#1b7fa3", "#c44e52", "#6a9f58"]` |
| Positive / negative (green = growth) | `["#6a9f58", "#c44e52"]` |
| Cool-to-warm gradient (3 tiers) | `["#1b7fa3", "#d47215", "#c44e52"]` |
| De-emphasis gray | `"#94a3b8"` (slate-400, use for context series) |
