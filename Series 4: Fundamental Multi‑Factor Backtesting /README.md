# Fundamental Multi‑Factor Backtesting  
In Hong Kong equity market, earnings shifts from corporate financial reports represent key alpha signals for medium to long horizon stock picking. This work develops a fundamental multi factor framework using financial data of Hong Kong listed equities. It walks through data cleaning, factor preprocessing, factor combination and semi annual rebalance backtesting, and adopts standard backtesting to evaluate the strategy’s historical risk return profile.

## Table of Contents

1. Data Selection  
2. Factor Construction  
2.1 Factor Library  
2.2 Factor Preprocessing  
2.3 Factor Effectiveness Test  
3. Multi Factor Combination  
4. Multi Factor Stock Selection Strategy  
5. Analysis of Strategy Backtest Results

## 1. Data Selection  
Sample size: 440 stocks; sample period: Jan 2019 – Dec 2025; total independent monthly cross sections: 1755.  
Industry coverage: IT & Internet, Financials, Real Estate, Consumer, Healthcare, Energy & Industrials, Utilities & Telecom.

## 2. Factor Construction  
### 2.1 Factor Library  
This study adopts three earnings growth fundamental factors: semi annual gross profit margin YoY (gpm_semi_YoY), semi annual ROE YoY (roe_6m_YoY), and semi annual ROA YoY (roa_YoY).  

&emsp;1) gpm_semi_YoY: semi annual gross profit margin YoY  

$$
gpmSemi = \frac{GrossProfit}{OperatingRevenue}
$$

$$
gpmSemiYoY = \frac{gpmSemi_{t}}{gpmSemi_{t-2}} - 1
$$

&emsp;2) roe_6m_YoY: semi annual ROE YoY

$$
roe = \frac{holderProfit}{ShareholdersEquity}
$$

$$
roe6mYoY = \frac{roe_{t}}{roe_{t-2}} - 1
$$

&emsp;3) roa_YoY: ROA YoY

$$
roa = \frac{holderProfit}{TotalAssets}
$$

$$
roaYoY = \frac{roa_{t}}{roa_{t-2}} - 1
$$

## 2.2 Factor Preprocessing
Raw financial factors feature outliers, inconsistent scales and style biases and cannot be directly weighted for factor combination. For this reason, each factor is processed via a standard pipeline consisting of MAD based winsorisation, Z score standardisation, and industry market cap neutralisation.

&emsp;1) MAD based winsorisation

$$
\begin{aligned}
Med &= median(factor) \\
MAD &= median\big(|factor - Med|\big) \\
Scale &= 1.4826 \times MAD \\
Upper &= Med + 3 \times Scale \\
Lower &= Med - 3 \times Scale
\end{aligned}
$$

Factor values are constrained between Lower  and Upper. Any out of bound observations will be truncated.

&emsp;2) Z score standardisation  
Factors exhibit substantial heterogeneity in value ranges and volatility magnitudes, so standardisation is a prerequisite for valid cross factor comparison.

$$
factorStd = \frac{factorRaw - \mu}{\sigma}
$$

μ denotes the cross sectional mean of the factor, and σ represents its cross sectional standard deviation.

&emsp;3) Industry and market cap neutralisation  
Earnings related factors are subject to industry effects and market cap style exposure. Linear regression is implemented to obtain pure alpha residuals purged of such style interferences.

$$
factorStd = \alpha + \sum \beta_i \cdot C(\mathrm{Industry}) + \gamma \cdot \ln MV + \varepsilon
$$

- $C(\mathrm{Industry})$: industry dummy variable  
- $\ln MV$: logarithm of total market value  
- Residual $\varepsilon$ corresponds to $factorNeutral$, the neutralised factor fed into factor combination.

##2.3 Factor Effectiveness Test  
Before weighted multi factor synthesis, we test each single factor’s predictability of future returns using cross sectional IC and time series ICIR.

&emsp;1) Cross sectional Information Coefficient  
Before multi factor combination, cross sectional IC and time series IC_IR are adopted to test each factor’s return predictability.

$$
IC_t = \mathrm{Corr}\big(FactorNeutral_{t},\; Return_{t \to t+1}\big)
$$

At every rebalancing date, we calculate the Information Coefficient (IC), the Pearson correlation between factor values and stocks’ semi annual forward returns. Factor predictability is evaluated using IC’s time series properties. AvgIC shows the direction of prediction, and IC_IR measures return stability, which guides our factor weighting scheme.  
IC values obtained at each rebalance are pooled across time to form a time series IC sample.  
The time series mean of IC reflects the factor’s overall predictive orientation: a positive mean suggests higher factor exposures tend to correspond to stronger subsequent stock returns. Less volatile IC over time indicates a more robust forecasting signal.  

&emsp;2) Times Series ICIR  
We compute IC_IR to measure how strong and persistent a factor’s predictability is. This metric also feeds into factor combination.

$$
\bar{IC} = \frac{1}{T}\sum_{t=1}^{T} IC_t
$$

$$
ICIR = \frac{\bar{IC}}{\sigma(IC_t)}
$$

- $\bar{IC}$ is the time‑series mean of IC
- $\sigma(IC)$ is its time‑series standard deviation.

## 3. Multi Factor Combination  
Following preprocessing and effectiveness evaluation, all three earnings growth factors exhibit positive predictive power. Nevertheless, individual factors show considerable time series volatility in their IC values. This study therefore adopts a constrained optimisation approach that maximises IC_IR subject to non negative weight constraints, to solve for factor weights and construct the composite stock selection factor.

&emsp;1) Optimisation Objective Function  
The objective is to maximise the IC_IR of the composite factor:  

$$
\max \; ICIR = \frac{\vec{w}^{\,T} \cdot \overrightarrow{IC}}{\sqrt{\vec{w}^{\,T} \Sigma \vec{w}}}
$$

- $\vec{w}$: factor‑weight vector
- $\overrightarrow{IC}$: vector formed by cross‑sectional IC of each factor
- $\Sigma$: covariance matrix of neutralised factors, capturing co‑movement across individual factors

&emsp;2) Constraints  

$$
\begin{cases}
\displaystyle\sum w_i = 1 \\
w_i \ge 0
\end{cases}
$$

This set of constraints impose long only factor weights and rule out short positions, which aligns with the practical long only stock selection context of the Hong Kong equity market. Factor weights are restricted to be non negative and sum to unity.

&emsp;3) Optimal Weight Estimation  
The SLSQP sequential least squares programming algorithm is utilised to solve the constrained weight optimisation problem.  
Procedures:  
Compute cross sectional IC for neutralised factors at each rebalance to form the IC vector; estimate factor covariance matrix Σ from contemporaneous neutral factor sample data; pass the IC vector and covariance matrix into the optimiser to solve for weights. Equal weights are used if optimisation does not converge.  

&emsp;4) Composite Factor Construction
After obtaining the weight vector $w_i$, the composite factor is constructed via linear weighting over neutralised factor outputs:

$$
Composite = \sum w_i \cdot FactorNeutral_i
$$

Weights are re estimated at every rebalancing period using real time cross sectional IC and covariance information. Accordingly, factor weights are dynamically updated rather than held static throughout the sample window.

## 4. Multi Factor Stock Selection Strategy  
&emsp;1) Rebalancing Schedule. 
The strategy rebalances semi annually, aligned with corporate financial report release cycles. Each rebalance becomes effective on announce_date, the official disclosure date of financial statements. Factor values are recalculated, optimal weights and the composite factor are recomputed, and portfolio positions are accordingly adjusted. Each holding window spans roughly 120 trading days (semi annual). Look ahead bias is eliminated in backtest implementation.

&emsp;2) StockSelection Rules  
At each rebalancing date, stocks are sorted by the computed composite factor. We select the top ranked 30 securities and construct an equally weighted long only portfolio.  
Portfolio turnover between old and new holdings is calculated upon each rebalance, and transaction costs are deducted proportionally based on realised turnover.  

$$
NetReturn = RawPortfolioReturn - Turnover \times (\mathrm{Commission} + \mathrm{StampTax})
$$

## 5. Analysis of Strategy Backtest Results  
&emsp;1) Net Value Curve Performance  
Driven by bear market conditions, portfolio net value declined throughout 2019 2020. After bottoming out in 2020, the equity curve trended upward amid volatility with higher troughs over time. Returns improved further from 2024 to 2025, delivering positive cumulative performance over the full sample horizon.  

<div align="center">

![performance](https://github.com/hanyu-L/HK-Equity-Multi-Factor-Model/blob/a32b4df35ac7eafba50ff8f5274ff864598e9984/Series%204%3A%20Fundamental%20Multi%E2%80%91Factor%20Backtesting%20/Net%20Value%20Curve%20Performance.png)

</div>

&emsp; 2) Performance Metrics

 <div align="center">
   
| Metric                  |   Value   |
|:------------------------|:---------:|
| Total Return            |  291.34%  |
| Annualized Return       |   25.51%  |
| Max Drawdown            |  -26.85%  |
| Annual Volatility       |   21.78%  |
| Sharpe Ratio (2% RF)    |    1.08   |

</div>

The strategy achieves a total period return of 291.34% and an annualised return of 25.51%. As an unhedged long only strategy for Hong Kong equities, its annualised volatility of 21.78% is consistent with typical market fluctuation characteristics. The maximum drawdown of 26.85% occurred during the early sample bear market episode. With a Sharpe ratio of 1.08, the strategy exhibits favourable risk adjusted return properties overall.










