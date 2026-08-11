# Gold Digest

A static, single-file HTML briefing on gold (XAU/USD) — spot price, weekly trend, key levels, and a 5-bullet "what moved gold this week" section. Styled for a financier audience (dark trading-desk aesthetic with a light-mode fallback).

Open [index.html](index.html) directly in a browser — no build step, no server, no dependencies.

## How daily updates work

All dynamic content lives in one place: the `<script id="digest-data" type="application/json">…</script>` block near the bottom of `index.html`. A daily routine only needs to overwrite the JSON inside that block — the HTML/CSS/JS around it never changes. **The block must contain valid JSON only — no HTML comments or trailing commas inside it**, since it's parsed with `JSON.parse`.

### Data schema

```jsonc
{
  "updated_at": "2026-08-11T08:00:00Z",   // ISO 8601 UTC timestamp shown in the header/footer
  "currency": "USD",
  "unit": "oz",
  "spot_price": 2415.30,                  // current XAU/USD spot
  "day_change": 6.20,                     // absolute 1-day change
  "day_change_pct": 0.26,
  "week_change": -14.80,                  // absolute 1-week change
  "week_change_pct": -0.61,
  "ytd_change_pct": 21.4,
  "week_high": 2441.90,
  "week_low": 2392.10,
  "sparkline": [2430.1, 2426.4, 2438.7, 2441.9, 2409.5, 2392.1, 2415.3], // ~7 daily closes, oldest -> newest
  "sparkline_labels": { "start": "Mon", "end": "Today" },
  "levels": [
    { "level": "Support 1", "price": 2392.00, "note": "short note" },
    { "level": "Resistance 1", "price": 2441.90, "note": "short note" }
    // any number of entries; first match of /support/i and /resistance/i
    // populate the stat-strip tiles, all entries populate the table
  ],
  "bullets": [
    // exactly 5 plain-text strings, rendered top to bottom as a single flat list
    // (no category labels/columns) so the section can be selected and pasted
    // straight into an email and still read as a clean bulleted list.
    // Each bullet should assess a driver of the gold price this week —
    // draw freely from macro/Fed policy, the dollar & real yields, geopolitics,
    // ETF/positioning flows, central-bank buying, etc. — mixed together in
    // whatever order best reflects what actually moved price, not grouped by topic.
    "...",
    "...",
    "...",
    "...",
    "..."
  ],
  "outlook": {
    "bias": "neutral",   // "bullish" | "bearish" | "neutral"
    "text": "1-3 sentence desk-style outlook."
  },
  "sources": [
    { "name": "Reuters — Commodities", "url": "https://..." }
  ]
}
```

### Updating it (for the automated routine)

1. Gather the day's data: spot price, day/week deltas, week high/low, YTD %, a ~7-point daily-close series for the sparkline, current support/resistance levels, exactly 5 plain-text bullets assessing what's driving the gold price this week (mixed across macro, Fed policy, the dollar & rates, geopolitics, flows, central-bank buying, etc. — not grouped by category), a short outlook, and source links.
2. Replace the entire contents of the `<script id="digest-data" type="application/json">` block in `index.html` with the new JSON payload.
3. Set `updated_at` to the current UTC timestamp.
4. Commit and push. If hosting on GitHub Pages / Netlify / Vercel from this repo, the push alone republishes the site.

If the page fails to parse the data block (e.g. malformed JSON), it shows a "data feed unavailable" banner and leaves the previous render's placeholders — check the browser console for the parse error.

## Design notes

- No external requests, fonts, or JS libraries — the whole page is self-contained (works from `file://`, a CDN, or any static host).
- Respects the visitor's OS light/dark preference (`prefers-color-scheme`), with an escape hatch (`data-theme` on `<html>`) if you add a manual toggle later.
- Sparkline is a hand-rolled inline SVG, colored green/red by week-over-week direction.
