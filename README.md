# Gold Digest

A static, single-file HTML briefing on gold (XAU/USD) for a financier audience. Every section is shown in **English with the Greek version directly underneath**, and the whole page is built to be copied straight into an email.

Open [index.html](index.html) in a browser. There is no build step, no server and no dependencies.

## Copying into an email

- **Copy for email / Αντιγραφή για email** at the top copies the whole briefing.
- **Copy section / Αντιγραφή ενότητας** under each section copies just that section.
- Both buttons put rich HTML and a plain-text version on the clipboard. Rich-text editors such as Gmail, Outlook and Apple Mail get the formatted version, and plain-text fields get clean text with both languages.
- Manual selection with Ctrl+C or Cmd+C also works, because what you see on screen is the same inline-styled markup the buttons copy.

The page is built to survive pasting:

- All content is rendered with inline styles and simple tables. Email clients drop stylesheets, flexbox and grid, so none of the layout depends on them.
- The page uses a light theme only. Dark backgrounds carry over badly when pasted into an email.
- Bullets are real HTML lists, so they paste as a bulleted list.
- The price chart is drawn as a PNG image, since email clients do not show SVG. The same prices are also written out as text below the chart in case an email client strips the image.
- Greek text carries `lang="el"` for screen readers and spell-checkers.

## How updates work

All dynamic content lives in one place: the `<script id="digest-data" type="application/json">…</script>` block near the bottom of `index.html`. An update only needs to overwrite the JSON inside that block. The HTML, CSS and JS around it do not change.

**The block must contain valid JSON only.** No comments, no trailing commas, no NaN or Infinity, because it is parsed with `JSON.parse`.

### Data schema

Every text field has an English version and a Greek version with the `_el` suffix. The Greek fields are optional for the renderer, but the update routine should always fill them. When a Greek field is missing, that line is simply not shown.

```jsonc
{
  "updated_at": "2026-10-11T16:05:00Z",   // ISO 8601 UTC; shown in UTC and Central European time
  "currency": "USD",
  "unit": "oz",
  "spot_price": 4193.83,                  // XAU/USD spot
  "spot_price_eur": 3745.49,              // XAU/EUR spot (optional; shown under the USD price)
  "eurusd_used": 1.1197,                  // EUR/USD rate used for the conversion (optional, not displayed)
  "day_change": 61.06,                    // absolute 1-day change
  "day_change_pct": 1.48,
  "week_change": 53.83,                   // absolute 1-week change
  "week_change_pct": 1.30,
  "ytd_change_pct": -2.80,
  "week_high": 4196.00,
  "week_low": 4066.06,
  "sparkline": [4140.0, 4156.3, 4169.68, 4086.72, 4118.63, 4174.78, 4193.83], // ~7 daily prices, oldest to newest
  "sparkline_labels": { "start": "Fri 2 Oct", "end": "Fri 9 Oct" },

  "price_notes": {                        // optional: how the numbers were sourced or derived
    "en": "…",
    "el": "…"
  },

  "levels_rationale": {                   // optional: why these levels were chosen
    "en": "…",
    "el": "…"
  },
  "levels": [
    // Any number of entries, any order. The nearest "Support…" level below spot and the
    // nearest "Resistance…" level above spot feed the key-figures table. All entries
    // appear in the levels table. Greek level names are derived automatically
    // (Support = Στήριξη, Resistance = Αντίσταση) unless "level_el" is given.
    { "level": "Support 1", "price": 4100.00, "note": "why it matters", "note_el": "γιατί έχει σημασία" },
    { "level": "Resistance 1", "price": 4226.00, "note": "…", "note_el": "…" }
  ],

  "bullets": [
    // 3-5 items; the first 5 are shown as a bulleted list, in order.
    // Each bullet assesses one driver of this week's price (Fed policy, dollar & rates,
    // geopolitics, ETF flows, central-bank buying, and so on).
    // Plain strings are still accepted (English only, no label).
    {
      "category": "Fed Policy",
      "category_el": "Νομισματική πολιτική Fed",
      "text": "1-2 terse sentences in a trading-desk tone.",
      "text_el": "Η ίδια πληροφορία στα ελληνικά, σε ύφος οικονομικού τύπου."
    }
  ],

  "outlook": {
    "bias": "neutral",                    // "bullish" | "bearish" | "neutral"
    "text": "2-4 sentence hedged outlook with key levels and the next event to watch.",
    "text_el": "Η ίδια προοπτική στα ελληνικά."
  },

  "sources": [
    // Only http(s) links are shown.
    { "name": "CNBC — Gold / US Dollar Spot (XAU=)", "url": "https://www.cnbc.com/quotes/XAU=" }
  ]
}
```

### Update steps (for the automated routine)

1. Gather the data: spot price in USD and EUR, day and week changes, week high and low, YTD change, about 7 recent daily prices, support and resistance levels, 3-5 driver bullets, a short outlook, and source links.
2. Write every text field in English and Greek (`category_el`, `text_el`, `note_el`, `outlook.text_el`, `levels_rationale.el`, `price_notes.el`).
3. Replace the entire contents of the `<script id="digest-data" type="application/json">` block in `index.html` with the new JSON.
4. Set `updated_at` to the current UTC timestamp.
5. Check that the block parses:

   ```sh
   node -e "JSON.parse(require('fs').readFileSync('index.html','utf8').match(/<script id=\"digest-data\" type=\"application\/json\">([\s\S]*?)<\/script>/)[1])"
   ```

6. Commit and push. When the site is hosted on GitHub Pages from this repo, the push republishes it.

If the data block fails to parse, the page shows a "data feed unavailable" banner. Check the browser console for the parse error.

## Design notes

- The page makes no external requests and loads no fonts or libraries. It works from `file://`, a CDN or any static host.
- The page chrome (background, buttons, card) uses a small stylesheet. Everything inside the briefing is styled inline so it keeps its look when pasted.
- The chart is drawn on a canvas and turned into a PNG. It is green when the period is up and red when it is down.
- Printing hides the copy buttons and prints the briefing on a white page.
