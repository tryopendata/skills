# Typography and Labels

Font selection, sizing, and label placement for publication-quality data visualization.

## Contents

- Font recommendations
- Size hierarchy
- Number formatting
- Label placement strategies
- Annotation design
- Accessibility

## Font recommendations

### Primary choices

| Font | Type | Best For | Notes |
|------|------|----------|-------|
| **Inter** | Sans-serif | All-purpose | Excellent tabular figures |
| **Source Sans Pro** | Sans-serif | Detailed charts | Very readable at small sizes |
| **Roboto** | Sans-serif | Generic fallback | Ubiquitous, safe |
| **IBM Plex Sans** | Sans-serif | Technical data | Clean, professional |

### Avoid

- Serif fonts for data labels (less readable at small sizes)
- Decorative fonts anywhere
- Mixing more than 2 font families

### Font stack

```css
font-family: Inter, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
             "Helvetica Neue", Arial, sans-serif;
```

### Loading Inter

```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;800&display=swap" rel="stylesheet">
```

## Size hierarchy

| Element | Size | Weight | Tracking | Use |
|---------|------|--------|----------|-----|
| Headline | 28-36px | 800 | -0.02em | Main message |
| Subtitle | 16-20px | 400 | 0 | Context/methodology |
| Data labels | 12-14px | 600 | 0 | Values on chart |
| Axis labels | 11-13px | 400-500 | 0 | Scale indicators |
| Source | 10-12px | 400 | 0 | Attribution |

## OpenData typography scale (by export format)

| Element | Twitter (1200x675) | Instagram (1080x1080) | Weight | Line Height |
|---------|-------------------|----------------------|--------|-------------|
| Headline | 48px | 56px | 800 | 1.1 |
| Subtitle | 20px | 22px | 400 | 1.4 |
| Data labels | 16px | 18px | 600 | 1.2 |
| Axis labels | 14px | 16px | 500 | 1.2 |
| Source | 12px | 14px | 400 | 1.3 |

### Headline rules

```typescript
interface HeadlineConfig {
  maxChars: 60;           // Auto-scale down if longer
  casing: "sentence";     // Not title case
  punctuation: "none";    // No period at end
  maxWords: 10;
}

// Formula: [What happened] + [How much/when/who]
const goodHeadlines = [
  "Revenue doubled in two years",
  "Netflix spent $82B on content since 2020",
  "Gen Z saves 50% more than millennials did at same age",
];
```

### Auto-scaling headlines

```typescript
function getHeadlineSize(text: string, baseSize: number): number {
  const charCount = text.length;
  if (charCount <= 30) return baseSize;
  if (charCount <= 45) return baseSize * 0.85;
  if (charCount <= 60) return baseSize * 0.7;
  return baseSize * 0.6;  // Force wrap for very long headlines
}
```

### Example styles

```typescript
const textStyles = {
  headline: {
    fontSize: 32,
    fontWeight: 800,
    letterSpacing: "-0.02em",
  },
  subtitle: {
    fontSize: 16,
    fontWeight: 400,
  },
  dataLabel: {
    fontSize: 14,
    fontWeight: 600,
    fontFeatureSettings: '"tnum"',  // Tabular numbers
  },
  axisLabel: {
    fontSize: 12,
    fontWeight: 500,
  },
  source: {
    fontSize: 11,
    fontWeight: 400,
    opacity: 0.7,
  },
};
```

## Number formatting

### Tabular numbers

Always use tabular figures for numeric labels so they align:

```typescript
<text style={{ fontFeatureSettings: '"tnum"' }}>
  {value.toLocaleString()}
</text>
```

### Abbreviation rules

| Range | Format | Example |
|-------|--------|---------|
| < 1,000 | Full | 847 |
| 1,000 - 999,999 | Thousands | 1.2K, 847K |
| 1M - 999M | Millions | 1.5M, 847M |
| 1B+ | Billions | 1.5B, 2.3B |

```typescript
function formatCompact(value: number): string {
  const abs = Math.abs(value);
  if (abs >= 1e9) return (value / 1e9).toFixed(1) + "B";
  if (abs >= 1e6) return (value / 1e6).toFixed(1) + "M";
  if (abs >= 1e3) return (value / 1e3).toFixed(1) + "K";
  return value.toLocaleString();
}
```

### Currency

```typescript
function formatCurrency(value: number, currency = "USD"): string {
  if (Math.abs(value) >= 1e9) return "$" + (value / 1e9).toFixed(1) + "B";
  if (Math.abs(value) >= 1e6) return "$" + (value / 1e6).toFixed(1) + "M";
  return new Intl.NumberFormat("en-US", {
    style: "currency",
    currency,
    maximumFractionDigits: 0,
  }).format(value);
}
```

### Percentages

```typescript
function formatPercent(value: number, decimals = 1): string {
  return (value * 100).toFixed(decimals) + "%";
}
```

## Label placement strategies

### Direct labeling (preferred)

Place labels directly on or near the data they describe. Avoid separate legends.

**Line charts:**
```typescript
// Label at the end of each line
<text
  x={xScale(lastPoint.x) + 8}
  y={yScale(lastPoint.y)}
  dominantBaseline="middle"
>
  {seriesName}
</text>
```

**Bar charts:**
```typescript
// Value above/beside the bar
const labelInside = barHeight > 24;

<text
  x={barX + barWidth / 2}
  y={labelInside ? barY + 16 : barY - 8}
  fill={labelInside ? palette.background : palette.text}
  textAnchor="middle"
>
  {formatCompact(value)}
</text>
```

### Sparse labeling

For infographics, don't label everything. Label:
- First and last points
- Maximum and minimum (if notable)
- Inflection points (where trend changes)

```typescript
const keyPoints = [
  { point: data[0], anchor: "start" },
  { point: data[data.length - 1], anchor: "end" },
];

// Add max if not at edges
const maxPoint = data.reduce((max, d) => d.y > max.y ? d : max);
if (maxPoint !== data[0] && maxPoint !== data[data.length - 1]) {
  keyPoints.push({ point: maxPoint, anchor: "middle" });
}
```

### Avoiding collision

**Simple alternation:**
```typescript
// Alternate above/below
const yOffset = i % 2 === 0 ? -12 : 18;
```

**Collision detection:**
```typescript
// Check if labels overlap
function labelsOverlap(a: Rect, b: Rect): boolean {
  return !(
    a.x + a.width < b.x ||
    b.x + b.width < a.x ||
    a.y + a.height < b.y ||
    b.y + b.height < a.y
  );
}

// Simple greedy placement
const placedLabels: Rect[] = [];
labels.forEach(label => {
  let yOffset = -12;
  let attempts = 0;
  while (attempts < 4) {
    const rect = { x: label.x, y: label.y + yOffset, width: 60, height: 16 };
    if (!placedLabels.some(p => labelsOverlap(rect, p))) {
      placedLabels.push(rect);
      label.yOffset = yOffset;
      break;
    }
    yOffset = yOffset < 0 ? -yOffset : -yOffset - 12;  // Try other side
    attempts++;
  }
});
```

## Annotation design

### Hierarchy

| Level | Visual Treatment | Count | Example |
|-------|------------------|-------|---------|
| Callout | Large, highlight, leader line | 1 | "Peak: $82B in Q3 2023" |
| Label | Medium, inline | 2-3 | "+42% YoY" |
| Note | Small, footer | 1 | "Excludes acquisitions" |

### Callout implementation

```typescript
interface Callout {
  x: number;
  y: number;
  text: string;
  subtext?: string;
}

function CalloutAnnotation({ callout, palette }: { callout: Callout; palette: ChartPalette }) {
  const labelX = callout.x + 40;
  const labelY = callout.y - 20;

  return (
    <g>
      {/* Highlight circle */}
      <circle
        cx={callout.x}
        cy={callout.y}
        r={8}
        fill={palette.primary}
        fillOpacity={0.2}
        stroke={palette.primary}
        strokeWidth={2}
      />

      {/* Leader line */}
      <line
        x1={callout.x + 8}
        y1={callout.y - 8}
        x2={labelX - 4}
        y2={labelY + 8}
        stroke={palette.textMuted}
        strokeWidth={1}
      />

      {/* Label */}
      <text
        x={labelX}
        y={labelY}
        fill={palette.text}
        fontSize={14}
        fontWeight={600}
      >
        {callout.text}
      </text>
      {callout.subtext && (
        <text
          x={labelX}
          y={labelY + 16}
          fill={palette.textMuted}
          fontSize={12}
        >
          {callout.subtext}
        </text>
      )}
    </g>
  );
}
```

### Leader lines

When labels must be positioned away from their data:

```typescript
<polyline
  points={[
    [dataX, dataY],                    // Start at data
    [dataX + 10, dataY - 10],          // Short offset
    [labelX - 4, labelY],              // End near label
  ].map(p => p.join(",")).join(" ")}
  fill="none"
  stroke={palette.textMuted}
  strokeWidth={1}
/>
```

## Accessibility

### Minimum sizes

| Element | Minimum | Recommended |
|---------|---------|-------------|
| Body text | 12px | 14px |
| Labels | 11px | 12px |
| Small print | 10px | 11px |

### Contrast ratios

- Normal text: 4.5:1 minimum
- Large text (>18px): 3:1 minimum
- Use dark gray (#1A1A1A or #333) on white, not pure black

### ARIA labels

```typescript
<svg
  aria-label={`${chartType} showing ${headline || description}`}
  role="img"
>
  <title>{headline}</title>
  <desc>{description}</desc>
  {/* chart content */}
</svg>
```

### Screen reader text

For complex charts, provide text alternatives:

```typescript
<g className="sr-only" aria-hidden="false">
  <text>
    Chart data: {data.map(d => `${d.label}: ${d.value}`).join(", ")}
  </text>
</g>
```

## Text wrapping

SVG doesn't auto-wrap text. For long headlines:

```typescript
function wrapText(text: string, maxWidth: number, fontSize: number): string[] {
  const words = text.split(" ");
  const lines: string[] = [];
  let currentLine = "";

  // Rough estimate: ~0.6 * fontSize per character
  const charsPerLine = Math.floor(maxWidth / (fontSize * 0.6));

  words.forEach(word => {
    if ((currentLine + " " + word).trim().length <= charsPerLine) {
      currentLine = (currentLine + " " + word).trim();
    } else {
      if (currentLine) lines.push(currentLine);
      currentLine = word;
    }
  });
  if (currentLine) lines.push(currentLine);

  return lines;
}

// Usage
const headlineLines = wrapText(headline, innerWidth, 32);

{headlineLines.map((line, i) => (
  <text key={i} x={MARGIN.left} y={36 + i * 38} fontSize={32} fontWeight={800}>
    {line}
  </text>
))}
```
