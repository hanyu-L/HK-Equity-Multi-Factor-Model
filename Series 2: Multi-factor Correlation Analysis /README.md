# Multi-factor Correlation Analysis
440 stocks, covering from January 2019 to December 2025, with a total of 1,755 independent monthly cross-sections.

## Table of Contents

1. Data ingestion  
2. Factor construction  
2.1 Financial factors  
Monthly cross-sectional financial factor calculation  
Valuation factors  
Growth factors  
Profitability quality factors  
Selected computational results  
2.2 Price-volume factors  
2.3 Factor preprocessing and neutralisation  
2.3.1 MAD-based 5σ winsorisation  
2.3.2 Cross-sectional Z-score standardisation  
2.3.3 Missing value imputation after standardisation  
2.3.4 Sector neutralisation and market capitalisation  
3.  Single factor testing framework  
3.1 Single factor testing framework  
3.1.1 Information Coefficient (Pearson Correlation Coefficient)  
3.2.2 IC Information Ratio (IC_IR)  
3.1.3 Quintile long-short returns  
3.2 Single-factor screening results  
4.  Correlation and multicollinearity of surviving factors  
4.1 Correlation testing methodology  
4.2 Correlation matrix analysis  
4.3 Multicollinearity  
4.4 Correlation analysis and factor combination  

## 1. Data ingestion
The dataset contains 440 stocks, covering the period from January 2019 to December 2025, with a total of 1,755 independent monthly cross-sections.
Industries included are IT & Internet, Financials, Real Estate, Consumer, Healthcare, Energy & Industrials and Utilities & Telecom.

## 2. Factor construction
### 2.1 Financial factors
Given uneven disclosure schedules and variable reporting lags for Hong Kong market financial data, this study applies a Point-In-Time (PIT) mechanism (implemented by the get_latest_disclosed_fin function) to avoid look-ahead bias. Rebalancing is anchored to the final trading day each month. Only financial reports disclosed on or prior to this cut-off date enter factor calculations. We explicitly separate the fiscal period end date (end_date) and formal disclosure date (std_report_date) to guarantee accurate time matching.
Special care is required for date conversion. Original timestamps exist as 8-digit numeric strings such as 20190630. Invoking pd.to_datetime directly misreads these values as Unix timestamps referenced to 1970. The resulting temporal mismatch impairs rolling 12-month lookups of historical financial data and produces NaNs for TTM indicators. All date columns must first be converted to strings and parsed using format="%Y%m%d".

### Monthly cross-sectional financial factor calculation
A-shares mandate annual and semi-annual filings with optional quarterly reports, whereas Hong Kong-listed issuers are not required to release semi-annual statements and follow heterogeneous disclosure schedules. A hierarchical rule set is applied to compute TTM figures:
&emsp; 1) When only an annual report is available for the current period, TTM is derived directly from this full-year data.  
&emsp; 2) When a semi-annual or quarterly report is disclosed, the 12-month rolling metric is reconstructed via: Last full-year figure − same-period figure from last year + current single-period result. Matching historical filings of the identical category and the prior full-year annual report is mandatory.  
&emsp; 3) The TTM value becomes NaN if the matching prior-period report or prior annual report cannot be located. All financial factors are filled with NaN for stocks lacking any financial disclosures.  

#### Valuation factors
&emsp; 1) Earnings-to-price ratios

$$EP_{ttm} = \frac{NetProfit_{ttm}}{MarketCap}$$

&emsp; $EPcut_{ttm}$: deducted non-recurring earnings per market cap  

$$EPcut_{ttm} = \frac{NetProfit\ Excluding\ Non-recurring\ Items_{TTM}}{Total\ market\ cap}$$

&emsp; 2) Asset-to-price ratios  
&emsp; BP: book value per market cap

$$BP = \frac{shareholders' equity}{total market cap}$$

&emsp; $SP_{ttm}$: sales revenue per market cap

$$SP_{ttm} = \frac{revenue_{ttm}}{total market cap}$$

&emsp; 3) Inverted valuation multiples  
&emsp; PE: price earnings ratio  

$$PE = \frac{total market cap}{net revenue_{ttm}}$$

&emsp; PB: price book ratio

$$PB = \frac{total market cap}{shareholders' equity}$$

&emsp; PCF: price cash flow ratio

$$PCF = \frac{total market cap}{operating cash flow_{ttm}}$$

&emsp; PS: price sales ratio

$$PS = \frac{total market cap}{revenue_{ttm}}$$

&emsp; 4) Growth-value alignment
&emsp; PEG: price earnings growth. Assign NaN when the growth rate is zero.

$$PEG = \frac{PE}{{sales\_g}_{ttm}}$$

#### Growth factors
&emsp; 1) TTM year-on-year growth
&emsp; Match the TTM financial statements corresponding to the fiscal closing date lagged by one full year. Observations without complete prior-year financial reports are set to missing.
&emsp; General formula:

$$G_{ttm} = \frac{X_{ttm,current}}{X_{{ttm,last\_year}}} - 1$$

&emsp; $X_{ttm,current}$ = current-period trailing 12-month metric derived from financial statements  
&emsp; $X_{{ttm,last\_year}}$ = trailing 12-month metric recorded at the identical fiscal closing date of the prior year.  

&emsp; If the prior-year TTM value is missing or equals zero, the factor is assigned np.nan.
-------------------开始有问题  

&emsp; Sales_G_ttm: TTM sales year-on-year growth

$$Sales\_G_{ttm} = \frac{rev_{ttm,current}}{rev_{\mathit{ttm,last\_year}}} - 1$$

&emsp; Profit_G_ttm: TTM net profit year-on-year growth

$$Profit\_G_{ttm} = \frac{net\_profit_{ttm,current}}{net\_profit_{\mathit{ttm,last\_year}}} - 1$$

&emsp; OCF_G_ttm: TTM operating cash flow YoY growth

$$OCF\_G_{ttm} = \frac{ocf_{ttm,current}}{ocf_{\mathit{ttm,last\_year}}} - 1$$

&emsp; ROE_G_ttm: TTM ROE year-on-year growth

$$ROE\_G_{ttm} = \frac{roe_{ttm,current}}{roe_{\mathit{ttm,last\_year}}} - 1$$


-----服了 一直改不好


&emsp; 2) Semi-annual single-period YoY growth
&emsp; This calculation applies exclusively to semi-annual and quarterly filings. Metrics are computed by matching financial statements of the identical reporting frequency from the previous year to derive year-on-year growth for single-period operating indicators and single-period ROE. Observations based on annual reports contain no semi-annual comparative data and are uniformly assigned np.nan.

&emsp; Sale_G_6m: semi-annual sales year-on-year growth

$$Sale\_G_{6m} = \frac{Operating\ revenue_{current}}{Operating\ revenue_{last\_year}} - 1$$

&emsp; Profit_G_6m: semi-annual net profit year-on-year growth

$$Profit\_G_{6m} = \frac{Profit\ attributable\ to\ equity\ holders_{current}}{Profit\ attributable\ to\ equity\ holders_{last\_year}} - 1$$

&emsp; OCF_G_6m: semi-annual operating cash flow YoY Growth

$$OCF\_G_{6m} = \frac{netcash\ operate_{current}}{netcash\ operate_{last\_year}} - 1$$

&emsp; ROE_G_6m: semi-annual ROE year-on-year growth

$$ROE\_G_{6m} = \frac{roe\ 6m\_YoY_{current}}{roe\ 6m\_YoY_{last\_year}} - 1$$


#### Profitability quality factors  
&emsp; 1) Annualised TTM profitability metrics
$roe_{ttm}$: return on equity TTM

$$roe_{ttm} = \frac{net\_profit_{ttm}}{total\ equity\ avg}$$

&emsp; $roa_{ttm}$: return on assets TTM 

$$roa_{ttm} = \frac{net\_profit_{ttm}}{total\ asset\ avg}$$

&emsp; $gpm_{ttm}$: gross profit margin TTM

$$gpm_{ttm} = \frac{gross\_profit_{ttm}}{rev_{ttm}}$$

&emsp; $cfo\_to\_np_{ttm}$: cash flow to net profit TTM

$$cfo\_to\_np_{ttm} = \frac{ocf_{ttm}}{net\_profit_{ttm}}$$

&emsp; $roic_{ttm}$: return on invested capital TTM

$$roic_{ttm} = \frac{ebit_{ttm}}{invest\_cap_{ttm}}$$

&emsp; 2) Semi-annual YoY quality growth metrics
&emsp; General formula for year-on-year growth:  

YoY = $\frac{\text{Current semi-annual single-period metric}}{\text{Prior-year corresponding semi-annual single-period metric}} -1$

&emsp; If observations are incomplete or the denominator is zero, the factor value will be set to np.nan.

&emsp; roe_6m_YoY: return on equity semi-annual YoY

$$roe\_6m\_YoY = \frac{roe\_6m\_YoY_{current}}{roe\_6m\_YoY_{last\_semi}} - 1$$

&emsp; roa_YoY: return on assets semi-annual YoY

$$roa\_YoY = \frac{roa\_YoY_{current}}{roa\_YoY_{last\_semi}} - 1$$

&emsp; gpm_semi_YoY: gross profit margin semi-annual YoY

$$gpm\_semi\_YoY = \left(\frac{GrossProfit_{current}}{OperatingRevenue_{current}} \bigg/ \frac{GrossProfit_{last\_semi}}{OperatingRevenue_{last\_semi}}\right) - 1$$




### Selected computational results



### 2.2 Price-volume factors
To align with the monthly static cross-sections used for TTM financial factor construction and facilitate subsequent factor merging, only data observed on the last trading day of each month is adopted for price-volume factor calculation.  
This approach reduces data volume, shortens computation runtime and streamlines the dataset. The trade-off is the loss of daily granularity. In practical terms, only static price-volume metrics independent of daily historical series can be constructed; momentum, multi-period rolling turnover, idiosyncratic volatility and other indicators requiring continuous daily records are not feasible.  
As only end-of-month cross-sectional data — including the month-end open, close, high, low prices, aggregate monthly trading volume and month-end market capitalisation — is available, historical daily sequences spanning the prior 1, 3 or 6 months cannot be retrieved. Accordingly, only three simple static factors are constructed.  
Month return

$$month\ return = \frac{close}{open} - 1$$

Amplitude

$$amplitude = \frac{high - low}{open} - 1$$

Turnover rate

$$turnover\ rate = \frac{turnover}{market\ cap}$$

### 2.3 Factor preprocessing and neutralisation
Considering the traits of Hong Kong stock market, penny stocks and illiquid securities are excluded to prevent distorted empirical results driven by illiquid outliers.  
After filtering, all candidate factors within the sample pool undergo preprocessing following the sequence below.  

#### 2.3.1 MAD-based 5σ winsorisation
We adopt median absolute deviation (MAD) winsorisation with threshold set at 5 times the MAD. Factor values exceeding the upper and lower bounds are truncated to the corresponding boundary levels. If the MAD of a factor equals zero, the original series is retained without adjustment.

$$lower = median - 5 \times mad$$

$$upper = median + 5 \times mad$$

#### 2.3.2 Cross-sectional Z-score standardisation
Monthly cross-sectional Z-score standardisation is applied to eliminate dimensional disparities across different factors.

$$z = \frac{factor - mean(factor)}{std(factor)}$$

#### 2.3.3 Missing value imputation after standardisation  
After winsorisation and standardisation for all factors across every monthly cross-section, missing observations are filled with the cross-sectional average of the corresponding factor within the month.

#### 2.3.4 Sector neutralisation and market capitalisation  
A monthly cross-sectional WLS regression is estimated using standardised factors as the dependent variable, where sector dummy variables and the logarithm of total market capitalisation act as independent variables. Each stock is weighted by its month-end total market capitalisation.  The neutralised factor corresponds to the regression residual.  

$$factor = \alpha + \sum \beta_i \cdot ind_{i} + \gamma \cdot ln(market\ cap) + \varepsilon$$

## 3.  Single factor testing framework  
### 3.1 Single factor testing framework  
Can refer to my series 1: Single factor testing framework, section 5 https://github.com/hanyu-L/Single-Factor-Testing-Framework/tree/main/Series%201%3A%20Single-Factor-Testing-Framework 

#### 3.1.1 Information Coefficient (Pearson Correlation Coefficient)
The IC is defined as the Pearson correlation between standardised factor values on the monthly cross-section and each stock’s forward excess return over the subsequent month. It gauges the directional sign and linear strength of the factor’s return predictability.

$$IC_{t} = Corr(F_{i,t}, R_{i,t+1})$$

where F_idenotes the factor value of stock i on the current-month cross-section, and R_(i,t+1) represents the corresponding stock’s forward return in the subsequent period.  
Full-period average IC:

$$AvgIC = \frac{1}{N}\sum_{t=1}^{N} IC_{t}$$

Metric interpretation:  
&emsp; If "AvgIC "> 0: Higher factor values correspond to stronger stock returns in the following month, signifying a positive stock-selection factor.  
&emsp; If "AvgIC "< 0: Lower factor values predict higher returns over the next month, representing an inverted stock-selection factor.  
A larger absolute value of $AvgIC indicates a stronger linear association between the factor and future returns, translating to more robust predictive power for individual monthly cross-sections.  

#### 3.2.2 IC Information Ratio (IC_IR)
IC_IR  evaluates how consistently a factor predicts returns over time and gauges the persistence of excess return forecasts.

$$ICIR = \frac{AvgIC}{Std(IC_{t})}$$

Std(IC_{t}) denotes the standard deviation of monthly IC series across the full sample period, capturing monthly fluctuations in the factor’s predictive power.  
A higher absolute value of IC_IR indicates more stable predictive performance and lower noise embedded in the factor.  

#### 3.1.3 Quintile long-short returns
On each monthly cross-section, factors are sorted by rank and partitioned into five groups (q=5). Monthly average returns are computed for each quintile.  
Monthly long-short portfolio return is defined as the return of Quintile 5 minus the return of Quintile 1. This metric tests the monotonic return pattern across factor quantiles and assesses the factor’s capacity to capture excess returns.  

### 3.2 Single-factor screening results
Given limited data availability, relatively lenient thresholds are adopted for factor selection:

$$|AvgIC| > 0.01,\quad |IC\_IR| > 0.15$$

Surviving effective factors after screening are BP, PB, and gpm_semi_YoY.  
Factor test results (sorted in descending order of IC_IR):  

 <div align="center">
</div>
 <div align="center">
   
| factor_name     | avg_IC    | IC_IR    | avg_long_short_ret |
|----------------|-----------|----------|--------------------|
| PB             | 0.062149  | 0.401035 | 0.019465           |
| Profit_G_6m    | 0.015145  | 0.134342 | 0.004217           |
| open           | 0.005474  | 0.047955 | 0.003314           |
| low            | 0.005102  | 0.044486 | 0.002874           |
| high           | 0.003483  | 0.030865 | 0.003287           |
| Sales_G_6m     | 0.003353  | 0.023682 | 0.006042           |
| OCF_G_6m       | 0.000477  | 0.004718 | 0.002657           |
| gpm_semi_YoY   | -0.01823  | -0.15064 | -0.00103           |
| BP             | -0.06515  | -0.46891 | -0.02186           |
| volume         | NaN       | NaN      | 0.000407           |
| month_return   | NaN       | NaN      | 0.000415           |
| amplitude      | NaN       | NaN      | 0.00453            |
| turnover_rate  | NaN       | NaN      | -0.01206           |
| EP_ttm         | NaN       | NaN      | -0.00426           |
| EPcut_ttm      | NaN       | NaN      | -0.00427           |
| SP_ttm         | NaN       | NaN      | -0.00635           |
| PE             | NaN       | NaN      | 0.003626           |
| PCF            | NaN       | NaN      | 0.007281           |
| PS             | NaN       | NaN      | 0.014056           |
| PEG            | NaN       | NaN      | 0.001339           |
| Sales_G_ttm    | NaN       | NaN      | 0.004828           |
| Profit_G_ttm   | NaN       | NaN      | 0.00594            |
| OCF_G_ttm      | NaN       | NaN      | 0.012985           |
| ROE_G_ttm      | NaN       | NaN      | NaN                |
| ROE_G_6m       | NaN       | NaN      | NaN                |
| roe_ttm        | NaN       | NaN      | 0.006677           |
| roe_6m_YoY     | NaN       | NaN      | NaN                |
| roa_ttm        | NaN       | NaN      | 0.004257           |
| roa_YoY        | NaN       | NaN      | NaN                |
| gpm_ttm        | NaN       | NaN      | 0.007752           |
| cfo_to_np_ttm  | NaN       | NaN      | 0.002161           |
| roic_ttm       | NaN       | NaN      | 0.006625           |

</div>

$AvgIC$ and $\text{IC}\underline{}\text{IR}$ of several factors are marked as NaN. This is because there is no monthly cross-section within the backtesting period satisfies the minimum sample-size requirement simultaneously.

## 4.  Correlation and multicollinearity of surviving factors  
### 4.1 Correlation testing methodology  
Pearson correlation coefficients are adopted to measure the degree of linear association between distinct factors, identifying information redundancy and evaluating factor independence.

$$\rho(F_{a},F_{b}) = \frac{Cov(F_{a},F_{b})}{\sqrt{Var(F_{a})}\cdot\sqrt{Var(F_{b})}}$$

Cov denotes covariance, and Var represents factor variance.

&emsp; $\rho=1$: perfect positive linear correlation  
&emsp; $\rho=-1$: perfect negative linear correlation  
&emsp; $\rho=0$: no linear correlation  

### 4.2 Correlation matrix analysis
We follow conventional criteria widely adopted in multi-factor quantitative research:  
&emsp; |ρ|>0.7: High linear correlation with substantial information redundancy  
&emsp; 0.3<|ρ|≤0.7: Moderate correlation indicating partial information overlap  
&emsp; |ρ|≤0.3: Weak or negligible correlation, implying independent factor dimensions  
Pearson correlation matrix for the three valid factors:  

 <div align="center">
</div>
 <div align="center">
  
|               | BP      | PB      | gpm_semi_YoY |
|---------------|---------|---------|--------------|
| BP            | 1.0000  | -0.8362 | -0.1003      |
| PB            | -0.8362 | 1.0000  | 0.1201       |
| gpm_semi_YoY  | -0.1003 | 0.1201  | 1.0000       |

</div>

Result analysis:  
Strong negative correlation exists between sub-factors within the same valuation style group.  
The correlation coefficient between BP and PB reaches −0.8362, representing substantial information redundancy, as the two metrics are mathematical reciprocals of each other.  
The absolute correlation values between "gpm_semi_YoY"  and valuation factors (BP, PB) are both below 0.2. No significant collinearity is observed, which means this factor delivers independent incremental information.

### 4.3 Multicollinearity
We identify multicollinearity based on the threshold |ρ|>0.7 for Pearson correlation coefficients. The test outcomes are summarised below:

 <div align="center">
</div>
 <div align="center">
  
| Factor 1 | Factor 2 | Correlation Coefficient | Test Result                     |
|----------|----------|--------------------------|---------------------------------|
| BP       | PB       | -0.8362                  | Multicollinearity detected      |

</div>

Only one pair of high-correlation factors is found across the full sample, and the redundancy is confined to sub-factors within the valuation category. Cross-style factors (valuation and profitability quality metrics) exhibit no substantial information overlap and are free of multicollinearity risks.  
If BP, PB and "gpm_semi_YoY"  are directly incorporated simultaneously into the weighted multi-factor model, BP and PB will offset each other. The two factors carry signals with opposite directions and highly overlapping information, which undermines the predictive power of the composite factor.  

### 4.4 Correlation analysis and factor combination
&emsp; 1)	Sub-factors within the same style bucket are prone to multicollinearity. BP and PB are homogeneous valuation metrics exhibiting a strong negative linear correlation; only one of the two shall be retained. 
&emsp; 2)	Cross-style factors offer diversification benefits in information space. Valuation and profitability quality factors show minimal linear correlation. Integrating these two dimensions diversifies drawdown risks inherent to single-style strategies and enhances the long-run return stability of the composite factor.














