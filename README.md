# U.S. Recession Forecasting with Macroeconomic Indicators

A Python econometrics/data-science project that tests whether publicly available macroeconomic indicators can help identify U.S. recession risk over a 12-month horizon.

## Project question

Can a small, economically motivated set of macroeconomic indicators improve recession-risk classification relative to a yield-curve-only baseline?

## Data

Monthly/aggregated data are retrieved from the Federal Reserve Economic Data (FRED) API.

| FRED series | Variable | Transformation |
|---|---|---|
| `USREC` | NBER recession indicator | Used to construct the 12-month-ahead target |
| `T10Y2Y` | 10Y–2Y Treasury spread | Daily observations averaged monthly |
| `UNRATE` | Unemployment rate | 3-month change |
| `HOUST` | Housing starts | 12-month percent change |
| `UMCSENT` | Consumer sentiment | 12-month change |
| `BAA10Y` | Baa–10Y Treasury credit spread | Daily observations averaged monthly |

## Models

Five nested logistic-regression specifications are compared:

1. **Model 1:** Yield spread
2. **Model 2:** + unemployment change
3. **Model 3:** + housing-start growth
4. **Model 4:** + consumer-sentiment change
5. **Model 5:** + corporate credit spread

Predictors are standardized inside a scikit-learn pipeline so scaling is learned only from training data.

## Validation design

A random train/test split is inappropriate for this forecasting problem because it can mix future observations into training.

The project therefore uses **expanding-window walk-forward validation**. Because the dependent variable looks 12 months ahead, the 12 months immediately before each test window are **purged from training** to prevent target leakage.

Five test windows are evaluated:

- 2000–2004
- 2005–2009
- 2010–2014
- 2015–2019
- 2020–2024

ROC-AUC is undefined for a fold containing only one target class; those fold-level values are left missing rather than forced to a score.

## Main result

Across 300 pooled out-of-sample monthly predictions (61 positive target months), the models produced:

| Model | Pooled walk-forward ROC-AUC |
|---|---:|
| Model 1 | 0.550 |
| Model 2 | 0.823 |
| Model 3 | 0.821 |
| Model 4 | 0.836 |
| Model 5 | **0.853** |

The yield-curve-only specification had limited pooled discrimination. Adding labor-market information produced the largest improvement. Housing did not improve pooled AUC on its own, while sentiment and credit-market information subsequently improved the expanded specifications.

Performance also varied substantially by historical period, with notably weak results during 2020–2024. This highlights an important limitation of macroeconomic forecasting: relationships estimated from conventional business cycles may be unstable during unusual economic regimes.

**ROC-AUC is a ranking/discrimination metric, not classification accuracy.** A pooled ROC-AUC of 0.853 should not be described as “85.3% accurate.”

## Repository structure

```text
recession-forecasting/
├── recession_model.ipynb
├── README.md
├── requirements.txt
├── .gitignore
├── .env.example
├── results/
│   └── pooled_walk_forward_auc.csv
└── LICENSE
```

## Setup

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd recession-forecasting
```

### 2. Create a virtual environment (recommended)

Windows:

```bash
python -m venv .venv
.venv\Scripts\activate
```

macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
python -m pip install -r requirements.txt
```

### 4. Configure FRED

Copy `.env.example` to `.env` and add your own FRED API key:

```text
FRED_API_KEY=your_key_here
```

`.env` is ignored by Git and should never be committed.

### 5. Run the notebook

Open `recession_model.ipynb` in VS Code or Jupyter and choose **Restart and Run All Cells**.

## Methodological limitations

- U.S. recessions are rare, so the number of independent downturn episodes is small.
- The 12-month-ahead target creates overlapping monthly labels.
- FRED historical data can be revised; this project does not use a real-time vintage database.
- ROC-AUC measures ranking, not probability calibration.
- Logistic-regression coefficients represent conditional associations, not causal effects.
- Structural breaks can weaken relationships learned from historical business cycles.
- Repeated model development can still introduce researcher degrees of freedom even with time-aware validation.
- A stricter real-time implementation would use ALFRED vintages and release-date-aware feature lags to ensure every historical forecast uses only the data actually available at that date.

## Skills demonstrated

Python · pandas · NumPy · matplotlib · scikit-learn · FRED API · logistic regression · feature engineering · time-series validation · leakage control · ROC-AUC · precision/recall · economic interpretation

## Resume description

**U.S. Recession Forecasting | Python, Econometrics, FRED API**

- Built nested logistic-regression models using Treasury spreads, unemployment, housing starts, consumer sentiment, and corporate credit spreads to forecast U.S. recession risk over a 12-month horizon.
- Designed leakage-aware expanding-window validation with a 12-month purge and evaluated 300 out-of-sample monthly forecasts per model.
- Improved pooled out-of-sample ROC-AUC from 0.550 for a yield-curve baseline to 0.853 for the five-variable specification while documenting substantial performance instability across economic regimes.

## License

MIT License. See `LICENSE`.
