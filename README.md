# 🏥 NHS A&E Attendances — Time Series Analysis & Forecasting

> Analysis of 15 years of NHS England A&E attendance data to identify seasonal patterns, long-run trends, and forecast future demand using SARIMA time-series modelling.

[![Python](https://img.shields.io/badge/Python-3.x-blue?logo=python)](https://python.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Active-brightgreen)]()

**[Notebook](ae-analysis.ipynb)**

---

## Table of contents

- [Project overview](#project-overview)
- [Key results](#key-results)
- [Dataset](#dataset)
- [Project structure](#project-structure)
- [Methodology](#methodology)
- [Key findings](#key-findings)
- [How to run locally](#how-to-run-locally)
- [Limitations and future work](#limitations-and-future-work)
- [Acknowledgements](#acknowledgements)

---

## Project overview

NHS A&E departments face significant seasonal demand fluctuations and a post-COVID surge in attendances, making workforce and resource planning extremely challenging. This project analyses 15 years of real NHS England attendance data to identify seasonal patterns and long-run trends, and builds a SARIMA forecasting model to predict future monthly attendances. Confidence intervals are included in all forecasts to honestly communicate prediction uncertainty — a critical requirement for any model used in real healthcare planning.

The project is relevant to anyone working in health data analytics, NHS planning, or public sector data science.

---

## Key results

| Metric | Value |
|---|---|
| Data range | August 2010 — March 2026 |
| Total months analysed | 188 |
| Forecast horizon | 12 months ahead |
| MAE | 145,612 attendances |
| RMSE | 157,889 attendances |
| MAPE | 6.3% |

> A MAPE of 6.3% means the model's monthly forecasts are on average within 6.3% of the actual figure — reasonable performance for a baseline SARIMA model on data containing a major structural break (COVID-19).

---

## Dataset

### Source

| Dataset | Source | Format | Licence |
|---|---|---|---|
| Monthly A&E Attendances and Emergency Admissions Time Series | [NHS England](https://www.england.nhs.uk/statistics/statistical-work-areas/ae-waiting-times-and-activity/) | XLS | Open Government Licence v3 |

The dataset is publicly available and free to download. It covers all A&E types including Major A&E departments, Minor Injury Units, and Walk-in Centres across England.

### Key columns used

| Column | Description |
|---|---|
| `Period` | Month of attendance (monthly frequency) |
| `Total Attendances` | Total A&E attendances across all department types in England |

---

## Project structure

```
nhs-ae-analysis/
│
├── ae-analysis.ipynb       # Full analysis: EDA, decomposition, SARIMA modelling, forecasting
├── .gitignore
└── README.md
```

> The raw XLS file is excluded from the repository. Download it directly from NHS England — see instructions below.

---

## Methodology

### 1. Data loading and cleaning

The NHS England time series file contains metadata rows at the top before the actual data. These were skipped on load using `pd.read_excel(skiprows=13)`. Only the `Period` and `Total Attendances` columns were retained. Missing values were dropped and dates were parsed to datetime format.

### 2. Exploratory data analysis

Three key patterns were identified:

- **Upward trend**: Total monthly attendances grew from approximately 1.75 million in 2010 to over 2.2 million by 2024 — a 26% increase over 15 years
- **Consistent seasonality**: Clear repeating annual pattern with peaks in summer months (May, July) driven by minor injuries, and secondary peaks in winter driven by respiratory illness
- **COVID-19 anomaly**: A dramatic drop in March-April 2020 followed by a gradual recovery — the most significant structural break in the series

### 3. STL decomposition

STL (Seasonal and Trend decomposition using Loess) was applied to decompose the series into three components:

- **Trend**: Smooth long-run direction — shows the consistent upward trajectory interrupted by COVID
- **Seasonality**: The repeating annual pattern — remarkably stable across all 15 years
- **Residuals**: Unexplained variation — the COVID period produces residuals far outside the normal range, confirming it as a genuine anomaly

### 4. SARIMA modelling

A SARIMA(1,1,1)(1,1,1,12) model was fitted on training data (August 2010 — December 2023) and evaluated on the test period (January 2024 — March 2026).

**Model parameters explained:**
- `(1,1,1)` — AR(1): use 1 lag; I(1): difference once to remove trend; MA(1): use 1 lag of forecast error
- `(1,1,1,12)` — seasonal equivalents with a 12-month period

### 5. Forecasting with uncertainty

A 12-month ahead forecast was generated with 95% confidence intervals. The widening of the confidence bands over time reflects the compounding uncertainty of longer-horizon forecasts — an important and honest feature of any real-world forecasting tool.

---

## Key findings

**2024 attendances exceed pre-COVID levels** — monthly attendances in 2024 are consistently higher than the equivalent months in 2019, confirming a structural increase in demand rather than a return to the pre-pandemic baseline.

**Seasonal pattern is highly predictable** — the STL decomposition shows the seasonal component is extremely stable across 15 years. This makes short-term (1-3 month) forecasts relatively reliable.

**The model predicts continued growth** — the 12-month forecast shows attendances continuing to rise, consistent with demographic trends (ageing population) and ongoing post-COVID backlog pressures.

**Winter remains the highest-risk period** — while total attendances peak in summer (driven by minor injuries), the proportion of serious emergency admissions is highest in winter, creating compounding pressure on already stretched departments.

---

## How to run locally

### Prerequisites

- Python 3.10+
- Anaconda recommended

### 1. Clone the repo

```bash
git clone https://github.com/emmanuellabinjo/nhs-ae-analysis.git
cd nhs-ae-analysis
```

### 2. Install dependencies

```bash
conda install statsmodels xlrd matplotlib seaborn
pip install scikit-learn
```

### 3. Download the data

Go to [NHS England A&E Statistics](https://www.england.nhs.uk/statistics/statistical-work-areas/ae-waiting-times-and-activity/) and download the **Monthly A&E Attendances and Emergency Admissions — England Time Series** file. Save it in the project folder.

Update the filename in the notebook to match the downloaded file name.

### 4. Run the notebook

Open `ae-analysis.ipynb` and run all cells in order.

---

## Limitations and future work

### Current limitations

- **SARIMA baseline only**: A single SARIMA(1,1,1)(1,1,1,12) model was used without systematic hyperparameter tuning. Optimal order selection using AIC/BIC grid search would likely improve performance.

- **COVID period not explicitly excluded**: The COVID anomaly was identified visually but not formally removed from the training data. A formal intervention model or exclusion of 2020-2021 from training would produce a cleaner model.

- **National-level only**: The analysis uses England-wide totals. Regional breakdowns (by NHS Trust or Integrated Care Board) would be far more useful for operational planning.

- **No exogenous variables**: The model uses attendance history only. Adding external predictors — flu rates, temperature, bank holidays, population age — could substantially improve accuracy.

### Planned improvements

- [ ] Implement AIC/BIC grid search for optimal SARIMA parameters
- [ ] Formally exclude COVID period from training data
- [ ] Break down analysis by NHS region
- [ ] Add exogenous regressors (flu rates, temperature) using SARIMAX
- [ ] Compare against Prophet and Holt-Winters models
- [ ] Build an interactive Streamlit dashboard for non-technical NHS planners

---

## Acknowledgements

- [NHS England](https://www.england.nhs.uk/) for publishing A&E statistics under the Open Government Licence
- [statsmodels](https://www.statsmodels.org/) for the SARIMA and STL implementations

---

## Licence

This project is licensed under the MIT Licence.

The underlying data is published under the [Open Government Licence v3.0](https://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/).

---

*Built as part of a data science portfolio. Questions or suggestions? Open an issue or connect on [LinkedIn](https://linkedin.com/in/emmanuel-labinjo).*