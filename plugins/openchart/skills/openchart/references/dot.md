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

## Spec

```typescript
{
  mark: "circle",
  data: DataRow[],
  encoding: {
    x: { field: string, type: "quantitative", axis?, scale? },
    y: { field: string, type: "nominal"|"ordinal", axis?, scale? },
    color?: { field: string, type: "nominal"|"ordinal" },
    size?: { field: string, type: "quantitative" },
  },
  chrome?: Chrome,
  annotations?: Annotation[],
  labels?: LabelConfig,
  legend?: LegendConfig,
  responsive?: boolean,
  theme?: ThemeConfig,
  darkMode?: DarkMode,
}
```

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
      "axis": { "format": "$,.0f", "label": "Annual salary" }
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
