# CPI Announcements and Stock Market Volatility

## Overview
This project investigates whether U.S. equity market volatility increases following Consumer Price Index (CPI) inflation announcements. Instead of predicting short-term stock returns, which are notoriously noisy, the project focuses on predicting **next-day volatility spikes**, a more stable and economically meaningful outcome.

## Research Question
Do CPI inflation announcements increase the probability of heightened market volatility in the S&P 500?

## Data Sources
- **S&P 500 daily price data**: Yahoo Finance (`^GSPC`)
- **CPI inflation data**: Federal Reserve Economic Data (FRED, CPIAUCSL)

## Methodology
1. Aligned CPI release dates with S&P 500 trading days using datetime index matching
2. Engineered an event-based CPI release dummy variable
3. Computed rolling volatility using a 20-day standard deviation of log returns
4. Defined a volatility spike as a next-day volatility exceeding its recent historical average
5. Trained a logistic regression model with time-series-aware train/test split
6. Evaluated performance using accuracy, ROC-AUC, and class-balanced metrics

## Model Features
- CPI release indicator (event-based)
- Current market volatility
- Daily log return

## Results
The model shows that CPI release days are associated with an increased probability of next-day volatility spikes, particularly during periods of elevated baseline volatility. While predictive performance is modest, results are consistent with financial market efficiency and existing macro-finance literature.

## Limitations
- CPI expectation (forecast) data was unavailable, preventing true CPI surprise modeling
- Daily data does not capture intraday announcement effects
- Linear models may not capture nonlinear macro-market interactions

## Future Work
- Incorporate consensus CPI forecasts to model CPI surprise
- Extend analysis to intraday volatility
- Evaluate nonlinear models such as tree-based classifiers
- Explore interactions between macro events and market regimes

## Tools Used
- Python
- pandas, numpy
- scikit-learn
- yfinance
- matplotlib

## Author
Linh Ngan
