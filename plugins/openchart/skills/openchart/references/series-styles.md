# Per-Series Styling

Override visual properties for individual series by name. Useful for making reference series visually distinct from primary data.

```typescript
seriesStyles?: Record<string, {
  lineStyle?: "solid"|"dashed"|"dotted",
  showPoints?: boolean,
  strokeWidth?: number,
  opacity?: number,
}>
```

Series names come from the `color` encoding field values. Only specified series get overrides; others render normally.

## Example

Show "Fed funds rate" as a dashed reference line alongside solid CPI data:

```json
{
  "seriesStyles": {
    "Fed funds rate": {
      "lineStyle": "dashed",
      "showPoints": false,
      "strokeWidth": 1.5,
      "opacity": 0.6
    }
  }
}
```

## Common patterns

| Pattern | Use case |
| --- | --- |
| `"lineStyle": "dashed"` | Benchmark / reference series |
| `"opacity": 0.4` | Context series that shouldn't dominate |
| `"showPoints": true` | Sparse series where individual values matter |
| `"strokeWidth": 3` | Hero series to emphasize |
