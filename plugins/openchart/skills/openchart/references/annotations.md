# Annotations

Annotations are the editorial layer on top of data visualizations. They highlight insights, call out outliers, and provide context. Supported on charts and graphs.

The `Annotation` type is a discriminated union on `type` (`'text' | 'range' | 'refline'`). For the full per-variant shape (every field, every default), load `Annotation`, `TextAnnotation`, `RangeAnnotation`, `RefLineAnnotation`, and `AnnotationDot` from `index.d.ts`. This file covers the *behavior* the types don't describe — anchoring, collision resolution, what specific fields actually do at render time, and which combinations are gotchas.

> `'rule'` is accepted at runtime as a deprecated alias for `'refline'` (the engine rewrites it with a console warning), but the TypeScript type no longer includes it. Always use `'refline'` in new specs.

## Text Annotation — behavior worth knowing

**`x` / `y` are data coordinates.** They resolve to pixels through the same scales as data marks, so annotations stay stable across responsive resizes. Always set them to the actual data point you're annotating; use `offset` to push the label into clear space.

**`text` supports `\n`** for hard line breaks and **`**bold**` spans** for inline emphasis: `"Inflation peaked at **8.5%**"`. Emphasis belongs on the key phrase, not the whole block — that's the NYT/Datawrapper voice. Only matched pairs turn bold; an unmatched `**` renders literally. Bold spans work in `subtitle` too.

**Text is never center-aligned.** Lines align on the edge facing the data point: left-aligned (ragged right) by default, right-aligned when `anchor: 'left'` puts the block to the left of the point. That's about how the lines sit against each other, not where the block lands: a `top`/`bottom` anchored block still straddles its data point horizontally, so "above" means above. Don't hand-compute a negative `dx` to re-center it — the engine already does.

**`subtitle`** renders a muted second-tone line below `text`. Use for "(methodology note)" or a source citation that belongs with the callout. Multi-line `text` still produces a multi-line primary block; `subtitle` is a separate block underneath.

**The lede rule:** when a `subtitle` is present and you set no `fontWeight`, the primary `text` resolves to weight 700 and the subtitle stays 400. That's the "Feb. 25 / 2015 maximum" stack. Set `fontWeight` explicitly to opt out.

**`dot`** draws an open-ring marker on the data point itself. You get one **by default** whenever a connector is enabled and carries no arrowhead — straight leaders and drop-lines included. An arrowed connector gets no marker unless you ask for one (the arrowhead already marks the spot). `dot: false` gives a bare label; an explicit `dot` always wins. Default style: theme-background fill (the open ring), stroke matching the connector, radius 4, strokeWidth 1.5. Pass an `AnnotationDot` object to override any of those.

**`anchor`** is the direction the label sits relative to the data point: `'top' | 'bottom' | 'left' | 'right' | 'auto'`. `'auto'` checks if the data point is in the upper or lower half of the chart and places the label in the opposite half with a 28px offset — that's it. Not intelligent whitespace detection. For predictable results with 2+ annotations, always specify explicit `anchor` and `offset`.

**`connector`** is `boolean | 'straight' | 'curve' | 'drop-line'` (or `{ type, arrow? }` for explicit arrow control):

- `true` / `'straight'`: straight line from label to anchor point, no arrowhead. Gray hairline.
- `'curve'`: single quadratic arc with a small open-V arrowhead, drawn in the label's text ink. This is the emphasis voice — use it for the one callout that carries the story.
- `'drop-line'`: vertical line through the data point's x; label sits beside the line and auto-flips to the opposite side if it would overflow the chart area. **Caveat:** the auto-flip only checks against the chart edge — it does not avoid neighboring marks or other annotations. Place these in clear regions.
- `false`: no connector.

The connector leaves the text block (primary text plus subtitle, as one box) on the side facing the data point, with a 6px standoff, and stops short of the marker at the other end. **Short connectors are dropped:** a leader under 8px, or one whose data point sits inside the text block, is suppressed and only the marker renders. A plain `{ type: 'text' }` with no `offset` sits 28px off the point, so it draws a ring *and* a real leader with zero authoring — you only lose the line by pulling the label back onto its own marker.

**`halo`** is on by default — it paints a stroke halo behind the text using the theme's background color, knocking out chart lines that pass behind. Set `halo: false` when the annotation text is white or light-colored sitting on a colored background (the halo would make the text invisible). Use `background` instead for an explicit rect.

**`responsive`** defaults to `true`, which means the annotation **hides at compact breakpoints (< 400px)**. Set `responsive: false` on annotations that must always be visible.

**Series-anchored text annotations:** When a text annotation's `(x, y)` lands on a colored series and the user toggles that series off via the legend, the annotation is suppressed (it would drift onto a different scale). It comes back when the series is re-shown. Range and refline annotations pass through legend toggles unchanged because they anchor to constant axis values, not series.

**Tips:**
- Use `\n` in `text` for multi-line annotations and `**bold**` for the key phrase inside a sentence
- Text annotations get a paint-order stroke halo by default (using the theme's background color) that knocks out chart lines behind the text without needing an explicit background rect
- Set `halo: false` when the annotation text is white or light-colored and sits on a colored element -- the halo would make the text invisible. Use `background` instead for an explicit rect.
- Set `background` to `true` (theme surface, follows light/dark) or a color string for an explicit background rect instead of the default halo
- `connector` defaults to `true` (straight gray leader from label to point, with an open-ring marker on the point)
- `"curve"` connector draws a single arc in the label's ink with an open-V arrowhead — reserve it for the one callout that carries the story
- `anchor: "auto"` lets the engine pick a position (see details below)
- `responsive: false` keeps the annotation visible at compact breakpoints (< 400px). Default behavior hides all annotations at compact sizes.

## Placement on Dense or Bubble Charts

Scatter/bubble charts create unique annotation challenges because circles are large and clustered. Small offsets (10-20px) almost always land on top of nearby dots. Follow these rules:

**Use large offsets.** On bubble charts, `offset: { dy: -50 }` or more is normal. Think of the annotation as living in the empty quadrant of the chart, not next to its data point. The connector line handles the visual link back.

**Anchor at the data point, offset into empty space.** Always set `x`/`y` to the actual data coordinates so the connector points to the right dot. Then use `offset` to push the text into a clear zone. Don't move the `x`/`y` anchor to approximate the label position (this disconnects the connector from the data).

**Scan for empty quadrants.** Before placing annotations, identify the chart's empty zones. Scatter plots typically have open corners (upper-left for low-x/high-y, lower-right for high-x/low-y). Place labels there with connectors back to their data points.

**Budget for bubble radius.** A dot with 13,000 enrollment and a dot with 400 enrollment have very different radii. The label for the larger dot needs to clear a bigger circle. When in doubt, overshoot the offset and let the connector do the work.

| Mistake | Fix |
| --- | --- |
| Small offset lands on adjacent dots | Use 40-100px offsets into empty chart regions |
| Annotation near chart edge gets clipped | Pull label inward with negative dx or dy |
| Anchor set to approximate position, not data point | Set x/y to actual data coords; use offset for visual position |
| Label in lower-left corner colliding with axes | Move to upper-left or use larger dy to clear axis labels |

## How the Engine Handles Placement

Understanding what the engine does (and doesn't do) helps you write annotations that render correctly on the first try.

**What the engine does automatically:**
- Resolves annotation `x`/`y` data coordinates to pixel positions via the same d3 scales used for data points
- Nudges text annotations away from obstacle rects (legend bounds, band-scale mark bounds) so annotations don't land on the legend
- Resolves annotation-to-annotation collisions using a greedy algorithm: if two text annotations overlap, the second one is repositioned to the nearest non-colliding spot (below, above, left, or right)
- Rebuilds connectors after any nudge, clamp, or auto-placement so they still exit the text block on the side facing the point (the subtitle moves with its label, and the connector re-derives from the data point, not from its previous pulled-back end)
- Suppresses a connector that would be shorter than 8px or would have to cross its own label
- Hides annotations at compact breakpoints (< 400px width) unless the annotation has `responsive: false`
- Demotes crowded callouts to numbered footnotes at narrow widths (thinning). Labels that sit in the chart margin — above a peak, below a trough — are fine; only labels escaping the chart entirely get demoted.

**What `anchor: "auto"` actually does:** Checks if the data point is in the upper or lower half of the chart area. Upper half places the label below-right with a 28px offset; lower half places it above-right with a 28px offset. This is simple heuristic placement, not intelligent whitespace detection. For predictable results, always specify explicit `anchor` and `offset` values.

**Practical limits of auto-collision resolution:** The engine's greedy algorithm works best when annotations start in different areas of the chart. If all annotations cluster in the same region, the nudging can push labels into awkward positions. Give the engine a head start by placing annotations in distinct zones.

## Estimating Annotation Size

These rough estimates help you reason about whether annotations will fit. They match the engine's internal text measurement.

- **Text width:** `characters * fontSize * 0.55` (default fontSize is 13). A 20-character annotation is roughly `20 * 13 * 0.55 = 143px`. Bold text (fontWeight 700, including `**bold**` spans) adds ~8% width.
- **Text height:** `lines * fontSize * 1.3`. A single line at 13px = ~17px tall.
- **Multi-line text** (`\n`): the widest line determines bounding box width. With `anchor: 'top'`/`'bottom'` the block is centered on the data point; with `anchor: 'left'`/`'right'` it hangs off the facing edge.
- **`subtitle`:** adds another block underneath at 0.85x the primary font size, plus a 2px gap.

These are heuristic estimates. The engine can't know the precise rendered width without a real font metric context. When placement is critical (scatter/bubble with multiple annotations), use the `playwright-cli` skill to screenshot and verify.

## Practical Rules for Multiple Annotations

- **Prefer 0-2 text annotations per chart.** Use reflines with labels for additional callouts. Refline labels are simpler and less collision-prone.
- When using 2+ text annotations, aim for different areas of the chart. The engine auto-resolves collisions, but good initial separation produces cleaner results.
- **Never use `anchor: "auto"` with 2+ text annotations.** Specify explicit anchors and offsets for each one.
- For scatter/bubble charts, use 40-100px offsets to push labels into empty quadrants, with connectors back to data points.

## Range Annotation — behavior worth knowing

`RangeAnnotation` (`type: 'range'`) highlights a region. Field shape: `RangeAnnotation` in `index.d.ts`. Behavioral notes:

- **Shape choice:** Set only `x1`/`x2` for a vertical band, only `y1`/`y2` for a horizontal band, all four for a rectangle.
- **Opacity:** Keep low (0.1–0.2) so the data inside the band stays readable.
- **Label centering:** `labelAnchor: 'top' | 'bottom' | 'auto'` centers the label horizontally over the range. `'left'` (default) and `'right'` position the label at the corresponding edge.
- **Ordinal axis limitation:** Range annotations on ordinal x-axes may merge into one continuous band when multiple ranges are specified — ordinal scales position values at discrete band slots, not on a continuous number line, so two separate ranges (e.g. `2008–2010` and `2020–2022`) can render as one solid band spanning the full interval. For visually distinct shaded bands, switch the x-axis to `type: "temporal"` with `axis: { format: "%Y" }`. Range annotations work most reliably on temporal and quantitative axes.

## Reference Line — behavior worth knowing

`RefLineAnnotation` (`type: 'refline'`) draws a horizontal or vertical threshold line. Field shape: `RefLineAnnotation` in `index.d.ts`. Behavioral notes:

- Set `x` for a vertical line, `y` for a horizontal line.
- Use `style: 'dashed'` for targets/thresholds — dashed reads as "aspirational" vs solid "this is the data."
- **Label placement on horizontal reflines:** `labelAnchor: 'top'` (default) and `'right'` place the label above at the right end; `'bottom'` places it below at the right end; `'left'` places it above at the left end (near the y-axis).
- **Label placement on vertical reflines:** `'left'` (default) and `'top'` place the label to the right of the line near the top; `'right'` places it to the left of the line near the top; `'bottom'` places it near the bottom.
- Use `labelOffset: { dx, dy }` for fine tuning.
- **`labelAnchor: 'left' | 'right'` is silently ignored on reflines** — those values are accepted in the type but only render on `RangeAnnotation`. For a side label on a refline, set `label: ""` and add a separate `type: "text"` annotation positioned where you want the side label.

## Annotation Editing

Chart elements are draggable when `onEdit` is passed to `<Chart>`. This covers text annotations, connector endpoints, range/refline labels, chrome, series labels, and the legend.

See [editing reference](editing.md) for the full `onEdit` API, `ElementEdit` type, and how to persist each edit back to the spec.

**Gotcha: onEdit output contains an invalid field.** The `onEdit` callback may serialize text annotations with an `offset: { dx, dy }` object. This is an internal tracking field and is NOT part of the TextAnnotation schema. When copying annotation positions from console output back into a spec, remove any `offset` field and use top-level `dx`/`dy` inside `offset` at the schema level as documented above. Pasting an extra `offset` wrapper back into the spec will cause a renderer crash.

## Example: Annotated Line Chart

Two text annotations placed on opposite sides of the chart. The trough annotation uses `anchor: "bottom"` to push below the data; the peak uses `anchor: "top"` to push above. This gives the engine's collision resolver a good starting position for each label.

```json
{
  "mark": "line",
  "data": [
    { "month": "2023-01", "price": 48200 },
    { "month": "2023-04", "price": 29100 },
    { "month": "2023-07", "price": 30800 },
    { "month": "2023-10", "price": 34500 },
    { "month": "2024-01", "price": 42800 },
    { "month": "2024-04", "price": 63400 },
    { "month": "2024-07", "price": 57200 },
    { "month": "2024-10", "price": 72800 }
  ],
  "encoding": {
    "x": { "field": "month", "type": "temporal" },
    "y": {
      "field": "price",
      "type": "quantitative",
      "axis": { "format": "$,.0f" }
    }
  },
  "chrome": {
    "title": "Bitcoin surged past $70K after spot ETF approvals",
    "subtitle": "Monthly closing price, Jan 2023 - Oct 2024",
    "source": "Source: CoinGecko"
  },
  "annotations": [
    {
      "type": "refline",
      "y": 42800,
      "label": "Jan 2024: ETF approved",
      "style": "dashed",
      "stroke": "#c44e52"
    },
    {
      "type": "text",
      "x": "2023-04",
      "y": 29100,
      "text": "Crypto winter trough",
      "anchor": "bottom",
      "offset": { "dx": 0, "dy": 25 },
      "connector": true
    },
    {
      "type": "text",
      "x": "2024-10",
      "y": 72800,
      "text": "New all-time high",
      "anchor": "top",
      "offset": { "dx": 0, "dy": -25 },
      "connector": true
    }
  ]
}
```
