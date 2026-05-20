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
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$S = \frac{n}{(n-1)(n-2)} \sum \left( \frac{X_i - \bar{X}}{\sigma} \right)^3$$    
Where:  
$n$: the lookback window, which is set to 20 trading days in this study.  
$X_i$: daily returns of individual stock on trading day $i$  
$\bar{X}$: sample mean of daily returns within the rolling window  
$\sigma$: sample standard deviation of daily returns within the rolling window  
$$\sum{(\frac{\ X_i\ -\ \bar{X}\ }{\sigma})}^3$$: sum of cubed standardized returns, which measures the skewness of return distribution.

### 2.2 Momentum Factor
The momentum factor reflects the speed and direction of asset price movements. Technically, it measures the magnitude of price fluctuations over a specific period. The factor takes a positive value amid upward price trends and turns negative when prices decline.

It is worth noting that momentum strategies tend to underperform under extreme market conditions such as market crashes and black swan events. Accordingly, integrating other factors or strategies is essential to optimize and strengthen its overall performance.  
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$Momentum\ =\ \frac{P_{t-1}}{P_{t-n}}\ -\ 1$$  
Where:  
$P_t$: the closing price on day t-1.<br>
$P_t-n$: the closing price recorded $n$ trading days prior.  
In this study, we set $n$ = 20, representing the closing price 20 trading days earlier (approximately one calendar month)

### 2.3 Turnover Factor
The turnover factor represents the frequency of stock trading turnover within a given period, which is applied to measure stock trading activity and liquidity premium.<br>
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$Daily\ Turnover\ Rate\ =\frac{Daily\ Trading\ Volume\ (shares)}{Total\ Outstanding\ Shares\ (shares)}\ ×100%$$  
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$20\ -\ Day\ Turnover\ Factor\ =\ \frac{1}{20}\ \sum_{i\ =\ 1}^{20}{Daily\ Turnover\ Rate}_i$$  
Where:  
Factor construction: daily turnover rate is calculated by dividing daily trading turnover by total market capitalization of listed firms.  
Smoothing processing: to eliminate random noise caused by single-day trading volatility, we adopt a 20-trading-day rolling average approach to construct the average turnover factor.

## 3. Data Preprocessing & Cleaning
First conduct universe filtering:<br>
•	Exclude penny stocks with excessively low prices and illiquid stocks with low trading volume on each trading day.<br>
•	Eliminate stocks trading below 0.5 HKD.<br>
•	Remove stocks whose turnover falls within the bottom 20% of the whole market.

Proceed with cross-sectional processing afterwards:<br>
•	Outlier removal via MAD method<br>b
•	Z-score standardization<br>
•	Missing values are filled with zero, which equals the mean value after standardization.

## 4. Factor Neutralization
To eliminate disturbances exerted by industry and size effects on factor performance and extract pure alpha excess returns, we conduct cross-sectional neutralization on raw factors. There are two mainstream neutralization approaches:<br>
&emsp; 1) Industry neutralization for discrete data<br>
&emsp; 2) Market capitalization neutralization for continuous data

### 4.1 The Unified Neutralization Model
On each trading day t, we conduct the following cross-sectional regression for each stock i to remove market capitalization effects and industry biases.<br>
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$f_{i,t} = \beta_{0,t} + \beta_{\text{size},t} \cdot \ln(MC_{i,t}) + \sum_{j=1}^{K-1} \gamma_{j,t} \cdot Ind_{i,j,t} + \epsilon_{i,t}$$  
Where:<br>
$f_{i,t}$: raw factor value of stock i on day t, including skewness, momentum and turnover.<br>
$\epsilon_{i,t}$: regression residual, namely the purified factor value after neutralization.

### 4.2 Industry (or Sector) Neutralization
This study adopts the residual regression method for factor neutralization. Dummy variable regression is applied to process discrete data such as industry classifications.

Assuming there are n industries in the market, the regression model captures the average industry exposure and strips such effects out of raw factor values. The derived residual \epsilon    serves as the industry-neutralized factor free from industry influences.<br>
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$f_i=\beta_0+\sum_{j=1}^{n}\gamma_j\cdot Ind_{i,j}+\epsilon_i$$  
Where:<br>
$f_i$: raw factor value of stock i, such as original momentum and skewness.<br>
$\beta_0$: intercept term representing the benchmark factor level across the whole market.<br>
$Ind_{i,j}$: industry dummy variable, which equals 1 if stock i belongs to industry j , otherwise 0.<br>
$\gamma_j$: specific effect of industry j, measuring its average contribution to the factor.<br>
$\epsilon_i$: regression residual, namely the neutralized alpha signal excluding industry deviations.

### 4.3 Size Neutralization
Logarithmic transformation is applied to market capitalization to capture nonlinear size effects and avoid assigning equal weights to all stocks.<br>
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$X_{size,i}\ =\ ln({MarketCap}_i)$$

Weight logic of WLS: The weight matrix $W$ adopted in regression is a diagonal matrix, whose diagonal elements are the square root of stock market capitalization.  
&emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; &emsp; $$w_{ii}=\sqrt{MarketCap_i}$$  
_Note: weighting by the square root of market capitalization aims to mitigate the excessive impact of small-cap stocks on regression coefficients and enhance the representativeness of mid-to-large-cap samples._

### 4.4 Final Standardization
After industry and market capitalization neutralization, we obtain the residual $\epsilon_{i,t}$ of each stock, which represents the pure factor value excluding size and industry influences. Cross-sectional Z-score standardization is conducted to ensure comparability among different factors.

$${\hat{f}}_{i,t}=\frac{\epsilon_{i,t}-\mu_{\epsilon,t}}{\sigma_{\epsilon,t}}$$

Where:<br>
$\mu_{\epsilon,t}$: mean value of all sample residuals on the current day.<br>
$\sigma_{\epsilon,t}$: standard deviation of all sample residuals on the current day.













