# Fundamental Multi-Factor Robust Test  
To rule out overfitting in series 4 multi-factor strategy, we implement robustness checks from three perspectives: stock selection scale, lagged rebalancing and performance under different market environments.  

## Table of Contents

1. Parameter Sensitivity Analysis  
1.1 Test Framework  
1.2 Net Value Performance of Grouped Backtests  
1.3 Test Result	 
2. Factor Decay Validity Test	 
2.1 Test Framework	 
2.2 Net Value Performance of Grouped Backtests	 
2.3 Test Result	 
3. Performance Test Across Distinct Market Regimes	 
3.1 Test Framework and Market Classification	 
3.2 Performance in Bear, Range-bound and Rising Markets	 
4. Conclusion	 

## 1. Parameter Sensitivity Analysis  
### 1.1 Test Framework  
The position size of each portfolio is a core adjustable parameter. Severe performance swings caused by varying stock counts signal unstable factor logic. We fix factor weights, rebalancing frequency and transaction costs, then build three portfolios consisting of the top 20, 30 and 50 stocks ranked by factor scores for backtesting. Cross-comparison of portfolio outcomes identifies parameter sensitivity toward stock selection quantity.  

### 1.2 Net Value Performance of Grouped Backtests  
All three portfolios suffered drawdowns amid the 2019–2020 bear market and rose synchronously after 2020, sharing highly similar trends. The Top30 portfolio maintained superior net value over the full period, outperforming the Top50 and Top20 portfolios sequentially. Adjusting stock selection quantity fails to reverse the strategy’s long-term trend and only alters return ceilings and volatility levels.  

 <div align="center">
Net Value Trends for Portfolios with Different Stock Capacities<br>
</div>

![test 1](https://github.com/hanyu-L/HK-Equity-Multi-Factor-Model/blob/49917274766a4f60caee85aa5dd8b9a1640ecd3d/Series%205%3A%20Robust%20Test%20/Robust%20Test%201%20-%20Parameter%20Sensitivity%20-%20Different%20Top%20N%20Selection.png)

All three portfolios suffered drawdowns amid the 2019–2020 bear market and rose synchronously after 2020, sharing highly similar trends. The Top30 portfolio maintained superior net value over the full period, outperforming the Top50 and Top20 portfolios sequentially. Adjusting stock selection quantity fails to reverse the strategy’s long-term trend and only alters return ceilings and volatility levels.  

### 1.3 Test Result

<div align="center">

| Portfolio | Annualized Return | Maximum Drawdown | Annualized Volatility | Sharpe Ratio |
|:---------:|:-----------------:|:----------------:|:---------------------:|:------------:|
| Top20     | 15.35%            | -41.16%          | 22.61%                | 0.5904       |
| Top30     | 25.51%            | -26.85%          | 21.78%                | 1.0793       |
| Top50     | 15.30%            | -33.92%          | 19.35%                | 0.6872       |

</div>

Conclusions  
&emsp;1) All portfolios move in tandem and keep growing even with changed stock counts, proving the multi-factor framework is stable against minor parameter adjustments.
&emsp;2) The optimal portfolio size is 30 stocks, which achieves the best return, drawdown control and Sharpe ratio. A 20-stock portfolio suffers concentration risk, while a 50-stock portfolio includes low-alpha stocks and earns lower returns.

## 2. Factor Decay Validity Test  
### 2.1 Test Framework  
In live trading, earnings report release and factor computation inevitably consume time, making immediate portfolio rebalancing upon receiving real-time factor signals impractical. If a factor’s predictive power fades rapidly, moderate delays in rebalancing will trigger sharp declines in portfolio returns, which signifies high time-sensitivity and weak stability of the factor itself.  
In this test, we fix stock selection volume, transaction cost rules and all other settings unchanged, and construct three rebalancing schemes: Lag0 rebalances portfolios on the original schedule; Lag1 postpones rebalancing by one full rebalancing cycle; Lag2 delays rebalancing for two cycles. By comparing return disparities across the three groups, we quantify the decay speed of the composite factor.  

### 2.2 Net Value Performance of Grouped Backtests  
The three net value curves share highly synchronized fluctuation patterns: all portfolios underwent drawdowns collectively from 2019 to 2020 and trended upward simultaneously afterward. The baseline Lag0 portfolio without rebalancing lags achieves the highest terminal net value. The Lag1 portfolio rebalanced with a one-cycle lag closely tracks the baseline trajectory, with nearly overlapping curves in later periods. By contrast, the Lag2 portfolio with two rebalancing cycles of delay maintains distinctly lower net values throughout the entire sample period and generates inferior cumulative returns.  
It can be concluded that delayed rebalancing fails to alter the overall trend characteristics of the strategy and only gradually erodes terminal portfolio returns.  

![test 2](https://github.com/hanyu-L/HK-Equity-Multi-Factor-Model/blob/49917274766a4f60caee85aa5dd8b9a1640ecd3d/Series%205%3A%20Robust%20Test%20/Robust%20Test%202%20-%20Factor%20Decay%20Test%20-%20Lag%20Rebalance.png)

 <div align="center">
Strategy Net Value Trends with Lagged Rebalancing
<br>
</div>

### 2.3 Test Result  

 <div align="center">
  
| Rebalancing Mode     | Annualized Return | Maximum Drawdown | Annualized Volatility | Sharpe Ratio |
|----------------------|-------------------|------------------|-----------------------|--------------|
| Lag0: base case      | 25.51%            | -26.85%          | 21.78%                | 1.0793       |
| Lag1: 1-cycle lag    | 24.30%            | -26.85%          | 22.18%                | 1.0055       |
| Lag2: 2-cycle lag    | 19.32%            | -26.85%          | 21.16%                | 0.8184       |
 </div>

Conclusions:  
&emsp;1) Drawdown levels stay consistent across groups, proving lagged rebalancing creates no extra downside risk. Returns and Sharpe ratios drop gradually with longer delays, verifying factor decay: newly updated financial signals work best.
&emsp;2) A single cycle of rebalancing lag only slightly hurts returns, so the strategy retains profitability in real trading as long as rebalancing delays stay within one cycle, granting decent operational flexibility.
&emsp;3) The factor is effective but fades slowly over time. In practice, investors ought to refresh factors and rebalance positions immediately after earnings announcements.

## 3. Performance Test Across Distinct Market Regimes  
### 3.1 Test Framework and Market Classification  
We divide the full sample into three periods: bear market (2019–2020), sideways market (2021–2023) and bull uptrend (2024–2025). Period-wise performance is measured to examine how the multi-factor portfolio adapts to different market styles.

### 3.2 Performance in Bear, Range-bound and Rising Markets  

 <div align="center">
| Market Classification         | Annualized Return | Maximum Drawdown | Annualized Volatility | Sharpe Ratio |
|-------------------------------|-------------------|------------------|-----------------------|--------------|
| Bear market (2019–2020)       | 70.22%            | -12.30%          | 28.92%                | 2.3592       |
| Sideways market (2021–2023)   | -2.94%            | -26.85%          | 18.61%                | -0.2656      |
| Bull uptrend (2024–2025)      | 64.56%            | 0.00%            | 17.23%                | 3.6314       |
</div>

Conclusion：  
&emsp;1) The strategy earns positive returns in both bull and bear markets. It achieves 70.22% annual return with merely 12.3% maximum drawdown in the 2019–2020 bear market, and rises steadily without drawdowns (Sharpe = 3.63) in the 2024–2025 uptrend, validating its excess return capacity in trending markets.  
&emsp;2) The strategy underperforms slightly during 2021–2023 sideways consolidation, posting a -2.94% annual return and the largest full-period drawdown of 26.85%. Random stock fluctuations break fundamental factor pricing rules and hinder alpha generation.  
&emsp;3) This strategy works best under trending regimes. To avoid losses in sideways markets, investors can reduce positions or halt the strategy when prolonged consolidation is expected.

## 4. Conclusion  
Backtest results show that the strategy remains stable with varying stock selection quantities, and the 30-stock portfolio delivers the best performance. The factor decays slowly, and one-period lagged rebalancing only slightly impairs returns, ensuring practical operability in real trading. The strategy is profitable in both bull and bear markets but underperforms with notable drawdowns in sideways markets. With solid overall robustness and no reliance on bullish trends, the strategy can be further improved by incorporating market style judgment to mitigate losses in oscillating market conditions.  





