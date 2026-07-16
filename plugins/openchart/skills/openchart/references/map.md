# Geo Map (Choropleth + Symbol)

Real-geography maps rendered from TopoJSON: choropleths (features filled by a quantitative value) and symbol maps (lat/lon points sized and colored over a basemap). Uses `type: "map"`. The library ships no boundary data — you supply TopoJSON, normally from the npm-standard atlases.

**Use cases:** unemployment by state, GDP by country, election results by county, store/campus locations over a state basemap, anything where actual shapes and areas matter.

**Important:** GeoMap uses its own component, not `<Chart>`:

```tsx
import { GeoMap } from '@opendata-ai/openchart-react'; // same name in vue/svelte
// vanilla: import { createMap } from '@opendata-ai/openchart-vanilla'

<GeoMap spec={mapSpec} />
```

## Getting basemaps

```bash
npm i us-atlas world-atlas
```

| Import | Use for | Pair with projection |
| --- | --- | --- |
| `us-atlas/states-albers-10m.json` | US states (recommended US default) | `"identity"` — pre-projected, AK/HI insets baked in, zero runtime spherical math |
| `us-atlas/counties-albers-10m.json` | US counties (~3,100 features) | `"identity"` |
| `us-atlas/states-10m.json` | US states, unprojected | `"albersUsa"` |
| `world-atlas/countries-110m.json` | World countries | `"equalEarth"` (equal-area) or `"mercator"` |

Both packages are public-domain data (US Census / Natural Earth) under ISC. Pass the imported topology as `geo.features`.

## Spec shape

```json
{
  "type": "map",
  "geo": { "features": "<TopoJSON import>", "projection": "identity" },
  "data": [{ "id": "48", "rate": 4.2 }],
  "encoding": {
    "key": { "field": "id", "type": "nominal" },
    "color": { "field": "rate", "type": "quantitative" }
  }
}
```

- `geo.features` (required): the TopoJSON topology object.
- `geo.idField`: feature property used as the join id. Defaults to `"id"` (correct for both atlases).
- `geo.projection`: `"albersUsa"` (default, AK/HI insets) | `"mercator"` | `"equalEarth"` | `"identity"` (pre-projected atlases only).
- `encoding.key`: the data field that joins rows to feature ids. Required for choropleth; optional in basemap-only mode.
- `encoding.color`: drives feature fill. Quantitative gets a **quantile** color scale (maps always bin by quantile) with a continuous legend; nominal gets categorical fills via `scale.domain`/`scale.range`.
- For the full shape, load `MapSpec`, `MapGeo`, `MapEncoding`, `MapPointsLayer` from `index.d.ts`.

## The join (where every geo tool dies)

Feature ids in the atlases are **strings**: 2-digit state FIPS (`"48"` = Texas), 5-digit county FIPS (`"48453"`), ISO 3166-1 numeric for world countries (`"840"` = US, `"250"` = France — NOT alpha codes or names). Your data's key column must match.

- Zero-pad FIPS codes: `1` and `"1001"` won't match `"01"` / `"01001"`. This is the classic CSV bug — leading zeros get stripped somewhere upstream.
- Name-based joins get one mercy: case-insensitive exact matching. No fuzzy matching.
- Unmatched data rows produce a compile warning (`UNMATCHED_DATA_KEYS`) naming the offending keys and suggesting the normalization (e.g. zero-padding). The chart still renders.
- Features with no data get a neutral fill plus one aggregate warning (`UNMATCHED_FEATURES`) — partial data is a supported pattern, not an error.

## Color

Quantitative color supports the sequential palettes only: `blue` (default), `green`, `orange`, `purple`, `teal`, set via `scale: { scheme: "green" }`. Anything else (including VL names like `"blues"` or `"viridis"`) fails validation — there is no silent fallback. Categorical color ignores `scheme` entirely (also a validation error); use `scale.domain` + `scale.range` for explicit category colors.

Value formatting in tooltips and the legend uses the **top-level `valueFormat`** (d3-format, e.g. `".1f"`, `"$,.0f"`). Maps do not read `encoding.color.format` — this differs from other spec families.

## Points layer (symbol maps)

`points` renders circles at lat/lon coordinates above the basemap, projected through the same projection:

```json
{
  "type": "map",
  "geo": { "features": "<states topo>", "projection": "albersUsa", "focus": { "features": "48", "padding": 8 } },
  "data": [],
  "encoding": { "key": { "field": "id", "type": "nominal" } },
  "points": {
    "data": [{ "lat": 30.27, "lon": -97.74, "name": "Austin", "value": 950 }],
    "longitude": { "field": "lon", "type": "quantitative" },
    "latitude": { "field": "lat", "type": "quantitative" },
    "size": { "field": "value", "type": "quantitative" },
    "color": { "field": "name", "type": "nominal" },
    "tooltip": [{ "field": "name", "type": "nominal", "title": "City" }]
  }
}
```

- `data: []` with no `encoding.color` = basemap-only mode: neutral features, points carry the story.
- `size` is area-proportional (r ∝ √value). No size legend renders — explain the scale in `chrome.subtitle`.
- `color` has its own scale, independent of the choropleth: quantitative uses the sequential palettes (same `scheme` rules), nominal honors `scale.domain`/`scale.range`.
- `opacity` defaults to 0.65 so overlapping points read as density.
- Data-update transitions for points are not yet supported.

## Camera (`geo.focus`)

Zoom/pan the map to a subset, resolved at compile time (no interactive panning):

| Value | Behavior |
| --- | --- |
| `"48"` or `["48", "35"]` | Fit those feature id(s) |
| `{ "features": "48", "padding": 8 }` | Fit with breathing room (map-local units) |
| `{ "points": true, "padding": 8 }` | Fit the point layer's cluster — use when points occupy a small corner of a big feature |
| `{ "points": { "field": "rating", "value": "F" } }` | Fit only the matching point subset (lets a story pan between sub-clusters) |
| `null` | Clear focus from a prior story step |

An id that matches nothing warns (`FOCUS_UNMATCHED`) and renders unfocused. Focus composes with the story API — panning between focus targets across scrolly steps works.

## Examples

### US state choropleth (the standard recipe)

```jsonc
{
  "type": "map",
  "geo": { "features": usStatesTopo, "projection": "identity" }, // us-atlas/states-albers-10m.json
  "data": [
    { "id": "32", "rate": 5.5 }, { "id": "02", "rate": 5.3 }, { "id": "48", "rate": 4.2 }
    // ... 2-digit FIPS strings
  ],
  "encoding": {
    "key": { "field": "id", "type": "nominal" },
    "color": { "field": "rate", "type": "quantitative" }
  },
  "valueFormat": ".1f",
  "chrome": {
    "title": "Where the Job Market Is Tightest",
    "subtitle": "State unemployment rate, seasonally adjusted, %",
    "source": "Source: Bureau of Labor Statistics"
  }
}
```

### World choropleth

```jsonc
{
  "type": "map",
  "geo": { "features": worldTopo, "projection": "equalEarth" }, // world-atlas/countries-110m.json
  "data": [
    { "id": "840", "gdp": 80035 }, { "id": "156", "gdp": 12720 }, { "id": "392", "gdp": 33815 }
    // ... ISO 3166-1 numeric strings
  ],
  "encoding": {
    "key": { "field": "id", "type": "nominal" },
    "color": { "field": "gdp", "type": "quantitative" }
  },
  "valueFormat": "$,.0f",
  "chrome": {
    "title": "The World in Dollars Per Person",
    "subtitle": "GDP per capita, current US$, 2023",
    "source": "Source: World Bank"
  }
}
```

Prefer `equalEarth` for world thematic maps — Mercator inflates high-latitude areas, which visually distorts exactly the comparison a choropleth invites.

## When to use vs. alternatives

| Situation | Use |
| --- | --- |
| US state data, equal visual weight per state | Tile map (`type: "tilemap"`) — small states stay readable |
| Accurate shapes/areas matter, or non-state geographies (counties, countries) | Geo map (`type: "map"`) |
| Point locations over geography | Geo map with `points` layer |
| Comparing exact values across regions | Bar chart or barlist (color is the lowest-precision channel) |
| Interactive slippy map, tile basemaps, panning | Leaflet/MapLibre — out of scope for openchart |

## Gotchas

| Issue | Details |
| --- | --- |
| Join keys are strings with leading zeros | `data: [{ id: 1 }]` won't match feature `"01"`. Zero-pad state FIPS to 2 digits, county FIPS to 5. The unmatched-keys warning tells you this, but get it right up front. |
| World ids are ISO numeric, not names or alpha codes | `"United States"`, `"US"`, `"USA"` all fail against `"840"` in world-atlas. Convert before joining. |
| Pre-projected atlases need `projection: "identity"` | `states-albers-10m` / `counties-albers-10m` are already in projected coordinates. Running them through `albersUsa` again produces garbage. |
| `albersUsa` drops territories | Puerto Rico, Guam, etc. project to null and warn (`NULL_PROJECTION`). Use `mercator` if territories matter. |
| 110m world atlas drops microstates | Monaco, Singapore, Andorra, Liechtenstein are absent at 110m — their data rows warn as unmatched. Use `countries-50m` (larger) if you need them. |
| Only sequential palettes | `scale.scheme` must be `blue`/`green`/`orange`/`purple`/`teal`. No diverging ramps on maps yet; no VL alias names. |
| `valueFormat`, not `encoding.color.format` | Maps read the top-level `valueFormat` only. |
| GIS-exported TopoJSON can render inside-out | d3-geo expects clockwise exterior rings (opposite of RFC 7946). The compile warns (`INVERTED_WINDING`) and names the fix (`topojson rewind`). The npm atlases are wound correctly. |
| `darkMode` defaults to `"off"` | Like tilemap, unlike charts (`"auto"`). Pass `darkMode="force"` for class-based dark mode apps. |

## Legend

Quantitative maps render the continuous (binned quantile) legend by default. `legend: { show: false }` hides it; `legend: { position: "bottom" }` moves it below the map (default top).
