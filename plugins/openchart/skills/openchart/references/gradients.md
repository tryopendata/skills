# Gradient Fills

Gradient fills can be applied to any chart mark that uses a `fill` property (bar, area, arc, point). Gradients follow Vega's gradient value type model: they can appear anywhere a color string can.

## GradientDef Types

```typescript
interface GradientStop {
  offset: number;   // 0-1, position along gradient
  color: string;    // CSS color
  opacity?: number; // 0-1, maps to SVG stop-opacity
}

interface LinearGradient {
  gradient: 'linear';
  stops: GradientStop[];
  x1?: number; // start x (0-1), default 0
  y1?: number; // start y (0-1), default 0
  x2?: number; // end x (0-1), default 0
  y2?: number; // end y (0-1), default 1 (top-to-bottom)
}

interface RadialGradient {
  gradient: 'radial';
  stops: GradientStop[];
  x1?: number; y1?: number; r1?: number; // inner circle (focal point)
  x2?: number; y2?: number; r2?: number; // outer circle
}
```

Coordinates are in **[0,1] normalized space** relative to the mark's bounding box. The default linear gradient runs top-to-bottom. Set `x1:0, y1:0, x2:1, y2:0` for left-to-right.

## Usage Patterns

### Static gradient on all marks via mark.fill

```json
{
  "mark": {
    "type": "bar",
    "fill": {
      "gradient": "linear",
      "stops": [
        { "offset": 0, "color": "#1b7fa3" },
        { "offset": 1, "color": "#1b7fa3", "opacity": 0.4 }
      ]
    }
  }
}
```

### Left-to-right gradient for horizontal bars

```json
{
  "mark": {
    "type": "bar",
    "fill": {
      "gradient": "linear",
      "x1": 0, "y1": 0, "x2": 1, "y2": 0,
      "stops": [
        { "offset": 0, "color": "#1b7fa3", "opacity": 0.4 },
        { "offset": 1, "color": "#1b7fa3" }
      ]
    }
  }
}
```

### Per-bar conditional gradients via color encoding

Each bar gets a different gradient based on data values:

```json
{
  "mark": "bar",
  "encoding": {
    "x": { "field": "metric", "type": "nominal" },
    "y": { "field": "value", "type": "quantitative" },
    "color": {
      "condition": [
        {
          "test": { "field": "status", "equal": "good" },
          "value": {
            "gradient": "linear",
            "stops": [
              { "offset": 0, "color": "#059669" },
              { "offset": 1, "color": "#a7f3d0" }
            ]
          }
        },
        {
          "test": { "field": "status", "equal": "bad" },
          "value": {
            "gradient": "linear",
            "stops": [
              { "offset": 0, "color": "#dc2626" },
              { "offset": 1, "color": "#dc2626", "opacity": 0.3 }
            ]
          }
        }
      ],
      "value": "#94a3b8"
    }
  }
}
```

### Area fade-to-transparent

```json
{
  "mark": {
    "type": "area",
    "fill": {
      "gradient": "linear",
      "stops": [
        { "offset": 0, "color": "#6366f1", "opacity": 0.85 },
        { "offset": 1, "color": "#6366f1", "opacity": 0.1 }
      ]
    }
  }
}
```

### Radial gradient on donut slices

```json
{
  "mark": { "type": "arc", "innerRadius": 40 },
  "encoding": {
    "y": { "field": "share", "type": "quantitative" },
    "color": {
      "condition": [
        {
          "test": { "field": "category", "equal": "Organic" },
          "value": {
            "gradient": "radial",
            "stops": [
              { "offset": 0, "color": "#0ea5e9" },
              { "offset": 1, "color": "#0369a1" }
            ]
          }
        }
      ],
      "value": "#94a3b8"
    }
  }
}
```

### Multi-stop gradient (3+ colors)

```json
{
  "gradient": "linear",
  "stops": [
    { "offset": 0, "color": "#059669" },
    { "offset": 0.5, "color": "#34d399" },
    { "offset": 1, "color": "#a7f3d0" }
  ]
}
```

## Direction Quick Reference

| Direction | Coordinates |
|-----------|-------------|
| Top to bottom (default) | `x1:0, y1:0, x2:0, y2:1` |
| Bottom to top | `x1:0, y1:1, x2:0, y2:0` |
| Left to right | `x1:0, y1:0, x2:1, y2:0` |
| Right to left | `x1:1, y1:0, x2:0, y2:0` |
| Diagonal | `x1:0, y1:0, x2:1, y2:1` |

## When to use horizontal bars

Use left-to-right gradients (`x1:0, y1:0, x2:1, y2:0`) for horizontal bars where `x: quantitative, y: nominal`. The gradient direction should match the data direction (the bar grows left to right, so the gradient should too).

## SVG rendering details

Gradients create `<linearGradient>` or `<radialGradient>` elements in the SVG `<defs>`. The renderer uses `gradientUnits="objectBoundingBox"`, which means coordinates are relative to each mark's bounding box. Identical gradients are deduplicated (one SVG element shared by all marks with the same gradient definition).

## Marks that don't support gradients

- **line** marks use `stroke`, not `fill`. Line gradients along the path are not supported.
- **text** marks always use flat fill colors.
- **rule** and **tick** marks use stroke colors only.

## Utilities

- `isGradientDef(value)` - type guard to check if a value is a GradientDef
- `getRepresentativeColor(fill)` - extracts a flat color from a gradient (returns last stop color). Used internally by tooltips, labels, and legends.
