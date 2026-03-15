# Infographic Design Principles

Creating publication-quality visualizations that tell stories, not just show data.

## Contents

- The infographic mindset
- Tufte's principles
- Headline-first design
- Annotation strategies
- Social media optimization
- Quality checklist

## The infographic mindset

### Charts vs Infographics

| Chart | Infographic |
|-------|-------------|
| Shows data | Tells a story |
| Title describes chart type | Headline states the insight |
| All data labeled | Only key data labeled |
| Generic styling | Distinctive, publication-quality |
| Viewer finds meaning | Designer guides meaning |

**Goal:** A viewer should understand the insight in 3 seconds by reading the headline, then the chart confirms it.

### The Chartr/Datawrapper aesthetic

What makes publication-quality visualizations distinctive:

1. **Headline-first** - The insight leads, chart confirms
2. **Minimal chrome** - No gridlines, sparse axes, no 3D
3. **Direct labeling** - Labels on data, not in legends
4. **Strategic annotations** - Highlight what matters
5. **Consistent typography** - Professional type hierarchy
6. **Intentional color** - Color means something

## Tufte's principles

Edward Tufte's core principles for honest, effective data visualization:

### Data-ink ratio

> "Above all else, show the data."

Every pixel should either show data or help interpret it. Remove:
- Decorative gridlines
- Background patterns
- 3D effects
- Unnecessary borders
- Redundant labels

```typescript
// Bad: Visual noise
<rect fill="#f5f5f5" />  {/* Background box */}
{gridLines.map(y => <line stroke="#ddd" />)}  {/* Many gridlines */}
<text>Value</text>  {/* Axis title when axis is obvious */}

// Good: Just data
<path d={linePath} stroke={palette.primary} />  {/* The data */}
<text>{formatNumber(maxValue)}</text>  {/* Key value */}
```

### Graphical integrity

**Lie factor:** Size of effect in graphic / Size of effect in data

Should always be close to 1. Don't:
- Start bar charts at non-zero (unless clearly marked)
- Use 3D that distorts perception
- Use area encoding without sqrt scaling
- Cherry-pick axis ranges

### Small multiples

When comparing many series, use small multiples instead of one complex chart:

```typescript
// Instead of 10 overlapping lines, use a grid
<div className="grid grid-cols-5 gap-4">
  {series.map(s => (
    <SmallChart key={s.name} data={s.data} title={s.name} />
  ))}
</div>
```

### Micro/macro readings

Design charts that work at two levels:
- **Macro:** Overall pattern visible at a glance
- **Micro:** Details available on closer inspection

## Headline-first design

### Writing effective headlines

| Bad | Good | Why |
|-----|------|-----|
| "Revenue by Year (2018-2024)" | "Revenue doubled in six years" | States the insight |
| "COVID Cases Over Time" | "Cases peaked in January 2022" | Specific finding |
| "Survey Results" | "78% prefer remote work" | Quantifies the story |

### Headline formula

```
[What happened] + [How much/when/who]
```

Examples:
- "Streaming revenue surpassed box office in 2020"
- "Gen Z saves 50% more than millennials did at same age"
- "Housing prices fell for first time in 10 years"

### Implementation

```typescript
interface InfographicProps {
  headline: string;      // Required: The insight
  subtitle?: string;     // Optional: Context/methodology
  source: string;        // Required: Data attribution
}

<text
  x={margin.left}
  y={32}
  fontSize={32}
  fontWeight={800}
  fill={palette.text}
  style={{ letterSpacing: "-0.02em" }}
>
  {headline}
</text>
{subtitle && (
  <text
    x={margin.left}
    y={56}
    fontSize={14}
    fill={palette.textMuted}
  >
    {subtitle}
  </text>
)}
```

## Annotation strategies

### Annotation hierarchy

| Level | Visual Treatment | When to Use |
|-------|------------------|-------------|
| **Callout** | Large text, leader line, circle highlight | Single most important insight |
| **Label** | Medium text, inline with data | 2-3 supporting facts |
| **Note** | Small text, footer | Methodology, caveats |

### What to annotate

Automatically detect and consider annotating:
- **Maximum/minimum values** - "Peak: $4.2B"
- **Inflection points** - "Growth reversed in Q3"
- **Threshold crossings** - "First quarter above $1B"
- **Significant changes** - "+42% YoY"
- **Outliers** - "Excluding one-time events"

```typescript
function suggestAnnotations(data: DataPoint[]): Annotation[] {
  const annotations: Annotation[] = [];

  // Find max
  const max = data.reduce((m, d) => d.value > m.value ? d : m);
  annotations.push({
    type: "callout",
    point: max,
    text: `Peak: ${formatNumber(max.value)}`,
  });

  // Find min (if significantly lower)
  const min = data.reduce((m, d) => d.value < m.value ? d : m);
  if (max.value / min.value > 2) {
    annotations.push({
      type: "label",
      point: min,
      text: `Low: ${formatNumber(min.value)}`,
    });
  }

  // Find trend changes
  const inflections = findInflectionPoints(data);
  inflections.slice(0, 2).forEach(point => {
    annotations.push({
      type: "label",
      point,
      text: `Trend shift`,
    });
  });

  return annotations;
}
```

### Visual treatment

```typescript
function CalloutAnnotation({ point, text, xScale, yScale, palette }) {
  const x = xScale(point.label);
  const y = yScale(point.value);

  return (
    <g>
      {/* Highlight circle */}
      <circle
        cx={x}
        cy={y}
        r={10}
        fill={palette.primary}
        fillOpacity={0.15}
        stroke={palette.primary}
        strokeWidth={2}
      />

      {/* Leader line */}
      <line
        x1={x + 8}
        y1={y - 8}
        x2={x + 40}
        y2={y - 30}
        stroke={palette.textMuted}
        strokeWidth={1}
      />

      {/* Label */}
      <text
        x={x + 44}
        y={y - 30}
        fontSize={14}
        fontWeight={600}
        fill={palette.text}
      >
        {text}
      </text>
    </g>
  );
}
```

## Social media optimization

### Platform dimensions

| Platform | Dimensions | Aspect | Notes |
|----------|------------|--------|-------|
| Twitter/X | 1200 x 675 | 16:9 | Optimal for timeline cards |
| Instagram Feed | 1080 x 1080 | 1:1 | Square format |
| LinkedIn | 1200 x 627 | 1.91:1 | Article preview |
| Instagram Story | 1080 x 1920 | 9:16 | Vertical, full-screen |

### Adapting layouts

```typescript
function getLayout(aspect: "16:9" | "1:1" | "9:16") {
  switch (aspect) {
    case "16:9":
      return {
        width: 1200,
        height: 675,
        headlineSize: 48,
        chartHeight: 400,
        margins: { top: 80, right: 48, bottom: 80, left: 48 },
      };
    case "1:1":
      return {
        width: 1080,
        height: 1080,
        headlineSize: 56,
        chartHeight: 600,
        margins: { top: 120, right: 56, bottom: 180, left: 56 },
      };
    case "9:16":
      return {
        width: 1080,
        height: 1920,
        headlineSize: 64,
        chartHeight: 800,
        margins: { top: 200, right: 56, bottom: 400, left: 56 },
      };
  }
}
```

### Watermark placement

```typescript
// Bottom-right, contrast-aware
function Watermark({ width, height, palette }) {
  return (
    <g transform={`translate(${width - 48}, ${height - 24})`}>
      <text
        textAnchor="end"
        fontSize={12}
        fontWeight={500}
        fill={palette.textMuted}
        fillOpacity={0.7}
      >
        OpenData
      </text>
    </g>
  );
}
```

### Source attribution

Always include data source:

```typescript
<text
  x={margin.left}
  y={height - 16}
  fontSize={11}
  fill={palette.textMuted}
>
  Source: {source}
</text>
```

## OpenData branding

### Watermark specifications

```typescript
interface WatermarkConfig {
  position: "bottom-right";
  offset: 48;  // px from edges
  height: {
    twitter: 60,    // px
    instagram: 72,  // px
  };
  opacity: 0.85;
}

function Watermark({ width, height, variant, format }: WatermarkProps) {
  const watermarkHeight = format === "instagram" ? 72 : 60;

  // Contrast-aware: light variant for dark areas
  const fill = variant === "light" ? "#FFFFFF" : "#1A1A1A";

  return (
    <g transform={`translate(${width - 48}, ${height - 48})`}>
      <image
        href={`/watermark-${variant}.svg`}
        width={watermarkHeight * 2}  // Maintain aspect ratio
        height={watermarkHeight}
        x={-watermarkHeight * 2}
        opacity={0.85}
      />
    </g>
  );
}
```

### Source attribution

```typescript
// Format options
const sourceFormats = {
  opendata: (name: string) => `Data: ${name} via OpenData`,
  external: (name: string) => `Source: ${name}`,
};

function SourceAttribution({ source, isOpenDataset, palette }) {
  const text = isOpenDataset
    ? sourceFormats.opendata(source)
    : sourceFormats.external(source);

  return (
    <text
      x={margin.left}
      y={height - 16}
      fontSize={12}
      fontWeight={400}
      fill="#9CA3AF"
    >
      {text}
    </text>
  );
}
```

## Data density guidelines

Infographics work best with focused data. Auto-suggest simplification:

| Chart Type | Max Rows | Auto-Suggestion |
|------------|----------|-----------------|
| Trend Line | 50 | "Aggregate to quarterly?" |
| Comparison Lines | 200 (40 per series × 5) | "Show top 5 series?" |
| Horizontal Bars | 15 | "Show top 10 with Others rollup?" |
| Vertical Bars | 30 | "Show recent 24 periods?" |
| Stacked Bars | 50 | "Reduce to 3-4 segments?" |
| Donut Chart | 6 | "Group small slices into Other?" |
| Scatter Plot | 100 | "Showing sample of N points" |

### Label density rules

```typescript
function getLabelStrategy(data: DataPoint[], chartType: string) {
  const count = data.length;

  if (chartType === "time-series") {
    // Label first, last, and max/min only
    return {
      show: [0, count - 1, findMaxIndex(data), findMinIndex(data)],
      strategy: "sparse",
    };
  }

  if (chartType === "bar") {
    if (count <= 8) return { show: "all", strategy: "all" };
    if (count <= 15) return { show: "every-other", strategy: "alternate" };
    return { show: "hover", strategy: "interactive" };
  }

  return { show: "none", strategy: "hover-only" };
}
```

## Memorable metaphor patterns

The best infographics transcend charts. Visual Capitalist examples:

### Geographic metaphors

- Rivers scaled by discharge cutting through forest shapes
- Satellite-style effects making geography feel alive
- Circular smoke rings representing emissions by country

### Physical metaphors

- Stack of coins for cumulative value
- Building heights for company valuations
- Tree rings for historical timeline

### When to use metaphors

| Use | Avoid |
|-----|-------|
| Editorial/viral content | Business dashboards |
| Annual reports | Real-time data |
| Explainer content | Dense multi-metric views |
| Hero statistics | Comparative analysis |

**Rule:** Metaphor must aid understanding, not just decoration. If it makes the data harder to read, use a standard chart.

## Quality checklist

### Before publishing

**Story:**
- [ ] Headline states the insight (not "Chart of X")
- [ ] A viewer can understand in 3 seconds
- [ ] Annotations highlight what matters

**Data integrity:**
- [ ] Axes start at appropriate values (usually 0 for bars)
- [ ] Scale is honest (no distortion)
- [ ] Source is attributed

**Visual clarity:**
- [ ] No gridlines (or very subtle)
- [ ] Sparse axis labels (not every value)
- [ ] Direct labels on data (not separate legend)
- [ ] Color has meaning (not decorative)

**Typography:**
- [ ] Clear hierarchy (headline > subtitle > labels > axis)
- [ ] Tabular numbers for values
- [ ] Readable at target size

**Accessibility:**
- [ ] Colorblind-safe palette
- [ ] Sufficient contrast
- [ ] Text is large enough (min 11px)

### Common fixes

| Problem | Fix |
|---------|-----|
| "I don't know what this shows" | Write headline that states insight |
| "Too busy" | Remove gridlines, reduce labels |
| "Colors are confusing" | Use one primary color, gray everything else |
| "Can't tell values apart" | Use bar chart instead of pie |
| "Legend is confusing" | Direct label on data |

## Templates

### Trend with callout

```typescript
function TrendWithCallout({ data, headline, callout, palette }) {
  const maxPoint = data.reduce((m, d) => d.value > m.value ? d : m);

  return (
    <svg width={800} height={500}>
      {/* Headline */}
      <text x={48} y={40} fontSize={28} fontWeight={800}>
        {headline}
      </text>

      {/* Chart */}
      <g transform="translate(48, 80)">
        <path d={linePath} stroke={palette.primary} strokeWidth={3} />

        {/* Callout on max */}
        <CalloutAnnotation
          point={maxPoint}
          text={callout || `Peak: ${formatNumber(maxPoint.value)}`}
        />
      </g>

      {/* Source */}
      <text x={48} y={480} fontSize={11} fill={palette.textMuted}>
        Source: {source}
      </text>
    </svg>
  );
}
```

### Comparison bars

```typescript
function ComparisonBars({ data, headline, highlightItem, palette }) {
  return (
    <svg width={800} height={500}>
      <text x={48} y={40} fontSize={28} fontWeight={800}>
        {headline}
      </text>

      <g transform="translate(48, 80)">
        {data.map((d, i) => (
          <g key={d.category} transform={`translate(0, ${i * 48})`}>
            {/* Bar */}
            <rect
              width={xScale(d.value)}
              height={36}
              fill={d.category === highlightItem
                ? palette.primary
                : palette.textMuted}
              fillOpacity={d.category === highlightItem ? 1 : 0.3}
            />

            {/* Label + Value */}
            <text x={-8} y={22} textAnchor="end" fontSize={13}>
              {d.category}
            </text>
            <text
              x={xScale(d.value) + 8}
              y={22}
              fontSize={13}
              fontWeight={600}
            >
              {formatNumber(d.value)}
            </text>
          </g>
        ))}
      </g>
    </svg>
  );
}
```

## Related skills

For visual design standards, load:
- `Skill(ce:design)` - Core design principles
- `Skill(frontend-design:frontend-design)` - Distinctive frontend aesthetics
