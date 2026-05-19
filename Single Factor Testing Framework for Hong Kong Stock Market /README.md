# Cross-sectional Single Factor Testing Framework for Hong Kong Stock Market
This paper develops a full-process quantitative single-factor research and backtesting framework for the Hong Kong stock market. It focuses on constructing momentum, turnover and skewness factors. By means of cross-sectional preprocessing, market capitalization neutralization, industry neutralization and WLS regression, we eliminate style bias impacts and extract pure alpha signals. This study further adopts IC and IR tests, grouped backtests and Fama-MacBeth regression to systematically verify the predictive ability and statistical significance of these factors in Hong Kong equities, which provides empirical support for the formulation of robust multi-factor stock selection strategies.

## Table of Contents

1. Data Ingestion
2. Factor Construction  
2.1 Skewness Factor  
2.2 Momentum Factor  
2.3 Turnover Factor  
3. Data Preprocessing & Cleaning
4. Factor Neutralization  
4.1 The Unified Neutralization Model  
4.2 Industry (or Sector) Neutralization  
4.3 Size Neutralization  
4.4 Final Standardization  
5. Single-Factor Test  
5.1 IC and RankIC  
5.2 IR  
5.3 Stratified Backtesting  
5.4 Factor Assessment Approach  
5.5 Testing Results  
6. Multi-Factor Correlation Analysis
7. Fama-MacBeth Regression Analysis

## 1. Data Ingestion
Data used in this report is imported via the yfinance API. We select daily OHLCV data and corporate metadata of 261 stocks across 7 industries, covering the period from January 1, 2022 to December 31, 2024.

Collected data fields includes:
Dynamic data: date, ticker, open, high, low, close, volume, stock name, turnover, 
Static metadata: market capitalization, sector, industry

## 2. Factor Construction
### 2.1 Skewness Factor
The skewness factor is calculated as the sample skewness of individual stock daily returns over the past 20 trading days. In financial markets, skewness captures shifts in recent market sentiment and asymmetric risks embedded in stock price movements. A positive skewness indicates a fatter right tail in return distribution that leans toward higher gains, while negative skewness points to a thicker left tail associated with downside losses.  
$$
S = \frac{n}{(n-1)(n-2)} \sum \left( \frac{X_i - \bar{X}}{\sigma} \right)^3
$$
