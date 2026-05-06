# 📉 Forecasting Recession Probability: A Time Series Approach with Macroeconomic and Behavioral Signals

> Predicting U.S. recession probability using autoregressive models, FRED economic indicators, and Google Search Trends sentiment data.

---

## 📖 Overview

This project explores whether recession probability in the United States can be forecasted by combining **time series modeling**, **macroeconomic indicators**, and **public sentiment data**. Using the Hamilton GDP-Based Recession Index as a continuous target variable, the study evaluates how well economic features and behavioral signals (Google Trends) predict recession risk.

Key findings:
- Recession probability is **highly persistent** and largely governed by its own past values
- The **AR(2) model** achieves strong in-sample performance (R² ≈ 0.93) with actual prior values, but collapses under recursive forecasting
- **Unemployment rate, yield spread, and federal funds rate** are the strongest one-month leading indicators
- **Google Trends data consistently reduces forecast accuracy**, suggesting it introduces noise rather than signal

---

## 📊 Dataset

### Table 1: Collected Data Description and Frequencies

Data spans **January 2004 – December 2025** (264 monthly observations) and is sourced from two publicly available platforms — FRED and Google Trends.

| Variable | FRED Series ID | Description | Frequency |
|---|---|---|---|
| Real GDP Growth | GDPC1 | Annualized quarterly growth rate (%) | Quarterly |
| CPI Inflation | CPIAUCSL | Year-over-year change in consumer prices (%) | Monthly |
| Federal Funds Rate | FEDFUNDS | Target interest rate set by the Fed (%) | Monthly |
| 10Y–2Y Yield Spread | T10Y2Y | Long minus short Treasury rate (%) | Monthly |
| Unemployment Rate | UNRATE | Share of labor force unemployed (%) | Monthly |
| Real PCE Growth | PCECC96 | Year-over-year consumer spending growth (%) | Monthly |
| Hamilton Index | JHGDPBRINDX | Recession probability (0–100) | Quarterly |
| Sahm Rule Index | SAHMREALTIME | Recession indicator (%) | Monthly |
| NBER Recession | USREC | Binary recession flag used for chart shading only | Monthly |
| Search: recession | Google Trends | Monthly relative search interest (0–100) | Monthly |
| Search: unemployment | Google Trends | Monthly relative search interest (0–100) | Monthly |
| Search: layoffs | Google Trends | Monthly relative search interest (0–100) | Monthly |
| Search: economic crisis | Google Trends | Monthly relative search interest (0–100) | Monthly |
| Search: inflation | Google Trends | Monthly relative search interest (0–100) | Monthly |

---

## 🛠️ Methodology

### 1. Data Collection & Preprocessing
- FRED data collected via the **FRED API**
- Google Trends data extracted using **pytrends**
- Quarterly series forward-filled to monthly frequency
- Derived features: Year-over-year inflation (from CPI), PCE growth, annualized GDP growth
- Logit transformation applied to the Hamilton Index to constrain predictions to [0, 100]

### 2. Feature Engineering
```
GDP_growth   = GDP × pct_change(1) × 400
PCE_Growth   = PCE × pct_change(12) × 100
Inflation    = CPI × pct_change(12) × 100
```

### 3. Models Evaluated

| Model | Description |
|---|---|
| **Baseline** | Uses t-2 value as the current prediction |
| **AR(2) — Actual Inputs** | Autoregressive model using true prior recession values |
| **AR(2) — Recursive** | Autoregressive model using its own predicted values |
| **Feature-Only (OLS)** | Economic/trend features with no AR term |
| **Augmented AR(2)** | AR(2) combined with economic ± Google Trends features |

### 4. Train / Test Split

#### Table 2: Train/Test Split Summary

| Dataset | Split | Start Date | End Date | Observations | Period |
|---|---|---|---|---|---|
| With Google Trends | Train | 2005-04-30 | 2021-10-31 | 199 | ~16.5 years |
| With Google Trends | Test | 2021-11-30 | 2025-12-31 | 50 | ~4.1 years |
| Without Google Trends | Train | 1991-04-30 | 2018-12-31 | 333 | ~27.7 years |
| Without Google Trends | Test | 2019-01-31 | 2025-12-31 | 84 | ~7 years |

---

## 📈 Results

### Table 3: Baseline Model Results (No Additional Features)

| Metrics | Base Model (t-2 value is the current value) | AR(2) (Taking Actual Previous Values) | AR(2) (Taking Predicted Values) |
|---|---|---|---|
| **Full Dataset (1991–2025)** | | | |
| R² Train | — | 0.9875 | 0.9875 |
| R² Test | **0.5956** | **0.9278** | **-0.0744** |
| RMSE | 14.93 | 6.31 | 24.34 |
| MAE | 6.74 | 2.54 | 13.81 |
| **Limited Dataset (2004–2025)** | | | |
| R² Train | — | 0.9592 | 0.9592 |
| R² Test | **0.4886** | **0.9148** | **-0.4111** |
| RMSE | 6.39 | 2.61 | 10.61 |
| MAE | 3.46 | 1.46 | 7.77 |

---

### Table 4: Model Results with Economic and Google Search Trend Data (Hamilton Index)

| Metrics | Economic Features (Logit-OLS) | AR(2) + Economic Features (Taking Actual Previous Values) | AR(2) + Economic Features (Taking Predicted Values) |
|---|---|---|---|
| **Without Google Trends** | | | |
| *Uniform Lag (1)* | | | |
| R² Train | 0.7464 | 0.9888 | 0.9888 |
| R² Test | **0.4266** | **0.9178** | **-1.9393** |
| RMSE | 17.78 | 6.73 | 40.25 |
| MAE | 13.41 | 3.16 | 29.87 |
| *Custom Lag* | | | |
| R² Train | 0.7901 | 0.9866 | 0.9866 |
| R² Test | **0.0931** | **0.9261** | **-0.2915** |
| RMSE | 22.57 | 6.44 | 26.94 |
| MAE | 17.47 | 2.91 | 21.07 |
| **With Google Trends** | | | |
| *Uniform Lag (1)* | | | |
| R² Train | 0.7075 | 0.9715 | 0.9715 |
| R² Test | **-1.6497** | **0.8260** | **-1.2431** |
| RMSE | 14.54 | 3.73 | 13.38 |
| MAE | 11.37 | 2.64 | 7.44 |
| *Custom Lag* | | | |
| R² Train | 0.8092 | 0.9749 | 0.9749 |
| R² Test | **-6.7745** | **0.7552** | **-13.8083** |
| RMSE | 25.29 | 4.49 | 34.90 |
| MAE | 19.86 | 2.85 | 24.03 |

---

### Table 5: Sahm Rule Recession Index Results — Without Google Trends

| Description | R² (Train) | R² (Test) | RMSE (Test) | MAE (Test) |
|---|---|---|---|---|
| Baseline | — | **0.5530** | 1.2638 | 0.4772 |
| AR(2) (Taking actual previous values) | 0.9855 | **0.9081** | 0.5731 | 0.2007 |
| AR(2) (Taking predicted previous values) | 0.9855 | **-0.09** | 1.9734 | 0.8371 |
| Macro-Only | 0.5603 | **-0.124** | 2.0038 | 1.0989 |
| AR(2) + Macro (Taking actual previous values) | 0.9872 | **0.9220** | 0.5279 | 0.1914 |
| AR(2) + Macro (Taking predicted previous values) | 0.9872 | **0.4877** | 1.3530 | 0.8252 |

---

### Table 6: Sahm Rule Recession Index Results — With Google Trends

| Description | R² (Train) | R² (Test) | RMSE (Test) | MAE (Test) |
|---|---|---|---|---|
| Baseline | — | **0.8184** | 0.0973 | 0.0806 |
| AR(2) (Taking actual previous values) | 0.9401 | **0.881** | 0.0789 | 0.0620 |
| AR(2) (Taking predicted previous values) | 0.9401 | **-2.223** | 0.4098 | 0.3702 |
| Macro-Only | 0.8805 | **-57.36** | 1.7440 | 1.3874 |
| AR(2) + Macro + Trends (Taking actual previous values) | 0.9705 | **-0.252** | 0.2554 | 0.2160 |
| AR(2) + Macro + Trends (Taking predicted previous values) | 0.9705 | **-82.58** | 2.0870 | 1.9053 |

---

## 🔍 Key Findings

- **Recession dynamics are highly persistent** — the baseline model alone explains ~50–60% of variance
- **Lagged correlation analysis** confirms that unemployment, yield spread, and federal funds rate are the strongest **one-month leading indicators**
- **Strong in-sample ≠ strong out-of-sample**: AR(2) with actual inputs excels (R² ≈ 0.93), but collapses to negative R² under recursive forecasting
- **Google Trends hurts performance** across all configurations — sentiment data introduces noise in this framework
- **Sahm Rule Index** shows similar behavior to the Hamilton Index, with a slight advantage in recursive forecasting when macro features are added (R² = 0.49)

---

## ⚠️ Limitations

- Google Trends data is only available from **2004 onwards**, limiting the training window and the number of observable recession events
- The Hamilton Index is released **quarterly**, requiring interpolation to monthly frequency, which may artificially simplify prediction
- **Custom lag selection** was computed on the full dataset rather than training data only, introducing data leakage and overfitting

---

## 🧰 Tools & Libraries

- **Python** — pandas, numpy, statsmodels, sklearn, matplotlib, seaborn
- **Data Sources** — FRED API (`fredapi`), Google Trends (`pytrends`)
- **Modeling** — AR(2), OLS regression with logit-transformed target
