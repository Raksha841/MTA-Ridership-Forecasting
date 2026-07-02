# MTA Subway Ridership Analysis & Forecasting

**Dataset:** MTA Subway Hourly Ridership (Jan 1–7, 2024)

## Business Objective

The MTA operates one of the largest public transit systems in North America, serving millions of riders daily across every NYC borough. This project analyzes hourly subway ridership data to answer: **where are riders, who are they, and how can the MTA better serve them?** The goal is to surface usage patterns by time, borough, and station, and to build forecasting models that support operational decisions like staffing, train frequency, and maintenance scheduling.

## Dataset

- **452,752** raw records → **451,230** after cleaning
- **157** hourly timestamps across **424** unique stations in **4** boroughs (Manhattan, Brooklyn, Queens, Bronx)
- Average of **33.49** riders per entry (std dev 79.35), ranging from 1 to 999
- Payment methods: MetroCard and OMNY

## What's in this repo

- **EDA:** Cleaning and exploring hourly ridership records — patterns by hour, borough, station, payment method, and fare class.
- **Time-series forecasting:** Five models built and compared on hourly ridership:
  - Holt-Winters Exponential Smoothing
  - Prophet
  - SARIMAX (grid search over seasonal/non-seasonal orders)
  - Random Forest (24 lag features + Holt-Winters as exogenous input)
  - XGBoost (same feature set as Random Forest)
- **Model comparison:** Validation-set MSE and R² across all five models.
- **Interactive interface:** A Gradio-based dashboard/chatbot (OpenAI-powered) for querying ridership data, model results, and forecasts conversationally.

## Model Results (Validation Set)

| Model | MSE | R² | Rank |
|---|---|---|---|
| Random Forest (Lags) | 270,947,740 | 0.84 | 1 (best) |
| XGBoost (Lags) | 312,092,896 | 0.82 | 2 |
| Prophet | 1,192,327,135 | — | 3 |
| Holt-Winters | 4,575,469,008 | — | 4 |
| SARIMAX (0,1,0)x(1,1,1,24) | 4,972,497,889 | — | 5 (worst) |

Random Forest was the strongest performer, explaining 84% of ridership variance — roughly a 94% lower MSE than SARIMAX. Machine learning models (with lag features) outperformed traditional time-series methods overall.

## Key Findings

- **Manhattan** accounts for **48.4%** of total system ridership, followed by Brooklyn (26.4%), Queens (17.1%), and the Bronx (8.2%).
- Ridership peaks at **8:00 AM** and again in the **4–5 PM** window; the quietest hour is **3:00 AM**.
- **OMNY** adoption is strong (~41%+ of ridership) and growing relative to legacy MetroCard.
- SARIMAX underperformed primarily due to limited data (only 157 hourly points — too few for reliable seasonal ARIMA at a 24-period seasonality).

## Repo Structure

```
mta-ridership-forecasting/
├── MTA_Ridership_Forecasting.ipynb   # Full analysis: EDA, models, chatbot dashboard
├── requirements.txt
└── README.md
```

## Data

This project uses the **MTA Subway Hourly Ridership (2020–2024)** dataset, publicly available via [NYC Open Data](https://data.ny.gov/). The raw CSV is not included in this repo due to size — download it from the source and update the file path in the notebook to point to your own copy.

## Setup

To run this project on your own computer:
1. Download the `MTA_Ridership_Forecasting.ipynb` file from this repo and open it in Jupyter or Google Colab.
2. Install the packages listed in `requirements.txt`.
3. Update the data path in the notebook to point to your local copy of the dataset.
4. The chatbot cells require an OpenAI API key — you'll be prompted to enter it securely at runtime. **Never hardcode or commit your API key.**

## Business Recommendations

- **Resource allocation:** Use Random Forest forecasts to optimize staffing, train frequency, and maintenance at high-demand stations (Times Sq-42 St, Flushing-Main St, Grand Central-42 St) during peak hours.
- **Targeted infrastructure/marketing:** Focus investment in Manhattan given its ridership share, while exploring service improvements to grow ridership in the Bronx and Queens.
- **Promote OMNY adoption:** Continue transitioning riders from MetroCard to OMNY to streamline operations and improve data quality.
- **Off-peak incentives:** Use early-morning low-demand windows to offer promotions that shift demand away from rush hours.
- **Proactive planning:** Apply Random Forest/XGBoost forecasts to budget allocation, capital projects, and emergency preparedness.

## Limitations

- **Short data window:** The dataset spans only Jan 1–7, 2024, limiting the ability to capture weekly/yearly seasonality (weekly decomposition at period=168 wasn't feasible with only 157 points).
- **No exogenous variables:** Models don't yet incorporate weather, holidays, special events, or service disruptions.
- **Interpretability tradeoff:** Random Forest and XGBoost outperform but are less interpretable than statistical models like SARIMAX.
- **Stationarity assumptions:** SARIMAX assumes stationarity, which may not hold perfectly for real-world ridership data.
- **Generalizability:** Results are specific to this dataset and time window; re-evaluation would be needed for other periods or transit systems.
