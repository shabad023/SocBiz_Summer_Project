# Agentic AI for Dynamic Tariff Optimization — EV Charging Networks

Open Project 2026 · Society of Business

A three-agent system that replaces flat-rate EV charging pricing with demand-aware dynamic tariffs. It forecasts station utilization, sets surge/discount prices based on the forecast, and tracks the outcome of each pricing decision over time.

## Problem

EV charging stations on flat ₹/kWh pricing face peak-hour congestion and off-peak idle capacity at the same time. A static price can't fix both. This project uses real charging-session data to build a pricing engine that adjusts to demand — surging when stations are crowded and discounting when they're empty.

## Approach

The system is built as three agents working in sequence:

1. **Demand Prediction Agent** — a Random Forest that predicts a station's utilization 2 hours ahead, using recent-utilization lags, rolling averages, hour of day, and station attributes.
2. **Tariff Pricing Agent** — translates the forecast into a price: surge above 80% utilization, discount below 30%, normal in between. Demand response to price is modelled with a demand elasticity of -0.3.
3. **Monitoring & Learning Agent** — evaluates each day's pricing against revenue, wait-time, and pricing-efficiency outcomes.

## Key results

Measured on a held-out test period, against the ₹15/kWh fixed-rate baseline:

| Metric | Result |
|---|---|
| Demand model R² (2-hour horizon) | ~0.86 |
| Demand model RMSE | ~0.067 |
| Improvement over naive baseline | ~27% |
| Revenue gain vs flat rate | ~+2.8% |
| Off-peak demand uplift | positive |
| Pricing efficiency | ₹15.00 → ₹15.62 per kWh |

A notable finding: the demand model beats a "predict current value" baseline by 27% at the 2-hour horizon. At a 30-minute horizon both score the same, because utilization barely changes in 30 minutes — so the 2-hour horizon is where the model adds real value and where operators get useful lead time.

## Datasets

- **ST-EVCDP (Shenzhen)** — main dataset. 247 stations, 30 days, 5-minute intervals (2.1M rows after reshaping). Used for all modelling.
- **ACN-Data (Caltech)** — supporting dataset of individual charging sessions, used to study idle-time behaviour.

## Repository structure

```
.
├── Socbiz_Summer_Project.ipynb   # main analysis notebook (run top to bottom)
├── data/                          # place the CSV files here
│   ├── occupancy.csv
│   ├── volume.csv
│   ├── price.csv
│   ├── duration.csv
│   ├── time.csv
│   ├── information.csv
│   └── stations.csv
├── outputs/                       # generated CSVs (predictions, results)
└── README.md
```

## How to run

1. Install dependencies:
   ```
   pip install pandas numpy matplotlib scikit-learn openpyxl
   ```
2. Place the dataset CSV files in the `data/` folder.
3. Update the `BASE` path at the top of the notebook to point to your `data/` folder.
4. Open the notebook and run all cells top to bottom (`Kernel → Restart & Run All`).

## Method notes

- **Chronological train/test split** — the model trains on the first 80% of the time period and tests on the last 20%, so it is always predicting the future from the past (never random split, which would leak future information into training).
- **Utilization rate** = occupancy ÷ total chargers, the core metric driving every agent.
- **Feature engineering** — lag features (5 min / 30 min / 1 hr ago), rolling averages (1 hr, 3 hr), and congestion flags.

## Limitations

- The demand elasticity (-0.3) is an assumed, literature-standard value, not estimated from this dataset.
- The wait-time metric is a demand-pressure proxy, not a full queueing model.
- Shenzhen demand patterns may not transfer directly to Indian cities.
- The comparison between dynamic-pricing and flat-rate stations is correlational; no causal claims are made.

## Tech stack

Python · pandas · scikit-learn · matplotlib
