# Visual QA for Rendered Charts

Systematic process for scanning rendered chart screenshots to identify visual defects. Run this after taking any verification screenshot.

## The Process

**Do not glance at a screenshot and say "looks good."** Instead, scan each zone deliberately and check for the specific defect types listed below.

### Zone-by-Zone Scanning Order

Scan the screenshot in this order. For each zone, check the listed defect types before moving to the next zone.

```
┌─────────────────────────────────┐
│ 1. CHROME ZONE (title/subtitle) │
├─────────────────────────────────┤
│        │                  │     │
│ 2.Y-AX │  4. PLOT AREA    │ 5.  │
│        │  (data, labels,  │ LEG │
│        │   annotations)   │ END │
│        │                  │     │
├─────────────────────────────────┤
│ 3. X-AXIS ZONE (tick labels)   │
├─────────────────────────────────┤
│ 6. FOOTER (source, byline)     │
└─────────────────────────────────┘
```

### Zone 1: Chrome (Title / Subtitle)

| Defect | What to look for | Common fix |
| --- | --- | --- |
| **Truncation** | Title or subtitle cut off at the right edge | Shorten text; for compact variants create a separate spec with shorter chrome |
| **Overlap with plot** | Title/subtitle text overlapping chart marks, annotations, or axis content | Move annotation down, or shorten chrome text |
| **Annotation collision** | A text annotation positioned in the chrome zone, overlapping title or subtitle | Reposition annotation lower (increase `dy`, change `anchor`) |

### Zone 2: Y-Axis

| Defect | What to look for | Common fix |
| --- | --- | --- |
| **Too few ticks** | Only 0 and max visible (e.g., `0` and `35`), making it impossible to read intermediate values | Add `axis: { tickCount: 5 }` or `tickCount: 6`. Readers need intermediate reference points to identify data values from the chart. |
| **Missing units** | Ticks show bare numbers (`10`, `20`) when the data represents percentages or currency | Add format string: `format: ".0f%"` for percentages, `format: "$,.0f"` for currency. Also remove redundant units from axis title (e.g., change `"Rate (%)"` to `"Rate"`). |
| **Label truncation** | Category names cut off at the left edge | Abbreviate labels or widen container |
| **Overlap** | Y-axis labels overlapping each other vertically | Reduce label count, abbreviate, or increase chart height |
| **Collision with plot** | Y-axis labels overlapping data marks or annotations | Add left padding via annotation offset adjustments |

### Zone 3: X-Axis

| Defect | What to look for | Common fix |
| --- | --- | --- |
| **Tick density** | Labels touching or overlapping each other horizontally | Add `axis: { tickCount: N }` to reduce tick count |
| **Temporal crowding** | Month-level ticks showing "Apr 2023Jul 2023Oct 2023" with no space | Reduce `tickCount` to show only year boundaries |
| **Band scale crowding** | Every category showing when only a subset fits | Add explicit `tickCount` override (engine respects this for band scales) |
| **Label clipping** | First or last tick label cut off at chart edge | Reduce tick count or adjust domain padding |

**Rule of thumb:** If any two adjacent tick labels have less than ~8px of whitespace between them, the ticks are too dense.

### Zone 4: Plot Area

| Defect | What to look for | Common fix |
| --- | --- | --- |
| **Label-on-label overlap** | Data labels (values on bars/points) overlapping each other | Switch to `labels: { density: "auto" }` or `"none"` |
| **Annotation-on-data overlap** | Annotation text sitting on top of data marks or labels | Adjust `offset`, change `anchor`, or add `background` |
| **Annotation-on-annotation overlap** | Multiple annotations overlapping each other | Reposition one, or remove the less important one |
| **Connector direction** | Connector line crossing through the annotation text or pointing away from the data point | Change `anchor` to the side closest to the data point; for curved connectors verify the sweep direction |
| **Connector length** | Very long diagonal connector crossing through unrelated data | Move annotation closer to data point, or use `connector: false` with proximity-based placement |
| **Refline label collision** | Refline label overlapping data labels or marks | Adjust `labelOffset` to position in whitespace gap |
| **Endpoint label overlap** | Series endpoint labels overlapping each other when lines converge | Adjust label offset, reduce label count, or switch to legend |
| **Data point clipping** | Dots/marks at extreme y-values cut off by the chart clip-path | Expand `scale.domain` to add headroom (e.g., `[-1, 10]` instead of `[0, 9.1]`) |
| **Dead space** | Large empty area on the right/left of chart with no data | Check `scale.domain` for overly wide ranges; for line charts, check `labels.density` is `"none"` if endpoint labels are reserving space. Long series names in `endpoints` mode reserve massive right margins. Switch to `"none"` + `legend: { position: "top" }`, or abbreviate series names |
| **Y-domain headroom waste** | Chart y-axis extends well beyond the data maximum (e.g., domain `[0, 55]` for data peaking at 48) | Set the domain ceiling to ~5-10% above the highest data point. Leave just enough room for annotations above the peak, not 15%+ of empty chart |

### Zone 5: Legend

| Defect | What to look for | Common fix |
| --- | --- | --- |
| **Overlap with endpoint labels** | Right-positioned legend colliding with series endpoint labels | Use `position: "bottom-right"` or `"bottom"` |
| **Overlap with axis ticks** | Bottom-positioned legend colliding with x-axis tick labels | Use `position: "bottom-right"` or `"right"` |
| **Overlap with data** | Legend box sitting on top of data marks | Change `position` to a clearer area |
| **Redundancy** | Legend present when direct labels already identify all series | Consider if legend can be moved to a less prominent position |

### Zone 6: Footer / Edges

| Defect | What to look for | Common fix |
| --- | --- | --- |
| **Source clipping** | Source or byline text cut off | Shorten text for compact variants |
| **Edge clipping** | Any content (marks, labels, annotations) cut off at container edges | Increase container size or adjust content positioning |
| **Right-edge overflow** | Endpoint labels or last data point extending past the plot boundary | Reduce rightmost annotation offset |

## Compact Variant Checks

Compact variants (container width < 400px or height < 350px) have unique failure modes. When verifying a compact story, pay extra attention to:

1. **Title overflow** - Titles that fit at 700px often clip at 320px. Create a separate `compactSpec` with shorter chrome text.
2. **Tick density** - Fewer ticks fit. Always add explicit `tickCount` overrides for compact specs.
3. **Label mode** - Dense data labels that fit at full size won't fit compact. Use `labels: { density: "none" }`.
4. **Legend size** - Legends take proportionally more space. Consider `position: "bottom-right"` or removing if direct labels exist.
5. **Annotation fit** - Annotations that fit at full size may overlap at compact. Remove or simplify annotations for compact variants.

## Data Accuracy Checks

These aren't visual defects but are detectable from the rendered output:

| Check | What to look for | Common fix |
| --- | --- | --- |
| **Stacked totals** | Stacked bar/column heights should be consistent if data represents percentages | Normalize data to sum to exactly 100% per group |
| **Sort order** | Bar charts should be sorted by value unless a natural order exists | Verify data array is sorted |
| **Scale honesty** | Bar/column charts should start at zero | Check `scale: { zero: true }` |

## Severity Classification

When reporting defects, classify by severity:

- **P0 - Misleading**: Chart communicates wrong information (wrong data, misleading axis, incorrect labels)
- **P1 - Unreadable**: Text cannot be read (overlap, truncation, clipping of essential content)
- **P2 - Ugly but readable**: Layout is awkward but content is legible (dense ticks with slight overlap, legend in suboptimal position)
- **P3 - Polish**: Minor spacing or alignment issues that don't affect readability
