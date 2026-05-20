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

$$Momentum\ =\ \frac{P_{t-1}}{P_{t-n}}\ -\ 1$$  

Where:  
$P_t$: the closing price on day t-1.<br>
$P_t-n$: the closing price recorded $n$ trading days prior.  
In this study, we set $n$ = 20, representing the closing price 20 trading days earlier (approximately one calendar month)

### 2.3 Turnover Factor
The turnover factor represents the frequency of stock trading turnover within a given period, which is applied to measure stock trading activity and liquidity premium.

$$Daily\ Turnover\ Rate\ =\frac{Daily\ Trading\ Volume\ (shares)}{Total\ Outstanding\ Shares\ (shares)}\ ×100%$$
$$20\ -\ Day\ Turnover\ Factor\ =\ \frac{1}{20}\ \sum_{i\ =\ 1}^{20}{Daily\ Turnover\ Rate}_i$$

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
On each trading day t, we conduct the following cross-sectional regression for each stock i to remove market capitalization effects and industry biases.

$$f_{i,t} = \beta_{0,t} + \beta_{\text{size},t} \cdot \ln(MC_{i,t}) + \sum_{j=1}^{K-1} \gamma_{j,t} \cdot Ind_{i,j,t} + \epsilon_{i,t}$$

Where:<br>
$f_{i,t}$: raw factor value of stock i on day t, including skewness, momentum and turnover.<br>
$\epsilon_{i,t}$: regression residual, namely the purified factor value after neutralization.

### 4.2 Industry (or Sector) Neutralization
This study adopts the residual regression method for factor neutralization. Dummy variable regression is applied to process discrete data such as industry classifications.

Assuming there are n industries in the market, the regression model captures the average industry exposure and strips such effects out of raw factor values. The derived residual \epsilon    serves as the industry-neutralized factor free from industry influences.

$$f_i=\beta_0+\sum_{j=1}^{n}\gamma_j\cdot Ind_{i,j}+\epsilon_i$$

Where:<br>
$f_i$: raw factor value of stock i, such as original momentum and skewness.<br>
$\beta_0$: intercept term representing the benchmark factor level across the whole market.<br>
$Ind_{i,j}$: industry dummy variable, which equals 1 if stock i belongs to industry j , otherwise 0.<br>
$\gamma_j$: specific effect of industry j, measuring its average contribution to the factor.<br>
$\epsilon_i$: regression residual, namely the neutralized alpha signal excluding industry deviations.

### 4.3 Size Neutralization
Logarithmic transformation is applied to market capitalization to capture nonlinear size effects and avoid assigning equal weights to all stocks.

$$X_{size,i}\ =\ ln({MarketCap}_i)$$

Weight logic of WLS: The weight matrix $W$ adopted in regression is a diagonal matrix, whose diagonal elements are the square root of stock market capitalization.

$$w_{ii}=\sqrt{MarketCap_i}$$

_Note: weighting by the square root of market capitalization aims to mitigate the excessive impact of small-cap stocks on regression coefficients and enhance the representativeness of mid-to-large-cap samples._

### 4.4 Final Standardization
After industry and market capitalization neutralization, we obtain the residual $\epsilon_{i,t}$ of each stock, which represents the pure factor value excluding size and industry influences. Cross-sectional Z-score standardization is conducted to ensure comparability among different factors.

$${\hat{f}}_{i,t}=\frac{\epsilon_{i,t}-\mu_{\epsilon,t}}{\sigma_{\epsilon,t}}$$

Where:<br>
$\mu_{\epsilon,t}$: mean value of all sample residuals on the current day.<br>
$\sigma_{\epsilon,t}$: standard deviation of all sample residuals on the current day.

## 5. Single-Factor Test
This chapter focuses on three core dimensions of factors:<br>
&emsp; 1)	IC: whether there exists predictive correlation between factor values and stock returns.<br>
&emsp; 2)	IR: whether such predictive power remains stable and reliable in time series.<br>
&emsp; 3)	Stratified Backtest: to verify whether stocks with higher or lower factor values can gain higher excess returns.

### 5.1 IC and RankIC
Pearson Correlation Coefficient (IC): measures the linear correlation between factor value X and stock return Y.

$$\rho_{X,Y}=\ \frac{Cov(X,Y)}{\sigma_X\sigma_Y}$$

Spearman Correlation Coefficient (Rank IC): measures the correlation between factor value ranks and stock return ranks.

$$\rho_s\ =\ 1\ -\ \frac{6\sum{d_i}^2}{n(n^2\ -\ 1)}$$

Where:
$d_i$: rank difference between factor rank and return rank of stock $i$<br>
$n$: number of stocks in cross-section sample, in this case, we set $n$ = 261

### 5.2 IR
Information Ratio: measures excess returns per unit risk. Generally, an IR above 0.5 indicates a highly robust factor.

$$IR=\frac{IC\mathrm{\ Mean}}{IC\mathrm{\ Std}}$$

Significance test: T-test is performed on the RankIC sequence to determine whether factor returns are statistically significantly different from zero.

### 5.3 Stratified Backtesting
To intuitively observe the stock selection differentiation ability of factors, we equally divide 261 stocks into 5 quintile groups by factor values on each cross-section. The average future returns and cumulative returns of each group are then calculated.

### 5.4 Factor Assessment Approach
IC Mean<br>
RankIC Mean<br>
IC Std<br>
IR (Information Ratio)<br>
IC > 0 Rate<br>
IC T-Stat<br>
IC P-Value<br>

Quintile grouped daily return<br>
Cumulative net value of each group<br>
Annualized return of stratified portfolios<br>
Long-short portfolio daily return<br>
Long-short portfolio cumulative net value<br>

### 5.5 Testing Results
The turnover factor presents highly significant IC statistics (RankIC=-0.0419, IR=-0.26, t-value=-7.0367, p<0.001) and exhibits distinct return monotonicity in stratified backtests, with returns declining steadily from Group 1 (37.39%) to Group 5 (-6.06%). It serves as the only valid negative predictive factor, demonstrating that low turnover rate possesses strong alpha capturing capability in the Hong Kong stock market.

In contrast, the momentum factor and skewness factor show insignificant IC values (p>0.05) without stable monotonic stratified returns, thus having no independent investment value during the sample period.<br>
 <div align="center">
IC/IR Statistics Summary<br>
</div>
 <div align="center">
   
 &emsp;&emsp;&emsp;&emsp;&emsp;| Turnover Factor | Momentum Factor | Skewness Factor |  
 ------------- | --------------- | --------------- | --------------- |  
 IC Mean       | -0.0321         | -0.0012         | 0.0011          |  
 RankIC Mean   | -0.0419         | -0.0035         | -0.0064         |  
 IC Std        | 0.1590          | 0.1558          | 0.0967          |  
 IR            | -0.2635         | -0.0223         | -0.0666         |  
 IC > 0 Rate   | 0.3787          | 0.5077          | 0.4614          |  
 IC T-Stat     | -7.0367         | -0.5955         | -1.7775         |  
 IC P-Value    | 0.0000          | 0.5517          | 0.0759          |  

</div>

 <div align="center">
Annualized Return by Quintile<br>
</div>
 <div align="center">
   
Group | Turnover Factor | Momentum Factor | Skewness Factor |  
 ---- | --------------- | --------------- | --------------- |  
1     | 0.373882         | 0.181530         | 0.050420          |  
2     | 0.149780         | 0.076396         | 0.137893         |  
3     | 0.146263          | 0.106955          | 0.110174          |  
4     | 0.042093         | 0.101185         | 0.154415         |  
5     | -0.060598          | 0.185286          | 0.200668          |  


</div>

On the daily cross-section, the 261 stocks are sorted by target factor values in ascending order and evenly divided into five quintile portfolios, labeled Group 1 (lowest factor value) to Group 5 (highest factor value), with each group containing approximately 52 to 53 stocks. The following indicators are calculated respectively:<br>
&emsp; 1)	Cumulative net value of the five groups<br>
&emsp; 2)	Cumulative return curve of long-short portfolios

Among the three factors studied, the Turnover Factor delivers the best performance. Its stratified backtest results are highly consistent with IC and IR statistics. The net value curves of quintile portfolios show steady monotonic stratification, and low-turnover portfolios consistently outperform high-turnover portfolios significantly. Meanwhile, the long-short portfolio return curve keeps rising steadily in the long run, proving that this factor owns sustainable and reliable predictive power and practical application value.

By contrast, the Momentum Factor is completely ineffective within the sample period. Its stratified net value curves overlap randomly without valid differentiation, and long-short returns fluctuate around zero, failing to generate stable excess returns.

The Skewness Factor shows inconsistent performance. Stratified backtests reveal certain positive return differentiation yet lack stability. Its long-short portfolio returns remain negative for a long time and fail to fully align with the direction of IC statistics, indicating weak effectiveness under current market conditions and requiring further verification.

 <div align="center">
Turnover Factor Stratified Backtest result<br>
</div>


















































































































































































































































































