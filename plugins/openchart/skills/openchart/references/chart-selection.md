# Story-Driven Chart Selection

How to choose the right chart for the story you're telling, grounded in perceptual science.

## Perceptual Accuracy Ranking

Cleveland & McGill (1984) measured how accurately humans read different visual encodings. Use higher-accuracy encodings for the most important data comparisons.

| Rank | Encoding | Accuracy | Example charts |
| --- | --- | --- | --- |
| 1 | Position on common scale | Highest | Bar, dot plot, line |
| 2 | Position on non-aligned scales | High | Small multiples |
| 3 | Length | High | Bar (length comparison) |
| 4 | Direction / Angle | Medium | Pie, donut |
| 5 | Area | Low | Bubble, treemap |
| 6 | Color saturation | Lowest | Heatmap |

Position-based encodings (bars, lines, scatter) beat area and angle-based ones (pie, bubble) for accurate reading. Choose the encoding that matches the precision your story needs.

## Story Type to Chart Type

| Story you're telling | Best chart | Why it works |
| --- | --- | --- |
| Change over time | Line | Position on common axis reveals trend |
| Ranking | Bar (horizontal) | Position + length, category labels stay readable |
| Part-to-whole | Donut (2-5 categories) | Angle is adequate for small category counts |
| Correlation | Scatter | Two position channels show relationship directly |
| Distribution / spread | Dot (strip plot) | Individual values visible, shows density |
| Composition over time | Stacked area | Volume + trend together in one view |
| Before / after comparison | Grouped column | Side-by-side position makes difference obvious |

## Data-Ink Ratio

Tufte's principle: maximize the share of ink devoted to data. Every non-data element (decorative gridlines, background fills, border boxes) should earn its place or get removed.

That said, don't go full minimalist. Research shows some embellishment aids memorability. The goal is intentionality, not austerity. Remove chartjunk (3D effects, gradient fills, decorative icons). Keep elements that support comprehension (subtle gridlines, axis labels, direct data labels).

## Lie Factor

The visual effect size should match the data effect size. A 50% increase in data should look like roughly a 50% increase on the chart. Common violations:

- Truncated y-axis on bar charts (exaggerates small differences)
- Area-based encodings where radius maps to value (area grows quadratically)
- Perspective/3D effects that distort proportions

For bars: start at zero. For lines: truncation is acceptable because position, not length, encodes the value.

## Small Multiples vs. Overloading

When a single chart has too many series, split into a grid of small charts with shared axes.

| Situation | Approach |
| --- | --- |
| 2-5 series | Single chart with color encoding |
| 6+ series on same scale | Small multiples (one chart per series, shared axes) |
| Series with different y-scales | Always separate charts or index to common baseline |
| Comparing a few series from a large set | Highlight + gray (color the 2-3 that matter, gray the rest) |

## Dual-Axis Alternatives

Dual y-axes create arbitrary visual correlations. The chart author controls where lines appear to cross by choosing axis ranges. Readers assume visual proximity means relationship.

| Instead of dual axes | Use | When |
| --- | --- | --- |
| Side-by-side charts | Two charts, own scales | Different units, different magnitudes |
| Indexed chart | Percentage change from baseline | Same direction, different magnitudes |
| Connected scatterplot | One variable per axis | Exploring actual correlation |
| Annotation | Primary series + text context | Secondary data is supporting context |

## When to Break the Rules

- **Audience familiarity**: if the audience expects a specific chart type (e.g., financial candlestick), use it even if another type is perceptually more accurate
- **Publication consistency**: match the chart style of the surrounding content
- **Simplicity over optimality**: a slightly less-optimal chart that's instantly understood beats a perceptually perfect chart that confuses readers
