# Dot Chart (Strip Plot)

Distribution and spread of values across categories. Uses `mark: "circle"`.

## Encoding Rules

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | Yes | quantitative |
| y | Yes | nominal, ordinal |
| color | No | nominal, ordinal |
| size | No | quantitative |
| detail | No | nominal |

## Spec shape

For the full `MarkDef` for `mark: "circle"` and the encoding surface, load `MarkDef`, `Encoding`, and `ChartSpec` from `index.d.ts`.

## Example

```json
{
  "mark": "circle",
  "data": [
    { "dept": "Engineering", "salary": 145000 },
    { "dept": "Engineering", "salary": 132000 },
    { "dept": "Engineering", "salary": 128000 },
    { "dept": "Design", "salary": 115000 },
    { "dept": "Design", "salary": 108000 },
    { "dept": "Marketing", "salary": 98000 },
    { "dept": "Marketing", "salary": 92000 },
    { "dept": "Marketing", "salary": 105000 }
  ],
  "encoding": {
    "x": {
      "field": "salary",
      "type": "quantitative",
      "axis": { "format": "$,.0f", "title": "Annual salary" }
    },
    "y": { "field": "dept", "type": "nominal" }
  },
  "chrome": {
    "title": "Engineering salaries skew higher than other departments",
    "subtitle": "Individual salary distribution by department",
    "source": "Source: HR data, 2024"
  },
  "labels": { "density": "none" }
}
```

## Related dot-family marks

- **`mark: "lollipop"`** -- a dot on a stem from the baseline, for ranked categorical values. `x` quantitative, `y` nominal/ordinal (horizontal). Negative values extend the stem left of the baseline. Reads as a lighter-weight bar chart when the bar's area would be visual overkill.
- **`mark: "beeswarm"`** -- one dot per observation, dodged so points don't overlap. Give it one quantitative positional channel (the value axis) plus an optional nominal channel for grouped lanes; `size` scales dot area. Use it over `circle` when you have many observations per group and want every point visible. Gotcha: the cross axis is pure pixel-space with no scale, so a tall stack can overflow a short container -- cap the `size` range or give it vertical room.

For the full `MarkDef` and encoding surface of either, load `MarkDef`, `Encoding`, and `ChartSpec` from `index.d.ts`.
