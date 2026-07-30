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

1) gpm_semi_YoY: semi annual gross profit margin YoY  

gpm\_semi=\frac{Gross\_profit}{Operating\_revenue}  

gpm\_semi\_YoY=\frac{gpm\_semi_{t}}{gpm\_semi_{t-2}}-1  


$$
\mathit{gpm\_semi} = \frac{\text{Gross\_profit}}{\text{Operating\_revenue}}
$$

$$
\mathit{gpm\_semi\_YoY} = \frac{\mathit{gpm\_semi}_{t}}{\mathit{gpm\_semi}_{t-2}} - 1
$$


---

$$
\mathit{\text{gpm\_semi}} = \frac{\text{Gross\_profit}}{\text{Operating\_revenue}}
$$

$$
\mathit{\text{gpm\_semi\_YoY}} = \frac{\mathit{\text{gpm\_semi}}_{t}}{\mathit{\text{gpm\_semi}}_{t-2}} - 1
$$

---

$$
gpmSemi = \frac{\text{Gross\_profit}}{\text{Operating\_revenue}}
$$

$$
gpmSemiYoY = \frac{gpmSemi_{t}}{gpmSemi_{t-2}} - 1
$$






