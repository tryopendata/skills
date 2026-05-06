# Tile Map

US state tile grid map with sequential color encoding. Each state gets an equal-sized square tile arranged in a geographic approximation (11 columns x 8 rows, NPR/FiveThirtyEight style). No projection math or GeoJSON required.

**Use cases:** Unemployment by state, election results, population density, housing prices, BLS/Census/HUD data keyed by US state code.

**Important:** TileMap uses its own component, not `<Chart>`. Import `TileMap` from `@opendata-ai/openchart-react`:

```tsx
import { TileMap } from '@opendata-ai/openchart-react';
// NOT: import { Chart } from '../components/Chart'  // Chart only handles mark-based specs

<TileMap spec={{ type: 'tilemap', data: { CA: 4.4, TX: 17.6 } }} />
```

If your app uses class-based dark mode (not `prefers-color-scheme`), pass `darkMode="force"` or `darkMode="auto"` to the `<TileMap>` component prop.

## Data Formats

TileMap accepts two data shapes:

### Record map (simplest)

State codes mapped directly to numeric values. Encoding is auto-generated.

```json
{
  "type": "tilemap",
  "data": { "CA": 4.4, "TX": 17.6, "NY": 4.5, "FL": 3.2 }
}
```

Values can be `null` to explicitly mark a state as missing data.

### Tabular data rows

Standard `DataRow[]` format. Requires `encoding` with `state` and `value` channels.

```json
{
  "type": "tilemap",
  "data": [
    { "code": "CA", "rate": 4.4 },
    { "code": "TX", "rate": 17.6 }
  ],
  "encoding": {
    "state": { "field": "code", "type": "nominal" },
    "value": { "field": "rate", "type": "quantitative" }
  }
}
```

## Encoding

Only used with tabular `DataRow[]` input. Auto-generated for record maps.

| Channel | Required | Type |
| --- | --- | --- |
| state | Yes | nominal |
| value | Yes | quantitative |
| tooltip | No | any |

## TileMapSpec

For the full shape, load `TileMapSpec` from `index.d.ts`. Behavior worth knowing:

- `data` accepts either `Record<string, number | null>` (state code → value, encoding auto-generated) or `DataRow[]` (requires explicit `encoding.state` + `encoding.value`).
- `palette` defaults to `'blue'`. Allowed values: `'blue' | 'green' | 'orange' | 'purple'`.
- `watermark` defaults to `true`.
- `darkMode` defaults to `'off'` here, unlike charts which default to `'auto'`.
- `valueFormat` accepts d3-format with literal suffix extension (e.g. `".1f%"`, `"$,.0f"`, `"~s"`).

## Palette Options

Sequential color scales applied to tile fills. The color scale maps from `min(values)` to `max(values)`.

| Palette | Description |
| --- | --- |
| `'blue'` | Default. Blue sequential ramp. |
| `'green'` | Green sequential ramp. |
| `'orange'` | Orange sequential ramp. |
| `'purple'` | Purple sequential ramp. |

In dark mode, palette stop order is reversed so higher values get lighter (more visible) colors against the dark background.

## Coverage

All 51 tiles (50 states + DC) always render. States with data get colored by the sequential scale. States without data get a neutral fill and display "--" as the value. This ensures the geographic layout is always complete regardless of data coverage.

## Builder Function

```typescript
import { tileMap } from '@opendata-ai/openchart-core';

// Record map
const spec = tileMap({ CA: 4.4, TX: 17.6, NY: 4.5 });

// With options
const spec = tileMap(
  { CA: 4.4, TX: 17.6, NY: 4.5 },
  {
    palette: 'green',
    chrome: { title: 'Unemployment by State' },
    valueFormat: '.1f',
  }
);

// Tabular data
const spec = tileMap(data, {
  encoding: {
    state: { field: 'code', type: 'nominal' },
    value: { field: 'rate', type: 'quantitative' },
  },
});
```

## Examples

### Unemployment Rate by State

```json
{
  "type": "tilemap",
  "data": {
    "AL": 2.8, "AK": 5.3, "AZ": 3.5, "AR": 3.4, "CA": 5.3,
    "CO": 3.5, "CT": 4.1, "DE": 4.3, "DC": 5.2, "FL": 3.1,
    "GA": 3.4, "HI": 3.1, "ID": 3.0, "IL": 4.8, "IN": 3.3,
    "IA": 2.8, "KS": 3.0, "KY": 4.3, "LA": 3.8, "ME": 3.1,
    "MD": 3.0, "MA": 3.8, "MI": 4.1, "MN": 2.9, "MS": 3.6,
    "MO": 3.5, "MT": 2.7, "NE": 2.1, "NV": 5.5, "NH": 2.4,
    "NJ": 4.8, "NM": 3.9, "NY": 4.5, "NC": 3.7, "ND": 2.0,
    "OH": 4.0, "OK": 3.2, "OR": 4.2, "PA": 3.4, "RI": 3.8,
    "SC": 3.3, "SD": 2.0, "TN": 3.3, "TX": 4.2, "UT": 2.8,
    "VT": 2.1, "VA": 3.0, "WA": 4.7, "WV": 4.0, "WI": 2.9, "WY": 2.6
  },
  "palette": "blue",
  "valueFormat": ".1f",
  "chrome": {
    "title": "Nevada and Alaska lead the nation in unemployment",
    "subtitle": "Unemployment rate by state, seasonally adjusted, %",
    "source": "Source: Bureau of Labor Statistics"
  }
}
```

### Partial Data (10 States)

States without data get a neutral fill. Useful for highlighting a subset.

```json
{
  "type": "tilemap",
  "data": {
    "CA": 39538, "TX": 29146, "FL": 21538, "NY": 20202, "PA": 13003,
    "IL": 12812, "OH": 11800, "GA": 10712, "NC": 10439, "MI": 10078
  },
  "palette": "orange",
  "valueFormat": "~s",
  "chrome": {
    "title": "The 10 most populous states",
    "subtitle": "Population in thousands, 2020 Census",
    "source": "Source: U.S. Census Bureau"
  }
}
```

### Dark Mode

```json
{
  "type": "tilemap",
  "data": { "CA": 4.4, "TX": 17.6, "NY": 4.5 },
  "darkMode": "force",
  "palette": "green",
  "chrome": {
    "title": "Dark mode tile map"
  }
}
```

## When to Use vs. Alternatives

| Situation | Use |
| --- | --- |
| US state-level data, equal visual weight per state | Tile map |
| Geographic data needing accurate shapes/areas | Choropleth (GeoJSON projection, use D3 directly) |
| Comparing exact values across states | Bar chart (more precise than color) |
| Time series by state | Line chart with color encoding |
| Single-state deep dive | Bar or table |

## Value Formatting

Use `valueFormat` to format values on tiles, in the gradient legend, and in tooltips:

```json
{
  "type": "tilemap",
  "valueFormat": ".1f%",
  "data": { "CA": 4.4, "TX": 17.6 }
}
```

Accepts d3-format strings with literal suffix extension. Common patterns: `".1f%"` (append %), `"$,.0f"` (currency), `"~s"` (SI like 10k).

## Legend

A gradient legend bar renders below the tiles by default, showing the sequential color scale with min and max value labels. Hide it with `legend: { show: false }`.

## Dark Mode

TileMap uses `darkMode: "off"` by default (unlike charts which default to `"auto"`). For apps with class-based dark mode toggles, pass `darkMode="force"` to the component or set `darkMode: "force"` in the spec.

In dark mode, the sequential palette stops are reversed so higher values map to brighter colors for visibility against the dark background.

## Gotchas

| Issue | Details |
| --- | --- |
| State codes must be uppercase 2-letter | `"ca"` won't match. Use `"CA"`. The engine matches against standard USPS state codes. |
| Missing states still render | All 51 tiles always appear. States without data get a neutral fill and "--" value. This is intentional for geographic context. |
| Record map values must be numbers or null | String values like `"4.4"` will be coerced via `Number()`, but explicit numbers are safer. |
| Encoding required for tabular data | If `data` is `DataRow[]`, you must provide `encoding` with `state` and `value` channels. Record maps auto-generate encoding. |
| No negative-value palette | The sequential scale includes negative values in the domain, but the single-direction palette doesn't communicate negative vs. positive well. Use a bar chart for data with meaningful negative values. |
| Small containers drop value labels | Below 24px tile width, value labels are hidden. Below 16px, state code font shrinks. The gradient legend hides below 200px container width. |
| React linked packages and Vite | When developing with `bun link` across repos, Vite aggressively caches pre-bundled dependencies. Always clear `node_modules/.vite` after rebuilding linked packages, and ensure ALL transitive packages are linked (core, engine, vanilla, react). |
