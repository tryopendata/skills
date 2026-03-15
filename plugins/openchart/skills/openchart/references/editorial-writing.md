# Editorial Writing

How to write the text that frames a chart: titles, subtitles, source lines, and annotations. The text layer is what transforms a data display into a story.

## Titles

**Formula:** State the insight in conversational language. The title is the takeaway, not a description of the chart.

- Present tense for current state ("Austin is America's fastest-growing city")
- Past tense for historical events ("Remote work doubled after 2020")
- Make it specific and falsifiable, not vague

| Good | Bad | Why |
| --- | --- | --- |
| "Remote work doubled after 2020" | "Remote work percentage over time" | States the finding |
| "Big Tech roars back" | "FAANG stock performance" | Has editorial voice |
| "Austin is America's fastest-growing city" | "City population growth rates" | Has a protagonist |
| "Q4 sales exceeded targets by 18%" | "Bar chart of sales" | Specific and falsifiable |
| "Women now outearn men in 3 metro areas" | "Gender pay gap data" | Surprising, newsworthy |
| "Renewables overtook coal in 2024" | "Energy source comparison" | Captures the inflection point |

**The "one takeaway" test:** If the reader looks at this chart for 5 seconds and walks away, what's the one thing they remember? That should be the title. If the answer differs from what your title says, rewrite it.

## Subtitles

Carry what the title can't: units, time range, geographic scope, methodology notes.

- Format: sentence case, not title case. "Percentage of total workforce, 2015-2024"
- Keep it under one line when possible
- If the subtitle needs two lines, the title is probably carrying too little context
- **Avoid orphaned words.** If the subtitle wraps, make sure the last line has at least 3-4 words, not just "2018-2025" stranded alone. Restructure the sentence or use abbreviations (e.g., "LI" for "low-income") to keep the subtitle on one line. A short abbreviation key in the subtitle (e.g., `"LI = low-income students"`) is better than a two-line subtitle with a dangling fragment.

Examples:
- "Indexed to 100 at January 2020"
- "Share of total energy generation, gigawatt-hours"
- "Survey of 2,400 respondents, margin of error +/- 3%"
- "Chronic absence rate by district, 2018-2025 (LI = low-income students)"

## Source Lines

Always include. Always format as "Source: Organization Name".

| Do | Don't |
| --- | --- |
| Source: Bureau of Labor Statistics | Data from BLS |
| Source: Company earnings reports | Source: https://sec.gov/filings/... |
| Source: World Bank, OECD | Source: Various |

No URLs (they're not readable in a chart). No periods at the end. If multiple sources, separate with commas.

## Annotations

Amanda Cox, editor of the NYT Upshot: "The annotation layer is the most important thing we do."

### The Annotation Budget

Most charts need 0-3 annotations. If you need more, the chart is trying to tell too many stories. Split it into multiple charts or simplify the narrative.

### When to Annotate

| Annotate when... | Example |
| --- | --- |
| An outlier needs context | "Hurricane Katrina" on a damage cost spike |
| A trend changes direction | "Policy enacted" at the inflection point |
| Two series cross | "Overtook competitor in Q3" |
| A threshold is meaningful | Reference line at regulatory limit |
| The reader would ask "what happened here?" | Any visible anomaly that begs explanation |

### Annotation Copy Style

Short noun phrases or fragments, not full sentences.

| Good | Bad |
| --- | --- |
| "Pandemic lockdowns" | "This is when pandemic lockdowns began" |
| "Fed rate hike" | "The Federal Reserve raised interest rates" |
| "IPO" | "The company went public on this date" |

Use `\n` for multi-line annotations when context requires more than a few words, but keep each line to 4-8 words.

### Placement

- Place the note next to the data point it describes. Never make the reader hunt.
- Use connectors (lines from label to data point) when the label must be offset for space.
- Skip connectors when the label sits directly adjacent to the element.
- Range annotations (shaded regions) should be low opacity (0.1-0.2) and used for periods, not individual points.

## Where Does This Text Go?

| Information type | Placement |
| --- | --- |
| Units, time range, scope | Subtitle |
| Specific data point callout | Annotation |
| Methodology caveat | Footer |
| Data recency warning | Subtitle |
| Overall narrative framing | Title |
| "What happened here?" context | Annotation |
