# Macroeconomic-PD-Satellite-Model
R implementation of an IFRS9 Satellite Model using ARDL and Bayesian Model Averaging (BMA). Features stationarity testing, dynamic Default Rate projections under Baseline, Adverse, and Optimistic macroeconomic scenarios, and calculation of the Vasicek systemic risk factor (Z-shift) for PiT PD estimation

# IFRS9 Macro-Satellite Model: ARDL & Bayesian Model Averaging

This repository hosts a robust **R framework** for developing a **Forward-Looking IFRS9 Satellite Model**. 
The project estimates the relationship between macroeconomic variables (GDP, Unemployment, Inflation, Interest Rates) and Portfolio Default Rates to generate **Point-in-Time (PiT) PD projections** under multiple economic scenarios (Baseline, Adverse, Optimistic).

## 📋 Project Overview

The core objective is to translate macroeconomic forecasts into credit risk parameters. The methodology utilizes an **Autoregressive Distributed Lag (ARDL)** approach, enhanced by **Bayesian Model Averaging (BMA)** to mitigate model risk and ensure stability in projections.

Key features:
- **Stationarity Analysis:** Automated ADF testing and differencing.
- **Advanced Feature Selection:** Multivariate grid search with VIF and economic sign constraints.
- **Model Averaging:** BMA based on BIC (Bayesian Information Criterion).
- **Stress Testing:** VAR-based scenario generation with Covid-adjusted volatility.
- **IFRS9 Parameters:** Calculation of the Vasicek Systemic Risk Factor ($Z$-shift).

---

## 🛠️ Methodological Workflow

The code executes the following logical steps:

### 1. Data Pre-processing & Transformation
* **Logit Transformation:** Historical Default Rates ($DR$) are transformed into a generic index $Z$ (Log-odds) to linearize the relationship and map the $(0,1)$ domain to $(-\infty, +\infty)$.
    $$Z_t = \ln\left(\frac{DR_t}{1 - DR_t}\right)$$
* **Stationarity Testing:** The **Augmented Dickey-Fuller (ADF)** test is applied to all variables. Non-stationary series are automatically differenced (e.g., $\Delta$ Unemployment) to avoid spurious regressions.

### 2. Feature Selection & Model Search
* **Univariate Filtering:** Variables are screened based on:
    * **Economic Sign:** e.g., Positive correlation with Unemployment, Negative with GDP.
    * **Statistical Significance:** p-values check.
    * **Residual Diagnostics:** Ljung-Box (autocorrelation) and Jarque-Bera (normality) tests.
* **Multivariate Grid Search:** The algorithm tests combinatorial models (up to $k$ variables) with different lag structures.
* **Multicollinearity Check:** Models with high **VIF** (Variance Inflation Factor) are discarded.

### 3. Bayesian Model Averaging (BMA)
Instead of relying on a single "best" model, the framework applies **BMA**:
* Valid models are weighted based on their **Posterior Probability** (derived from BIC).
* Final coefficients are a weighted average, capturing both the autoregressive inertia ($Z_{t-1}$) and the impact of macro variables.

### 4. Macro Scenario Generation (VAR)
* **Vector Autoregression (VAR):** Estimated on historical macro data to capture interdependencies.
* **Covid-Adjustment:** A dummy variable strategy is used to isolate the 2020-2021 volatility.
* **Shock Calculation:** *Adverse* and *Optimistic* scenarios are defined using the 1st and 99th percentiles of the historical residuals (re-incorporating Covid volatility for a prudent stress test).

### 5. Forward-Looking Projections
The BMA coefficients are applied to future macro scenarios (2025-2027) using a **dynamic step-by-step projection loop**. This calculates the Default Rate for $t+1$ using the estimated $Z_t$ as the new lag for $t+2$.

### 6. IFRS9 Z-Shift Estimation
The model calculates the **Systemic Risk Factor ($Z$)** following the **Vasicek / Carlehed & Petrov** framework:
1.  Extraction of the historical **Asset Correlation ($\rho$)**.
2.  Derivation of the $Z_t$ vector for each future scenario.
    $$Z_t = \frac{\Phi^{-1}(LRA) - \Phi^{-1}(ODR_t) \cdot \sqrt{1 - \rho}}{\sqrt{\rho}}$$
This factor is essential for shifting Through-The-Cycle (TTC) Transition Matrices into Point-in-Time (PiT) matrices.
