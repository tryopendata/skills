# Design Review

How to evaluate whether a visualization meets editorial standards. Run this checklist after generating a spec and before calling it done.

## The 5-Second Test

Cover the title. Look at the chart for 5 seconds. Can you state the takeaway?

- **Yes**: the visual hierarchy is working. Data tells the story on its own.
- **No**: something is wrong. Too many series, unclear encoding, or weak visual hierarchy.

Now uncover the title. Does it match what you saw? If the title says something different from what the chart shows, either the title or the chart needs to change.

## The One Takeaway Test

If the reader looks at this chart for 5 seconds and walks away, what's the one thing they remember?

- If you can't answer it, the chart lacks focus
- If the answer differs from the title, rewrite the title
- If it takes more than 5 seconds to find the takeaway, simplify

## Design Review Checklist

| # | Check | Pass criteria |
| --- | --- | --- |
| 1 | Story clarity | Title states a specific, falsifiable insight |
| 2 | Visual hierarchy | Title dominates, then data, then supporting context |
| 3 | Color restraint | 5 or fewer distinct colors; highlight+gray strategy preferred |
| 4 | Data-ink ratio | No decorative elements, minimal gridlines, no 3D effects |
| 5 | Label readability | No overlapping labels, no text below 10px |
| 6 | Annotation economy | 0-3 annotations, each one earns its place |
| 7 | Axis honesty | Zero-baseline for bars, no misleading truncation without callout |
| 8 | Encoding types correct | quantitative for numbers, temporal for dates, nominal for categories |
| 9 | Field names match data | Every encoding field exists in the data objects |
| 10 | Category count reasonable | Pie/donut uses 2-6 categories max |
| 11 | Axis formatted | Currency, percentages, and large numbers have d3-format strings |
| 12 | Source attribution | `chrome.source` is populated with organization name |
| 13 | Responsiveness | Chart remains readable at 320px width |
| 14 | Accessibility | Colors distinguishable in grayscale, text labels present |

## Design Anti-Patterns

| Pattern | Problem | Fix |
| --- | --- | --- |
| Title describes chart type ("Bar chart of sales") | Reader learns nothing from the title | Title states the insight ("Q4 sales exceeded targets by 18%") |
| Rainbow palette | No perceptual ordering, visually chaotic | Use sequential ramp or 2-3 categorical colors |
| Dual y-axes | Creates arbitrary visual correlations | Side-by-side charts or indexed to common baseline |
| 3D charts | Distorts area and angle perception | Always use flat 2D |
| Truncated y-axis without callout | Exaggerates small differences | Start bars at zero; annotate axis break for lines |
| Legend with 8+ entries | Forces constant eye ping-pong | Direct label, filter to top 5, or use small multiples |
| Annotation essay (5+ annotations) | Text competes with data for attention | 1-2 word callouts; move context to subtitle |
| More than 5 colors | Hues become indistinguishable | Group small categories into "Other" or use highlight+gray |
| Pie for comparison across groups | Angle is a low-accuracy encoding for comparison | Use bar chart instead |
| Missing source attribution | No credibility, no provenance | Always include `chrome.source` |
| Decorative gridlines | Visual noise that doesn't aid reading | y-axis gridlines only, light gray, remove x-axis grid |

## Common Fixes

| Failing check | Action |
| --- | --- |
| Title is a chart description | Rewrite: state the finding, not the chart type |
| Too many colors | Switch to highlight+gray or group categories |
| Labels overlapping | Reduce label density to `"auto"` or `"endpoints"`, abbreviate text |
| Axis not formatted | Add `axis: { format: "$,.0f" }` or `".1f%"` (literal suffix) or `".1%"` (d3 native, data in 0-1) |
| Labels missing units | Add `labels: { format: ".1f%" }` to append `%`, or `"$,.0f"` for currency |
| Missing source | Add `chrome: { source: "Source: Organization Name" }` |
| Too many annotations | Keep the 1-2 most important, move rest to subtitle or remove |
| Pie with 7+ slices | Switch to horizontal bar chart |
| Chart feels cluttered | Remove background fill, reduce gridlines, increase whitespace |

## Rendered Output Verification

The checklist above covers the spec. After rendering, also verify the actual screenshot. Load [visual-qa.md](visual-qa.md) for the full defect catalog. Quick version:

| Zone | Check | Fail example |
| --- | --- | --- |
| Chrome | Title/subtitle fully visible, not overlapping plot | Title truncated at container edge |
| X-axis | Tick labels spaced with visible gaps between them | "Apr 2023Jul 2023Oct 2023" touching |
| Y-axis | Category labels fully visible, not overlapping | Long names clipped at left edge |
| Plot area | No text-on-text overlap (labels, annotations) | Annotation sitting on top of subtitle |
| Legend | Not colliding with labels, axis ticks, or data | Legend overlapping endpoint labels |
| Edges | No content clipped at container boundaries | Last data label cut off at right edge |
| Connectors | Pointing from label toward data point, not crossing text | Connector line crossing through its own annotation text |

**For compact variants** (< 400px wide): verify separately. Titles clip, ticks crowd, labels collide. Create a separate `compactSpec` with shorter chrome, explicit `tickCount`, and `labels: { density: "none" }`.

When a spec has 2+ text annotations on scatter/bubble charts, use the `playwright-cli` skill to screenshot and scan for annotation placement issues before finalizing.

## Ship It

All 14 checks pass: it's publication-ready.

1-2 minor items remaining (e.g., axis format could be cleaner, subtitle could be tighter): note them and move on. Perfect is the enemy of shipped.

Major items failing (wrong chart type, misleading axis, no title insight): fix before shipping.
