# Trader Performance vs Market Sentiment — Hyperliquid Analysis

Round-0 assignment: Data Science / Analytics Intern, Primetrade.ai

## What's in this repo

```
├── Trader_Sentiment_Analysis.ipynb   # Main notebook — fully executed, all charts embedded
├── notebook.py                       # Same notebook in plain-Python (jupytext) source form
├── analysis.py                       # Standalone script version of the same pipeline
├── data/
│   ├── historical_data__1_.csv       # Hyperliquid trade log (211,224 rows)
│   └── fear_greed_index__1_.csv      # Bitcoin Fear/Greed Index (2,644 rows)
├── outputs/                          # CSV/JSON exports of every intermediate table
│   ├── daily_account_metrics.csv     # Account x Day panel (core dataset for the analysis)
│   ├── account_segments.csv          # Account-level segment + archetype labels
│   ├── daily_with_segments.csv       # Daily panel joined with account segments
│   ├── sentiment_performance_summary.csv
│   ├── sentiment_behavior_summary.csv
│   └── log_part*.json / log_bonus_*.json   # Raw numbers behind every claim below
└── charts/                           # All 10 PNG charts (also embedded in the notebook)
```

## How to run

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn
python3 analysis.py          # regenerates outputs/ and charts/ from the raw CSVs
# or, to reproduce the notebook end-to-end:
pip install jupyter ipykernel nbclient jupytext
jupyter execute --output=Trader_Sentiment_Analysis.ipynb Trader_Sentiment_Analysis.ipynb
```

The notebook is already executed and saved with all outputs, so it can also just be
opened and read directly without re-running anything.

---

## Write-up

### Methodology

Trade-level Hyperliquid data (211,224 rows, 32 accounts, May 2023 – May 2025) was
timestamp-parsed and aggregated into an **Account x Day panel** (2,340 rows), then
joined to the daily **Fear/Greed Index**, collapsed from its 5-class scale
(Extreme Fear/Fear/Neutral/Greed/Extreme Greed) into 3 buckets (Fear/Neutral/Greed)
for the core comparisons. Win rate is computed only over trades that realize
non-zero `Closed PnL` (i.e. closing trades), not all trades — most opening trades
correctly show `0` PnL and would otherwise dilute the metric. A drawdown proxy is
the per-account running peak-to-trough of cumulative realized PnL. Trader segments
are tercile splits on average trade size and trade frequency, plus a median split on
%-profitable-days for consistency, with a complementary KMeans (k=3) clustering on 5
standardized behavioral features to surface natural "archetypes."

**Data quality:** both source files were clean — zero missing values and zero
duplicate rows in either the trade log or the sentiment index. The only cleaning
step needed was dropping 6 trades (out of 211,224) that fell outside the Fear/Greed
index's date coverage.

**A limitation worth stating up front:** the raw trade log has no `leverage` or
account-equity/margin field, even though the brief mentions leverage distribution as
an example metric. True leverage multiples cannot be computed from this data. We use
average trade size (USD) and total trading volume as an intensity proxy wherever
"leverage" would otherwise be analyzed, and flag it explicitly.

### Key insights

1. **Mean vs. median PnL tell different stories.** Mean daily PnL looks highest on
   Fear days, but that's driven by a handful of very large winning trades (PnL is
   heavily right-skewed). The **median** daily PnL and **% of profitable account-days**
   are both higher on **Greed** days (median ≈$265 vs ≈$123; 64.3% vs 60.4%
   profitable days), and the Fear-vs-Greed gap in daily PnL is only borderline
   significant (Mann-Whitney p≈0.06); the win-rate gap is not significant (p≈0.26).
   On a typical day, traders do modestly better in Greed regimes — a few outsized
   wins in Fear regimes pull the average up, but don't represent the typical outcome.

2. **Traders behave very differently under Fear vs Greed.** During Fear, accounts
   trade ~37% more often (105 vs 77 trades/day), in ~43% larger size ($8.5K vs
   $6.0K average trade), and roughly 2x the daily volume ($757K vs $352K) — with a
   much stronger long bias (long/short open ratio ≈2.2 vs ≈1.3), consistent with
   contrarian "buy-the-dip" behavior. The net long bias even flips slightly negative
   during Greed, suggesting some traders fade rallies rather than chase them.

3. **Segment matters more than sentiment alone.** Large-size and historically
   consistent traders earn their best days during **Fear** (large-size segment: ≈$9.8K
   mean daily PnL in Fear vs ≈$2.6K in Greed; consistent winners: ≈$6.5K vs ≈$3.0K),
   while **inconsistent** traders' best relative results cluster in **Greed**
   (≈$6.2K vs ≈$3.0K in Fear) — a pattern consistent with disciplined, larger
   traders capitalizing on fear-driven mispricing, while less disciplined traders
   get their best (and likely most fragile) results chasing momentum in euphoric
   markets.

*(Bonus, for context: a KMeans clustering surfaces three natural archetypes — a
5-account "Whale/Position Trader" group, a 9-account "High-Frequency Scalper" group,
and an 18-account "Retail/Casual" group. A simple Random Forest predicting next-day
profitability from sentiment + same-day behavior reaches 68% accuracy — below the
77% "always profitable" baseline on raw accuracy, but with much better recall on
losing days (48% vs 0%) after class-balancing; today's trade size, trade count and
PnL matter more than sentiment itself, so this is a first-pass signal, not a
production model.)*

### Strategy recommendations

**1. Scale position size selectively during Fear, but only for proven traders.**
Large-size and historically-consistent traders in this sample earn meaningfully more
on Fear days than Greed days. Rule of thumb: *on Fear/Extreme Fear days, allow
position sizing to scale up (within existing risk limits) for accounts with a >55%
historical win rate; leave sizing unchanged for newer or inconsistent accounts.*

**2. Treat high trade-frequency during Greed as a risk flag, not a strength.**
Inconsistent traders' best relative results cluster in Greed regimes, while
sample-wide median daily PnL is actually lower during the already-more-active Fear
regime — a sign that momentum-chasing in euphoric markets produces more variable,
less repeatable outcomes. Rule of thumb: *during Greed/Extreme Greed days, cap
trade-frequency increases for accounts in the "Inconsistent" segment (below-median
win rate/profitable-day rate), and require a confirmation step before adding to a
same-day position.*

### Limitations & caveats

- No leverage/equity field in the source data — leverage-specific claims from the
  brief are answered using trade size/volume as a stated proxy instead.
- Only 32 accounts, concentrated mostly in late 2024–April 2025 — segment-level
  numbers (especially the 5-account Whale archetype) should be read directionally,
  not as precise population statistics.
- The Mann-Whitney tests show the core Fear-vs-Greed performance gap is directional
  but not strongly statistically significant at this sample size — a larger, more
  balanced sample would sharpen the conclusions in insight #1.
