# Scatter Chart

Correlation between two quantitative variables. Optional size and color for 3rd/4th dimensions. Uses `mark: "point"`.

## Encoding Rules

| Channel | Required | Allowed types |
| --- | --- | --- |
| x | Yes | quantitative |
| y | Yes | quantitative |
| color | No | nominal, ordinal |
| size | No | quantitative |
| detail | No | nominal |

## Spec

```typescript
{
  mark: "point",
  data: DataRow[],
  encoding: {
    x: { field: string, type: "quantitative", axis?, scale? },
    y: { field: string, type: "quantitative", axis?, scale? },
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

**Tips:**
- Use `size` for bubble charts (3rd quantitative dimension)
- Use `color` to differentiate categories
- Use `labels: { density: "none" }` for dense scatter plots
- Use `scale: { zero: false }` on axes to zoom into the data range when zero is irrelevant

## Color Should Reinforce the Story

When the x-axis already encodes a meaningful variable (e.g., poverty rate), **use color to double-encode the same narrative**. Don't leave all dots the same color. Bucket the data into 3-4 tiers and map them to an ordinal color scale that progresses from cool (low) to warm (high). This makes the pattern legible at a glance, even before the reader processes axis positions.

Example: a scatter of poverty vs. chronic absence should color-code dots by poverty tier (e.g., `<15%` = blue, `15-40%` = pink, `40-60%` = green, `60%+` = orange/red). The color gradient reinforces the x-axis position and makes the correlation visible as a color gradient, not just a spatial pattern.

## Annotation Placement

Bubble charts need especially large annotation offsets because circles are big and often clustered. See the [annotations reference](annotations.md) for specific guidance on offset sizing and connector usage on dense charts.

## Builder

```typescript
import { scatterChart } from "@opendata-ai/openchart-core";

const spec = scatterChart(data, "spending", "lifeExp", {
  size: "pop",
  color: "continent",
  chrome: { title: "Health spending vs life expectancy" },
});
```

## Example

```json
{
  "mark": "point",
  "data": [
    { "country": "US", "spending": 12555, "lifeExp": 77.5, "pop": 331 },
    { "country": "Germany", "spending": 7383, "lifeExp": 81.7, "pop": 83 },
    { "country": "Japan", "spending": 4691, "lifeExp": 84.8, "pop": 125 },
    { "country": "UK", "spending": 5268, "lifeExp": 81.4, "pop": 67 },
    { "country": "Brazil", "spending": 1518, "lifeExp": 75.9, "pop": 214 }
  ],
  "encoding": {
    "x": {
      "field": "spending",
      "type": "quantitative",
      "axis": { "label": "Health spending per capita ($)" }
    },
    "y": {
      "field": "lifeExp",
      "type": "quantitative",
      "axis": { "label": "Life expectancy (years)" }
    },
    "size": { "field": "pop", "type": "quantitative" },
    "color": { "field": "country", "type": "nominal" }
  },
  "chrome": {
    "title": "Higher spending doesn't always mean longer lives",
    "subtitle": "Health expenditure per capita vs life expectancy, selected countries",
    "source": "Source: World Bank"
  }
}
```
