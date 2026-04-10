# Stock Market Reaction to CPI Announcements
An event-driven analysis of S&P 500 behavior around monthly CPI releases, combining econometric event study methodology with machine learning classifiers to detect and predict volatility spikes.
Data: 2,515 trading days · 2015–2024 · S&P 500 + FRED CPI + VIX
Methods: Event study (CAR) · Logistic Regression · Random Forest · GARCH(1,1) · SHAP

## Overview
Consumer Price Index announcements are among the most market-sensitive macro releases — they directly shape Fed rate expectations and, by extension, equity valuations. This project asks two questions:

1. Do CPI releases statistically change S&P 500 returns or volatility?
2. Can we predict whether the next day will see a volatility spike, given CPI-related features?

The analysis is structured as a reproducible Jupyter notebook covering data ingestion, feature engineering, statistical testing, event study methodology, regime analysis, and supervised ML with walk-forward validation.

## Project Structure
├── Stock_Market_CPI_Analysis_Improved.ipynb   # main notebook
├── README.md
└── data_raw_sp500_return.csv                  # exported S&P 500 returns 

## Methodology
1. Data Sources
SourceSeriesFrequencyYahoo Finance (yfinance)^GSPC S&P 500DailyFRED (pandas-datareader)CPIAUCSL — CPI All Urban ConsumersMonthlyFREDVIXCLS — CBOE Volatility IndexDaily
2. CPI Date Alignment
A critical methodological fix from the original analysis: FRED timestamps CPI data as the 1st of the reference month (e.g., 2023-01-01), but the BLS publishes the figure approximately 12 calendar days into the following month. Merging on the FRED date silently places the event flag on a non-trading day where it disappears.
Fix: Each monthly observation is shifted forward by a ~12-day calendar offset, then snapped to the next available trading day. For production use, replace with the exact BLS release calendar.
3. Feature Engineering
FeatureDescriptioncpi_releaseBinary flag: is today an aligned CPI release day?cpi_surpriseActual YoY CPI − naïve lag-1 forecast (proxy for market surprise)vol_5d5-day rolling return standard deviationvol_20d20-day rolling volatility (original feature)vol_60d60-day rolling volatilityreturn_lag1Previous day's log returnreturn_lag55-day lagged log returnVIXDaily VIX close (forward-filled for missing values)
4. Target Variable
vol_spike = 1 if next-day 20-day rolling volatility exceeds its own 60-day rolling mean.
Class balance: ~55% no-spike / ~45% spike.
5. Statistical Tests
- Welch's t-test on returns: event window vs. non-event days
- Levene's test on variance: tests whether CPI days exhibit different return dispersion
- Event study (CAR): Cumulative Abnormal Returns computed over a ±5 trading day window around each of ~120 release events, with 95% confidence intervals
6. Machine Learning
- Walk-forward cross-validation using TimeSeriesSplit(n_splits=5) — the correct approach for financial time series that prevents future data leaking into training folds.
- ModelCV ROC-AUCNotesLogistic Regressionreported per runBaseline, scaled features, class-balancedRandom Forestreported per run300 trees, max depth 6, class-balanced
- Final evaluation on a true holdout: last 20% of the time series (chronological, never seen during training).
7. GARCH(1,1)
Rolling standard deviation is a naïve volatility proxy. The arch library's GARCH(1,1) model captures volatility clustering — the empirically well-documented property that large return moves tend to cluster in time. Conditional volatility from GARCH is plotted against rolling std with CPI release markers for comparison.
8. SHAP Feature Importance
TreeExplainer SHAP values reveal which features drive the Random Forest's volatility spike predictions, including the direction of each feature's contribution (beeswarm plot).

## Key Findings
CPI release days show a higher average return (~0.13%) compared to non-event days (~0.04%), but the difference is not statistically significant (Welch t-test p ≈ 0.81)
The volatility spike classifier achieves ROC-AUC ~0.76 with logistic regression alone, suggesting that the feature set carries predictive signal even without the CPI surprise term
Regime analysis reveals different return and volatility profiles between low-inflation (2015–2020) and high-inflation (2021–2023) periods
The CAR event study shows whether pre-announcement drift or post-announcement momentum exists across the ±5 day window

## Limitations
CPI surprise is approximated using a naïve lag-1 YoY forecast. Real consensus estimates (Bloomberg, Cleveland Fed nowcast) would produce a more accurate surprise series and likely improve model performance significantly.
Daily frequency misses the within-day reaction. CPI is released at 8:30am ET before market open; the true shock absorption happens in the first 30–60 minutes of trading. Intraday data (Polygon.io, Yahoo Finance 1-min) would capture this.
Single index (S&P 500). Rate-sensitive sectors (utilities, REITs, financials) are expected to react more sharply and are not analyzed individually here.
No options data. Implied volatility (VIX term structure, at-the-money straddle prices) would give a forward-looking view of expected reaction size.


## Installation
bashpip install yfinance pandas-datareader scikit-learn arch shap matplotlib seaborn
All other dependencies (pandas, numpy, scipy) are part of a standard data science environment.

## Usage
Open the notebook and run all cells sequentially. Each section is self-contained with inline comments.
bashjupyter notebook Stock_Market_CPI_Analysis_Improved.ipynb
For Google Colab: upload the .ipynb file and uncomment the pip install cell at the top.

## Potential Extensions
A — CPI surprise with real consensus data
Fetch historical consensus forecasts from the Cleveland Fed or Bloomberg. The surprise component is theoretically the primary driver of market moves; improving this signal is the highest-ROI next step.
B — Intraday resolution
Use 5-minute bars around the 8:30am release time to measure immediate price discovery and how quickly the surprise is absorbed.
C — Asymmetric beat/miss reactions
Test whether CPI beats (actual > expected) and misses produce symmetric or asymmetric absolute returns — a well-documented phenomenon in equities.
D — Broader macro event comparison
Apply the same CAR event study to PCE, PPI, NFP, and FOMC decisions. Rank announcements by average absolute abnormal return to identify which releases matter most.
E — Sector-level analysis
Pull ETF data for XLU, XLRE, XLF, XLE and compare event-window behavior across sectors expected to have different rate sensitivities.
F — Walk-forward backtesting
Convert the vol-spike signal into a simple options strategy (e.g., buy straddles before predicted spike days) and evaluate P&L with realistic transaction costs.

## Dependencies
PackagePurposeyfinanceS&P 500 OHLCV datapandas-datareaderFRED API (CPI, VIX)scikit-learnLogistic Regression, Random Forest, TimeSeriesSplitarchGARCH(1,1) volatility modelshapModel interpretabilityscipyWelch t-test, Levene testmatplotlib / seabornVisualization

##Athor:
Ngan Tran
