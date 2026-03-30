# Format Strings

Both `axis.format` and `labels.format` accept [d3-format](https://d3js.org/d3-format) strings, plus a literal suffix extension.

## Literal Suffix Pattern

Append non-format characters after a valid d3-format specifier. The engine tries d3-format first; if it fails, it splits off the trailing suffix and applies d3-format to the rest.

## Full Format Table

| Format | Input | Output | Use case |
| --- | --- | --- | --- |
| `".1f"` | 12.5 | `12.5` | One decimal, no units |
| `".1f%"` | 12.5 | `12.5%` | Percentage (data already in %, not 0-1) |
| `".0f%"` | 12 | `12%` | Whole-number percentage |
| `"$,.0f"` | 1234 | `$1,234` | Currency with commas |
| `"$,.2f"` | 3.75 | `$3.75` | Currency with cents |
| `".1%"` | 0.125 | `12.5%` | d3 native percent (multiplies by 100) |
| `"~s"` | 10000 | `10k` | SI suffix (k, M, G) for large numbers |
| `"~s"` | 1500000 | `1.5M` | SI suffix, auto-scales |
| `"$~s"` | 78000 | `$78k` | Currency with SI suffix. **Caution:** non-round thousands show decimals (60420 -> $60.42k). Round data values to clean thousands or use "$,.0f" for exact values |
| `",.0f"` | 132979 | `132,979` | Comma-separated, no decimals |

## Percentage Gotcha

**When data is already in percentage form** (e.g., `12.5` meaning 12.5%), use `".1f%"` not `".1%"`. The d3 `%` type multiplies by 100, so `0.125` becomes `12.5%` but `12.5` becomes `1,250.0%`.

## Tick Density

Format strings control _what_ ticks look like. `tickCount` controls _how many_ appear. Always pair them:

```json
"axis": { "format": ".0f%", "tickCount": 5 }
```

Without `tickCount`, many charts default to showing only the domain endpoints (e.g., `0%` and `50%`). This makes it nearly impossible for readers to identify specific data point values. Use 5-6 ticks as a default for most charts. The engine treats `tickCount` as a suggestion and picks clean round values nearby.

## Axis Title Redundancy

When the format string already includes units (`%`, `$`, `k`), drop the units from `axis.title`. Write `"Chronic absence rate"` with `format: ".0f%"`, not `"Chronic absence rate (%)"`. The ticks themselves show `10%`, `20%`, `30%`, which makes `(%)` in the title redundant.
