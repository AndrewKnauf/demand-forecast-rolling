# 30ML Production Forecast Methodology — 2026-05-08 Locked Run

Purpose: immediate production-manager handoff for the 30ML May-July 2026 ordering horizon.

## Source files locked in this run

1. Monthly_BR_Cleaned_Combined_production_05012026.xlsx
   - Used as the canonical Amazon Business Reports monthly actual layer.
   - April 2026 is treated as a closed actual month, not a trend or nowcast.
2. Amazon_FBA_Inventory_Report_05082026.csv
   - Used for T30/T90 trend sanity checks, stock coverage, and inventory risk flags.
   - It is not used as the primary demand source.
3. all_asin_prorated_monthly_ads_calendar_production.xlsx
   - Used as an ad-state flag only.
   - The ads file ends at March 2026 for the 30ML rows, so it is not used as an April or May driver.
4. Amazon_BR_Apr_01-20_2026.xlsx
   - Not used for April demand in this locked run because April actuals are available in the monthly BR file.

## Current GPT54 logic carried forward

- Corrected demand is computed before seasonality.
- Seasonality uses reliability-weighted 2024 and 2025 history.
- 30ML grouping uses family, volume bucket, stability bucket, and migration risk.
- Low-confidence SKUs are stabilized with a peer-paced May-July shape.
- Inventory T30/T90 is a near-term check, not the forecast engine.
- Ads are flags only, not direct multipliers.

## Stepwise formula logic

### Step 1 — Monthly actuals

Observed_Units_ASIN_Month = Amazon Business Reports Units Ordered

FBA_pct = FBA_units / (FBA_units + FBM_units)

BuyBox_pct = monthly weighted Buy Box percentage from the cleaned monthly BR file.

### Step 2 — Month reliability score

Each ASIN-month starts at 100. Penalties are applied for known demand-censoring signals:

- Buy Box below 95 percent.
- FBA share below 95 percent.
- Zero units with sessions.
- Missing/zero month.
- Ads low versus 2025 median is a minor flag only.

Important exception: January-March 2024 Buy Box values are zero across the file, so they are treated as missing and not penalized.

### Step 3 — Corrected monthly demand

Corrected_Units = Observed_Units when the month is clean.

If FBA share or Buy Box indicates suppression, a conservative correction multiplier is applied with severity caps:

- Severe suppression cap: 1.65x
- Medium suppression cap: 1.40x
- Mild suppression cap: 1.22x
- Minor suppression cap: 1.10x

This correction layer is conservative because the current locked run is monthly-first and does not use daily OOS detail as the primary engine.

### Step 4 — Seasonality

For each ASIN, Apr-Dec month shares are calculated from corrected 2024 and 2025 demand.

Year-month weights are based on reliability, with a modest recency preference to 2025 when reliable.

ASIN final month share:

Final_Share = alpha * ASIN_Share + (1 - alpha) * Peer_Pool_Share

Alpha is higher for high-volume stable ASINs and lower for low-volume, sparse, unstable, or low-confidence ASINs.

### Step 5 — Growth factor

YTD_Growth = Corrected Jan-Apr 2026 / Corrected Jan-Apr 2025

Historical_Growth = Corrected Apr-Dec 2025 / Corrected Apr-Dec 2024

Final_Growth is a shrunk blend of ASIN YTD growth, ASIN historical growth, peer growth, and a neutral anchor. Caps prevent unstable SKUs from overreacting.

### Step 6 — April actual replacement

April 2026 is forced to the actual monthly BR value:

Apr26_Final = Apr26_Actual

April actual versus the model's pre-actual April value is used only to softly re-anchor May-July. It does not use the April 1-20 trend file.

### Step 7 — Inventory trend check

Inventory_Pace = 0.65 * T30 + 0.35 * (T90 / 3)

T30/T90 affects May-July only as a sanity check blend and inventory-risk flag.

### Step 8 — Low-confidence peer-paced override

If Confidence < 60:

May-July is blended from:

- Base GPT54-style corrected history forecast
- April actual multiplied by peer Apr-to-month ratios
- Inventory T30/T90 pace check

This is the production ordering stabilization rule, not a claim that the base benchmark has permanently changed.

## Run totals

ASINs forecast: 152
April 2026 actual total: 14975
May forecast total: 16044
June forecast total: 12559
July forecast total: 13046
May-July total: 41649
Low-confidence ASINs: 37
Critical coverage flags: 7
Low coverage flags: 78
