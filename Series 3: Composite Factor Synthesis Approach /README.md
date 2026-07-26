# Composite Factor Synthesis Approach  
Following the prior series 2, this report performs factor synthesis based on pre-processed factors.

1. Methodology Design  
1.1 Equal-weighted Synthesis  
1.2 Static IC_IR Weighted Synthesis  
1.3 Dynamic IC_IR Weighting with Half-Life Decay  
1.4 Principal Component Analysis (PCA) Method  
1.5 Return-Based Dynamic Weighting with Half-Life Decay  
1.6 IC Maximization of Average IC Weighting  
1.7 Maximization of IC_IR Weighting  
2.	Empirical Results  

## 1. Methodology Design  
Factor synthesis adopts three valid factors that have undergone industry and market capitalisation neutralisation in the previous chapter. These factors are classified by investment style: value factors including BP (Book-to-Price ratio) and PB (Price-to-Book ratio), alongside the profitability quality factor "gpm_semi_YoY"  (Semi-annual Gross Profit Margin YoY).  

### 1.1 Equal-weighted Synthesis
The composite factor is constructed by assigning equal weights to all candidate factors. When multiple style groups are involved, factors within each style are aggregated to form style-specific factors first, and cross-style factors are then equally weighted to generate the final composite factor.  
Within the value category, BP and PB exhibit severe multicollinearity. This study adopts principal component analysis (PCA) to extract the first principal component and construct the composite value factor. The composite value factor further equally weighted with the profitability quality factor "gpm_semi_YoY"  to obtain the cross-style composite factor. Factor validity tests are conducted subsequently.  
Backtesting results show that the equal-weighted composite factor achieves an average IC of 0.056, an "IC_IR"  of 0.3612, and a monthly average quintile long-short portfolio return of 0.0194 over the full sample period.  

### 1.2 Static IC_IR Weighted Synthesis
Factor weights are determined by the absolute value of each factor’s full-sample "IC_IR"  to construct the composite factor. Within the value category, BP and PB are weighted by their respective absolute "IC_IR"  values to form an aggregated value factor. The value indicator with stronger "IC_IR"  performance serves as the proxy for the value style, and cross-style weights between the value proxy and the profitability quality factor "gpm_semi_YoY"  are assigned based on absolute "IC_IR" . The cross-style composite factor is thereby constructed, followed by unified performance evaluation.  
The cross-style weights are reported as 0.757 for the aggregated value factor and 0.243 for the profitability quality factor. Backtesting results indicate that the static "IC_IR"  weighted composite factor yields an average IC of 0.0041, an "IC_IR"  of 0.0314, and a monthly average quintile long-short portfolio return of −0.0004 across the full sample period.

Result analysis:  
The absolute average IC of the composite factor falls below the validity threshold of 0.015, while its "IC_IR"  hovers near zero. The monthly long-short return is slightly negative, implying the factor cannot generate consistent excess returns.  
The poor performance stems from the strong negative correlation between BP and PB. Static weight allocation based on "IC_IR"  alone fails to fundamentally resolve the mutual offset of opposing signals.  
Compared with the equal-weighted approach, static weighting determines fixed weights using full-sample historical performance. Although more information is incorporated, this method suffers substantial in-sample overfitting risks, leading to noticeably weaker predictive power relative to equal-weighted synthesis.  

### 1.3 Dynamic IC_IR Weighting with Half-Life Decay  
For all candidate factors, the time-series IC within the rolling window is weighted via exponential half-life decay to calculate IC_IR. The absolute value of each factor’s real-time IC_IR serves as the weighting criterion, and weights are reallocated monthly to construct the dynamically updated composite factor.  
This study adopts a 36-month rolling estimation window with a half-life parameter H=24 months. The half-life weighting scheme follows the principle that observations closer to the target cross-section receive higher weights. Let gap denote the interval between a historical observation and the target cross-section. The raw decay weight is defined as:  

$$\mathit{w}_{gap} = 2^{-\frac{\mathit{gap}}{\mathit{H}}}$$

Normalization is implemented to obtain standardized weights:  

$$\mathit{w}_{gap}' = \frac{\mathit{w}_{gap}}{\sum \mathit{w}_{gap}}$$

Normalized weights are applied to compute the weighted average IC. The standard deviation of IC is estimated from the raw unweighted IC time series to derive the real-time IC_IR.  
Within the value group, BP and PB are dynamically weighted by the absolute value of real-time IC_IR to form an aggregated value factor. The value indicator with higher absolute IC_IR acts as the proxy for the value style. Cross-style weights between this value proxy and the profitability quality factor "gpm_semi_YoY" are assigned according to absolute IC_IR, generating the composite factor on a monthly basis. Boundary checks for denominators are embedded in the code to avoid division-by-zero errors under extreme conditions. Uniform performance evaluation is conducted after composite factor construction.  

Result analysis:  
The composite factor constructed via dynamic IC_IR weighting with a 24-month half-life fails to deliver satisfactory performance. Backtesting shows that the composite factor achieves an average IC of −0.0039, an "IC_IR"  of −0.0307, and a monthly average quintile long-short portfolio return of 0.0004 over the full sample period, indicating its inability to generate persistent excess returns.  

### 1.4 Principal Component Analysis (PCA) Method  
Principal Component Analysis (PCA), proposed by Pearson in 1901, is a widely adopted unsupervised dimensionality reduction technique. PCA projects a set of highly correlated N-dimensional factors onto a new k-dimensional coordinate system where k<N. The resulting k orthogonal features are termed principal components, which are mutually uncorrelated, enabling dimensionality reduction and removal of redundant information. Consider a factor matrix $$(\mathit{x}_{1},\mathit{x}_{2},\dots,\mathit{x}_{N})_{\mathit{T}\times \mathit{N}}$$ consisting of N factors, where each x_i denotes a T-dimensional column vector. The implementation steps are outlined as follows:  
&emsp; 1) Standardise the raw factor matrix;  
&emsp; 2) Compute the covariance matrix of factors and derive its corresponding eigenvalues and eigenvectors  
&emsp; 3) Sort eigenvalues in descending order and select the first k principal components whose cumulative explained variance ratio exceeds 85%  
&emsp; 4) Weight the selected principal components by their respective variance explained ratios and aggregate them linearly to construct the global PCA composite factor  
In this experiment, the first two principal components satisfy the 85% variance threshold, delivering a cumulative explained variance ratio of 0.946. PCA performs linear transformation purely based on the cross-sectional distribution of factor values, without incorporating predictive information such as forward returns or factor IC.  
Backtesting results reveal that the global PCA composite factor achieves an average IC of 0.0661, an "IC_IR"  of 0.4216, and a monthly average quintile long-short portfolio return of 0.0207 over the full sample period.  
The global PCA composite factor exhibits the strongest return predictability. By jointly reducing the dimensionality of value and profitability quality factors, orthogonal transformation eliminates multicollinearity between BP and PB while preserving independent information embedded in the two investment styles. Unlike IC_IR weighting and other alternative schemes, PCA requires no manual style grouping and does not rely on historical performance to assign weights.  


### 1.5 Return-Based Dynamic Weighting with Half-Life Decay  
The return-based dynamic weighting method with half-life decay determines factor weights using exponentially weighted historical quintile long-short returns within a rolling window, and constructs the composite factor cross-section by cross-section. This study adopts a 36-month rolling estimation window with half-life parameter H=24 months. The half-life weighting scheme follows the principle that observations closer to the target cross-section carry larger weights. Let gap denote the interval between a historical observation and the target cross-section. The raw decay weight is defined as:  

$$w_{gap} = 2^{-\frac{gap}{H}}$$

Raw weights are normalised to obtain standardised weights:

$$w_{gap}' = \frac{w_{gap}}{\sum w_{gap}}$$

We first compute monthly quintile long-short returns for each factor in every cross-section. Within the rolling window, historical long-short return series are weighted by the decay scheme to derive weighted average long-short returns. Weights for BP, PB and "gpm_semi_YoY"  are assigned proportional to the absolute value of weighted average returns to form the composite factor. Boundary constraints are embedded in the code to avoid division-by-zero errors.  
Backtesting outputs show that the composite factor constructed via return-based half-life weighting achieves an average IC of 0.0012, an "IC_IR"  of 0.0098, and a monthly average quintile long-short portfolio return of −0.0012.

### 1.6 IC Maximization of Average IC Weighting
The maximum IC weighting method originates from Qian’s Quantitative Equity Portfolio Management. Its core objective is to solve for a weight vector that maximizes the correlation between the composite factor and forward returns. The optimization formulation is specified as follows:

$$\max IC = \frac{\vec{w}^{\,T} \cdot \overrightarrow{IC}}{\sqrt{\vec{w}^{\,T} V \vec{w}}}$$

where $\vec{w}$ denotes the factor weight vector, $\overrightarrow{IC}$ stands for the vector of average factor IC values, and V represents the cross-sectional correlation matrix of factors. Given that all factors are standardized in advance, the correlation matrix is equivalent to the covariance matrix.  
To ensure sound economic interpretation and prevent short positions on raw factors, non-negativity constraints are imposed on weights:

$$
\begin{aligned}
\max \quad & \mathit{IC} \\
\text{s.t.} \quad & \vec{w} \ge 0 \\
& \sum w_i = 1
\end{aligned}
$$

This study constructs the optimization problem using the full valid sample, with BP, PB and "gpm_semi_YoY"  as candidate factors. Maximizing IC is transformed into minimizing the negative IC. Equal weights (1/3 for each factor) are adopted as initial guesses to solve for optimal static weights. The composite factor is formed via the linear combination of raw factors with optimized weights, followed by consistent performance evaluation.  
The solved optimal weights are "BP"=0.000, "PB"=1.000, and "gpm_semi_YoY"=0.000.  
Backtesting results show that the composite factor under maximum IC optimization achieves an average IC of 0.0685, an "IC_IR"  of 0.435, and a monthly average quintile long-short portfolio return of 0.0206 over the full sample period.  

Result analysis:  
The maximum IC optimized composite factor delivers the strongest predictive performance. The optimization assigns the full weight to PB, which renders the composite factor identical to the standalone PB factor, with slightly better performance than cross-style composite alternatives.  
Within the sample period, the profitability quality factor "gpm_semi_YoY"  exhibits weak predictive power, while PB dominates among value factors. In addition, BP and PB are strongly negatively correlated. Incorporating BP introduces signal offset, so the optimizer excludes BP and the profitability quality factor.  
Nevertheless, this approach bears notable limitations. Optimal weights are solved statically from full-sample data, which raises severe in-sample overfitting risks. Its out-of-sample predictive robustness may deteriorate amid shifts in market investment styles.  

### 1.7 Maximization of IC_IR Weighting
The maximum "IC_IR"  weighting method originates also from Qian’s Quantitative Equity Portfolio Management. Its core idea is to estimate the average IC and volatility of the composite factor using historical time-series IC values, and solve for a weight vector to maximize the composite factor’s "IC_IR" . The optimization objective is defined as:  

$$\max \mathit{IC}_{IR} = \frac{\vec{w}^{\,T} \cdot \overrightarrow{\mathit{IC}}}{\sqrt{\vec{w}^{\,T} \Sigma \vec{w}}}$$

where $\vec{w} denotes the factor weight vector, $\overrightarrow{IC}$ is the vector of full-sample average IC for each factor, and Σ represents the time-series covariance matrix of factor IC. To avoid negative weights and ensure economically meaningful factor interpretation, constrained optimization is implemented with the following restrictions:

$$
\begin{aligned}
\max \quad & \mathit{IC}_{IR} \\
\text{s.t.} \quad & \vec{w} \ge 0 \\
& \sum w_i = 1
\end{aligned}
$$

This study takes BP, PB and "gpm_semi_YoY"  as candidate factors. The maximization of "IC_IR"  is transformed into minimizing the negative "IC_IR" . Equal weights serve as initial guesses to solve for the global static optimal weights. The composite factor is constructed via the linear combination of raw factors with optimized weights, followed by consistent performance evaluation.  
The solved optimal weights are "BP"=0.000, "PB"=1.000, and "gpm_semi_YoY"=0.000.  
Backtesting results show that the composite factor under maximum "IC_IR"  optimization achieves an average IC of 0.0685, an "IC_IR"  of 0.435, and a monthly average quintile long-short portfolio return of 0.0189 over the full sample period.  
The maximum "IC_IR"  optimization scheme delivers reliable predictive power, yet it may yield extreme weight allocations, where the composite factor degenerates into the standalone PB factor. Within the sample period, PB outperforms other factors in predictive performance. Incorporating BP and "gpm_semi_YoY"  introduces signal offset and lowers the overall "IC_IR" , so the optimizer assigns all weights to PB.  
This method computes static optimal weights based on full-sample historical performance, which induces substantial in-sample overfitting risks. The extreme weight structure abandons multi-style diversification, making the composite factor less robust amid shifts in market investment styles.

## 2. Empirical Results
Candidate synthesis factors: BP, PB, "gpm_semi_YoY"  
Rolling window: 36 months  
Half-life: 24 months  
This study implements six composite factor synthesis approaches under a unified monthly cross-sectional backtesting framework. We conduct horizontal performance comparison based on average IC, "IC_IR"  and monthly average quintile long-short portfolio returns.  
Backtesting performance exhibits clear stratification across methods. Composite factors constructed via maximization of IC, maximization of "IC_IR" , global PCA dimensionality reduction and equal-weighted synthesis generate persistent valid predictive signals. In contrast, three linear weighting schemes, namely static "IC_IR"  weighting, return-based dynamic weighting with half-life decay and dynamic "IC_IR"  weighting with half-life decay, deliver weak performance and cannot reliably produce excess returns.  
Performance divergence stems primarily from whether the multicollinearity between value sub-factors BP and PB is properly addressed. Static and dynamic weighting schemes based on IC time series merely allocate weights according to historical performance and fail to resolve persistent mutual offset of opposing signals between the two correlated value factors, leading to continuous erosion of value-style information. By contrast, PCA preprocessing purifies value-related information and effectively mitigates signal offset. The optimized weights from two weight-maximization models concentrate entirely on the single factor PB, indicating that cross-style factor fusion fails to generate tangible performance gains within the examined sample interval. The above evidence suggests that eliminating intra-style factor redundancy prior to constructing cross-style composite factors constitutes a critical prerequisite for ensuring the predictive power of composite factors.





