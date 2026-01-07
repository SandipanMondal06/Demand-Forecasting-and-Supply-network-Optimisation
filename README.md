# Demand Forecasting and Supply–Transport Cost Optimization

## 1. Project Overview

This project builds a **complete supply chain analytics pipeline** that combines:

1. **Synthetic demand generation** for multiple products and stores  
2. **Time series diagnostics and forecasting** using statistical models  
3. **Multi-echelon transportation cost modeling** from plants → DCs → stores  

The objective is to create a **realistic, scalable demand forecasting foundation** that can later be integrated with **optimization models (LP/MIP, network flow, VRP)**.

This repository is designed as a **research-grade / portfolio-grade implementation**, not a toy example.

---

## 2. Demand Data Generation

### 2.1 Dimensions of the Dataset

| Component | Count |
|---------|------|
| Weeks | 100 |
| Products | 20 |
| Stores | 10 |
| Total Time Series | 200 |
| Total Rows | 20,000 |

Each **(store, product)** pair has its **own independent time series**.

---

### 2.2 Demand Construction Logic

Demand for each `(store, product)` is generated using the following additive structure:

\[
\text{Demand}_t = \text{Base} + \text{Trend}_t + \text{Seasonality}_t + \text{Noise}_t
\]

#### (a) Base Demand
- Random integer between **30 and 100**
- Represents baseline consumption level

#### (b) Trend Component
- Linear trend coefficient sampled from **Uniform(-0.5, 1.5)**
- Allows:
  - Declining demand
  - Stable demand
  - Growing demand

\[
\text{Trend}_t = \beta \cdot t
\]

#### (c) Seasonality Component
- **Sinusoidal seasonality**
- Period = **4 weeks** (monthly demand cycles)
- Random amplitude and phase

\[
\text{Seasonality}_t = A \cdot \sin\left(\frac{2\pi t}{4} + \phi\right)
\]

This creates **store–product-specific seasonal behavior**.

#### (d) Noise Component
One of the following is randomly selected **per time series**:
- Gaussian noise (demand volatility)
- Poisson noise (count-like randomness)
- Exponential noise (right-skewed shocks)

This avoids unrealistic homogeneity across products.

#### (e) Post-processing
- Negative demand clipped to zero
- Converted to integer demand units

---

### 2.3 Output

The final dataset:

| Column | Description |
|------|-------------|
| `week` | Weekly timestamp |
| `store` | Store identifier |
| `product` | Product identifier |
| `demand` | Realized demand |

Saved as: demand_data.csv


---

## 3. Exploratory Time Series Analysis

### 3.1 Visualization

A single `(store, product)` time series is plotted to visually confirm:
- Trend presence
- Seasonal oscillations
- Noise behavior

This step is **sanity validation**, not modeling.

---

## 4. Stationarity Testing (ADF Test)

### 4.1 Why Stationarity Matters

Most classical time-series models (ARIMA, SARIMA) assume **stationary input**:
- Constant mean
- Constant variance
- Stable autocorrelation

Non-stationary data must be **differenced**.

---

### 4.2 Augmented Dickey–Fuller (ADF) Test

- Null hypothesis: **Series has a unit root (non-stationary)**
- Alternative: **Series is stationary**
- Significance level: **5%**

Decision rule:
p-value < 0.05 → Stationary
p-value ≥ 0.05 → Non-stationary


---

### 4.3 Results

- **All 200 time series are non-stationary**
- This is expected due to:
  - Trend
  - Seasonality

This confirms the **need for differencing and seasonal modeling**.

---

## 5. Supply Chain Cost Modeling

This project models a **two-stage supply network**: Plants → Distribution Centers → Stores


---

### 5.1 Network Structure

| Entity | Count |
|------|------|
| Plants | 3 |
| Distribution Centers | 5 |
| Stores | 10 |
| Products | 20 |

---

### 5.2 Plant → DC Cost Matrix

Each row represents:
- A plant
- A DC
- A product
- Cost per unit shipped

Costs:
- Random uniform between **1.0 and 5.0**


---

### 5.1 Network Structure

| Entity | Count |
|------|------|
| Plants | 3 |
| Distribution Centers | 5 |
| Stores | 10 |
| Products | 20 |

---

### 5.2 Plant → DC Cost Matrix

Each row represents:
- A plant
- A DC
- A product
- Cost per unit shipped

Costs:
- Random uniform between **1.0 and 5.0**


This dataset supports:
- Multi-commodity flow
- Product-specific transportation costs

---

### 5.3 DC → Store Cost Matrix

Each row represents:
- A DC
- A store
- A product
- Cost per unit shipped

Costs:
- Random uniform between **1.0 and 3.0**


---

## 6. Time Series Forecasting (SARIMA)

### 6.1 Why SARIMA?

Demand exhibits:
- Trend
- Seasonality
- Autocorrelation

SARIMA explicitly models:

\[
\text{SARIMA}(p,d,q)(P,D,Q)_s
\]

Where:
- \(p,q\): AR & MA terms
- \(d\): Trend differencing
- \(P,Q\): Seasonal AR & MA
- \(D\): Seasonal differencing
- \(s = 4\): Weekly seasonality

---

### 6.2 Model Selection via `auto_arima`

For each `(store, product)`:

- Seasonal modeling enabled
- Season length `m = 4`
- Automatic selection of:
  - `d`, `D`
  - `(p, q)`
  - `(P, Q)`
- Stepwise search to reduce computational cost
- AIC minimization objective

Intercept is explicitly included to capture baseline demand.

---

### 6.3 Safeguards

- Minimum series length enforced
- Error handling for unstable parameter combinations
- Invalid models skipped

---

### 6.4 Output Stored per Series

| Parameter | Meaning |
|---------|--------|
| `p,d,q` | Non-seasonal ARIMA |
| `P,D,Q,s` | Seasonal ARIMA |
| `AIC` | Model quality metric |

This allows:
- Model comparison
- Meta-analysis across stores/products
- Downstream ensemble logic

---

## 7. Interpretation of Results

### 7.1 Observations

- Different products require **different differencing orders**
- Seasonal differencing (`D=1`) is common
- Some series collapse to pure seasonal models
- Others require richer ARMA dynamics

This reflects **realistic retail heterogeneity**.

---

## 8. How This Extends to Optimization

This project intentionally **stops before optimization**, but enables:

### 8.1 Deterministic Optimization
- Forecasted demand → constraints
- Transport costs → objective
- LP / MIP formulation

### 8.2 Stochastic Optimization
- Scenario-based demand forecasts
- Chance constraints

### 8.3 Network Flow / VRP
- Multi-commodity flows
- Store-level replenishment

---

## 9. Key Strengths of This Project

- Realistic synthetic data (not Kaggle copy)
- Correct statistical diagnostics
- Scalable modeling across 200 time series
- Clean separation of forecasting and cost modeling
- Direct extensibility to OR problems

---

## 10. Next Logical Extensions

- Forecast error evaluation (MAPE, RMSE)
- Rolling-origin backtesting
- Ensemble forecasting
- Demand-driven inventory optimization
- Multi-period transport optimization

---

## 11. Conclusion

This repository demonstrates **end-to-end supply chain analytics thinking**:
- Statistical rigor
- Engineering scalability
- Optimization readiness

It is suitable for:
- Research portfolios
- Applied ML + OR roles
- Graduate-level projects





