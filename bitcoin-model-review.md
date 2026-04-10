# Bitcoin Model Review

Date: 2026-04-08

## Goal

Tighten the Bitcoin astrology path so it uses the broader windows implied by the book instead of over-indexing on narrow daily clusters.

## Inputs Reviewed

- `book_text/Raghavan, Maya - Bitcoin and Astrology (clean text).txt`
- `data/bitcoin_ephemeris_2026.json`
- `data/bitcoin_backtest_20y.json`
- Local BTC market history from `/api/market-snapshot?days=120`

## Findings

### 1. The book supports broader windows

The book is not just a day-trading trigger list. It emphasizes:

- Rahu regimes
- Sun ingress windows
- Mercury retrograde danger periods
- Moon/nakshatra triggers
- broader phase transitions before narrow entries

That means the runtime should expose both:

- macro windows for the regime call
- daily rows for entry refinement

### 2. Strict Bitcoin backtest is too small

From `data/bitcoin_backtest_20y.json`:

- evaluated windows: `4`
- win rate: `50.0%`

This is too small to justify aggressive claims.

### 3. Broader supportive windows are performing better

Using the regenerated 2026 Bitcoin ephemeris against the recent BTC series:

- `score >= 4.5`
  - sample: `3`
  - 3-day win rate: `66.7%`
  - 5-day win rate: `66.7%`
  - average 5-day return: `+2.27%`
- `score >= 3.5`
  - sample: `12`
  - 3-day win rate: `58.3%`
  - 5-day win rate: `66.7%`
  - average 5-day return: `+2.08%`

### 4. Best observed broader interval in the cache

- `2026-07-17` to `2026-08-16`
- label: `Rahu in Aquarius + Sun in Cancer`
- average score: `5.13`

### 5. Best realized macro window in current overlapping market history

- `2026-01-01` to `2026-01-13`
- label: `Rahu in Aquarius + Sun in Sagittarius`
- realized return: `+7.43%`

## PCA / Pattern Notes

The first principal component is dominated by the overall score. The next components separate:

- Saturn-to-natal-Jupiter pressure
- Jupiter-to-natal-Ketu support
- Sun/Rahu cluster effects

This supports using:

- a deterministic regime score first
- then specific reason clusters for explanation

instead of purely subjective commentary.

## Recommendation

Use the Bitcoin macro windows as the primary timing layer for multi-week questions such as:

- next timing window
- best upcoming window
- next 1-2 months

Then use the daily rows only to refine the entry inside the active macro regime.
