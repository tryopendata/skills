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
