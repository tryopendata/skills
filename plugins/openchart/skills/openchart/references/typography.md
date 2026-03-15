# Typography

Concrete sizing, weight, and formatting rules for chart text elements.

## Hierarchy

| Element | Size | Weight | Color | Purpose |
| --- | --- | --- | --- | --- |
| Title | 18-20px | Bold (700) | Primary (#222) | The takeaway |
| Subtitle | 14-16px | Regular-Semibold (400-600) | Secondary (gray) | Context |
| Axis labels | 12-14px | Regular (400) | Secondary (gray) | Scale reference |
| Data labels | 11-12px | Regular-Medium (400-500) | Primary | Precise values |
| Annotations | 11-14px | Regular (400) | Primary or accent | Editorial callouts |
| Source / byline | 11-12px | Regular (400) | Tertiary (light gray) | Attribution |

Use at least 2 clearly different hierarchy levels so the reader's eye knows where to start. The title should be noticeably larger and bolder than everything else. Supporting text steps down in size, weight, and contrast.

## Numerals

**Tabular (monospaced) numerals** for all data-facing text: axes, table columns, data labels, tooltips. Every digit occupies the same width, so columns of numbers align vertically and multi-digit values are instantly comparable by length.

**Lining figures** (uniform height, sitting on the baseline) rather than oldstyle figures (which vary in height like lowercase letters). Oldstyle figures look nice in paragraphs but are hard to scan in tables and axes.

## Font Selection

Sans-serif typefaces for all chart text. They render cleanly at small sizes and scan faster than serifs in data-dense contexts. Good options with tabular numeral support:

- Inter, Roboto, Source Sans Pro, Noto Sans
- System sans-serif stack as fallback

Serif typefaces can work for display titles if the publication style calls for it, but keep all other chart text in sans-serif.

Avoid: thin/light weights below 16px (low contrast, hard to read), condensed widths (normal-width text at a smaller size is more readable than bigger condensed text), decorative or script fonts.

## Formatting Rules

**Sentence case everywhere.** "Revenue by quarter" not "Revenue By Quarter" or "REVENUE BY QUARTER". Sentence case is faster to read because mixed-case word shapes aid recognition.

**Alignment:** left or right-aligned text creates clean edges that run parallel to chart elements. Center alignment only for chart titles. In tables, right-align numeric columns so decimal points line up.

**Minimum size:** never go below 10px. If text doesn't fit at 10px, the solution isn't smaller text. It's one of:
- Abbreviate labels (Jan, Feb vs January, February)
- Use direct labeling instead of a legend
- Restructure the chart (horizontal bar for long category names)
- Rotate labels (max 45 degrees, only for short labels)

**Line length for annotations:** 4-8 words per line. Use `\n` for multi-line annotations when needed.
