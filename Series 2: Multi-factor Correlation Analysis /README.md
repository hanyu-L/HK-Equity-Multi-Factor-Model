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
1)	When only an annual report is available for the current period, TTM is derived directly from this full-year data.  
2)	When a semi-annual or quarterly report is disclosed, the 12-month rolling metric is reconstructed via: Last full-year figure − same-period figure from last year + current single-period result. Matching historical filings of the identical category and the prior full-year annual report is mandatory.  
3)	The TTM value becomes NaN if the matching prior-period report or prior annual report cannot be located. All financial factors are filled with NaN for stocks lacking any financial disclosures.  

### Valuation factors
#### 1) Earnings-to-price ratios
$$EP_{ttm} = \frac{NetProfit_{ttm}}{MarketCap}$$

$EPcut_{ttm}$: deducted non-recurring earnings per market cap  

$$EPcut_{ttm} = \frac{NetProfit\ Excluding\ Non-recurring\ Items_{TTM}}{Total\ market\ cap}$$

#### 2) Asset-to-price ratios
BP: book value per market cap

$$BP = \frac{shareholders' equity}{total market cap}$$

$SP_{ttm}$: sales revenue per market cap

$$SP_{ttm} = \frac{revenue_{ttm}}{total market cap}$$

#### 3) Inverted valuation multiples
PE: price earnings ratio

$$PE = \frac{total market cap}{net revenue_{ttm}}$$

PB: price book ratio

$$PB = \frac{total market cap}{shareholders' equity}$$

PCF: price cash flow ratio

$$PCF = \frac{total market cap}{operating cash flow_{ttm}}$$

PS: price sales ratio

$$PS = \frac{total market cap}{revenue_{ttm}}$$

### Growth-value alignment
PEG: price earnings growth. Assign NaN when the growth rate is zero.

$$PEG = \frac{PE}{{sales\_g}_{ttm}}$$

### Growth factors
#### 1） TTM year-on-year growth
Match the TTM financial statements corresponding to the fiscal closing date lagged by one full year. Observations without complete prior-year financial reports are set to missing.
General formula:

$$G_{ttm} = \frac{X_{ttm,current}}{X_{{ttm,last\_year}}} - 1$$

$X_{ttm,current}$ = current-period trailing 12-month metric derived from financial statements  
$X_{{ttm,last\_year}}$ = trailing 12-month metric recorded at the identical fiscal closing date of the prior year.  

If the prior-year TTM value is missing or equals zero, the factor is assigned np.nan.
-------------------开始有问题
Sales_G_ttm: TTM sales year-on-year growth

$$Sales\_G_{ttm} = \frac{rev_{ttm,current}}{rev_{\mathit{ttm,last\_year}}} - 1$$

Profit_G_ttm: TTM net profit year-on-year growth

$$Profit\_G_{ttm} = \frac{net\_profit_{ttm,current}}{net\_profit_{\mathit{ttm,last\_year}}} - 1$$

OCF_G_ttm: TTM operating cash flow YoY growth

$$OCF\_G_{ttm} = \frac{ocf_{ttm,current}}{ocf_{\mathit{ttm,last\_year}}} - 1$$

ROE_G_ttm: TTM ROE year-on-year growth

$$ROE\_G_{ttm} = \frac{roe_{ttm,current}}{roe_{\mathit{ttm,last\_year}}} - 1$$


-----服了 一直改不好


#### 2） 	Semi-annual single-period YoY growth
This calculation applies exclusively to semi-annual and quarterly filings. Metrics are computed by matching financial statements of the identical reporting frequency from the previous year to derive year-on-year growth for single-period operating indicators and single-period ROE. Observations based on annual reports contain no semi-annual comparative data and are uniformly assigned np.nan.

Sale_G_6m: semi-annual sales year-on-year growth

$$Sale\_G_{6m} = \frac{Operating\ revenue_{current}}{Operating\ revenue_{last\_year}} - 1$$

Profit_G_6m: semi-annual net profit year-on-year growth

$$Profit\_G_{6m} = \frac{Profit\ attributable\ to\ equity\ holders_{current}}{Profit\ attributable\ to\ equity\ holders_{last\_year}} - 1$$

OCF_G_6m: semi-annual operating cash flow YoY Growth

$$OCF\_G_{6m} = \frac{netcash\ operate_{current}}{netcash\ operate_{last\_year}} - 1$$

ROE_G_6m: semi-annual ROE year-on-year growth

$$ROE\_G_{6m} = \frac{roe\ 6m\_YoY_{current}}{roe\ 6m\_YoY_{last\_year}} - 1$$


