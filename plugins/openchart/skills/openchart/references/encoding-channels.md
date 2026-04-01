# Encoding Channels

Charts (line, area, bar, arc, point, circle, text, rule, tick, rect) use encoding channels to map data fields to visual properties.

## Channel List

```typescript
encoding: {
  x?: EncodingChannel,           // horizontal position
  y?: EncodingChannel,           // vertical position
  color?: EncodingChannel,       // series differentiation
  size?: EncodingChannel,        // bubble/dot scaling (quantitative)
  detail?: EncodingChannel,      // grouping without visual mapping
  opacity?: EncodingChannel,     // data-driven opacity
  shape?: EncodingChannel,       // point shape (circle, square, diamond, etc.)
  strokeDash?: EncodingChannel,  // data-driven dash patterns
  text?: EncodingChannel,        // text content for text marks
  tooltip?: EncodingChannel | EncodingChannel[],  // tooltip fields
  x2?: EncodingChannel,          // secondary x (ranges)
  y2?: EncodingChannel,          // secondary y (ranges)
  theta?: EncodingChannel,       // angular position (arc marks)
  radius?: EncodingChannel,      // radial position (arc marks)
}
```

## Stack Control

The `stack` property on quantitative encoding channels controls how multi-series bar/column charts handle overlapping categories. Follows Vega-Lite conventions.

| Value | Behavior |
| --- | --- |
| `undefined` / `true` / `"zero"` | Stacked (default). Segments stack cumulatively. |
| `null` / `false` | Grouped/dodged. Bars render side-by-side within each category. |

Set `stack` on the **quantitative** channel (x for horizontal bars, y for vertical columns).

**Grouped horizontal bar:**
```json
{
  "encoding": {
    "x": { "field": "weeks", "type": "quantitative", "stack": null },
    "y": { "field": "expense", "type": "nominal" },
    "color": { "field": "era", "type": "nominal" }
  }
}
```

**Grouped vertical column:**
```json
{
  "encoding": {
    "x": { "field": "year", "type": "nominal" },
    "y": { "field": "capacity", "type": "quantitative", "stack": null },
    "color": { "field": "type", "type": "nominal" }
  }
}
```

**When to use grouped vs stacked:** Use stacked when values represent parts of a whole (e.g., budget breakdown by category). Use grouped when values are independent comparisons (e.g., 1984 vs 2024 costs) where stacking would misrepresent the data by making it look cumulative.

## Conditional Encoding

Use `condition` + `value` to apply different visual properties based on data values:

```json
{
  "encoding": {
    "color": {
      "condition": { "test": { "field": "value", "gte": 0 }, "value": "#4CAF50" },
      "value": "#F44336"
    }
  }
}
```

When `condition.test` is true, the `condition.value` is used. Otherwise, the top-level `value` is the fallback. The `test` object uses the same filter predicate syntax as `FilterTransform` (see [data-transforms.md](data-transforms.md)).
