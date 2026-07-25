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

