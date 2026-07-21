# Trader Performance vs Market Sentiment - Hyperliquid Analysis

**Round-0 Assignment – Data Science / Analytics Intern**  
**Company:** Primetrade.ai

---

# Project Structure

```text
├── Trader_Sentiment_Analysis.ipynb   # Main notebook with analysis and charts
├── data/
│   ├── historical_data.csv
│   └── fear_greed_index.csv
├── outputs/
│   ├── daily_account_metrics.csv
│   ├── account_segments.csv
│   ├── daily_with_segments.csv
│   ├── sentiment_performance_summary.csv
│   └── sentiment_behavior_summary.csv
├── charts/
│   └── All generated charts
├── analysis.py
├── README.md
└── requirements.txt
```

---

# How to Run

Install the required libraries:

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn
```

Run the analysis:

```bash
python analysis.py
```

Or open the notebook:

```bash
jupyter notebook Trader_Sentiment_Analysis.ipynb
```

---

# Methodology

Two datasets were used for this analysis:

1. **Hyperliquid Historical Trader Data**
2. **Bitcoin Fear & Greed Index**

### Data Preparation

- Loaded both datasets using Pandas.
- Checked for missing values and duplicate records.
- Converted timestamps into daily dates.
- Merged both datasets using the Date column.
- Removed only 6 trades that were outside the sentiment dataset date range.

The trade data contains **211,224 trades** from **32 traders** between **May 2023 and May 2025**.

Daily trader metrics were created, including:

- Daily Profit & Loss (PnL)
- Win Rate
- Average Trade Size
- Number of Trades
- Trading Volume
- Long vs Short Ratio

Since the dataset does not contain actual leverage values, average trade size and trading volume were used as a proxy for trading intensity.

---

# Key Insights

## 1. Trader Performance Changes with Market Sentiment

Although the average PnL appears higher during **Fear** days because of a few very large winning trades, the **median daily PnL** is higher during **Greed** days.

- Traders had more profitable days during Greed.
- The difference between Fear and Greed is noticeable but not strongly statistically significant.

**Conclusion:** Most traders perform slightly better during Greed markets, while Fear markets occasionally produce very large profits.

---

## 2. Trader Behavior Changes During Fear

During Fear markets, traders become much more active.

Compared to Greed days:

- Around **37% more trades** are placed.
- Average trade size is **about 43% larger**.
- Daily trading volume is almost **2 times higher**.
- Traders open more **Long positions**, showing a "buy the dip" strategy.

**Conclusion:** Traders take more aggressive positions when the market is fearful.

---

## 3. Experienced Traders Perform Better During Fear

Trader performance depends on experience and trading style.

- Large-volume traders earn their highest profits during Fear markets.
- Consistent profitable traders also perform better during Fear.
- Less consistent traders perform relatively better during Greed markets.

**Conclusion:** Skilled traders use market fear as an opportunity, while less experienced traders perform better when markets are optimistic.

---

# Strategy Recommendations

## Strategy 1

Increase position size only for experienced and consistently profitable traders during **Fear** markets.

This allows skilled traders to take advantage of market opportunities while keeping risk under control.

---

## Strategy 2

Limit excessive trading during **Greed** markets, especially for inconsistent traders.

Adding trade limits or requiring extra confirmation before entering new positions can help reduce unnecessary losses.

---

# Limitations

- The dataset does not include actual leverage information.
- Only 32 trader accounts are available, so results may not represent all traders.
- Some statistical tests show trends rather than strong significance, meaning a larger dataset could provide more reliable conclusions.

---

# Conclusion

This analysis shows that market sentiment influences both trader behavior and performance.

- Fear markets lead to higher trading activity, larger trade sizes, and stronger long positions.
- Greed markets produce better typical daily profitability.
- Experienced traders benefit more from Fear markets, while less consistent traders perform better during Greed.

These findings can help design trading strategies based on market sentiment and trader profiles.
