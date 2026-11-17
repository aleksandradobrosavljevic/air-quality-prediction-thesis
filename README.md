# Air Quality Forecasting in Serbia — PM10 & PM2.5 Prediction

Bachelor's thesis project: forecasting daily PM10 and PM2.5 concentrations across Serbian air quality monitoring stations (2013–2024) using classical time series models, gradient boosting, and deep learning.

## Overview

Air pollution (particulate matter PM10 and PM2.5) poses a serious risk to public health, and short-term forecasting can support early-warning systems for vulnerable populations. This project builds and compares forecasting models for 1-, 3-, and 7-day-ahead predictions of PM10 and PM2.5 concentrations, combining official air quality measurements with meteorological data.

## Data

- **Air quality data**: Daily PM10 and PM2.5 measurements (2013–2024) from Serbian monitoring stations (SEPA, the Serbian Environmental Protection Agency, via data.gov.rs), originally distributed across yearly Excel files (`2013.xlsx` ... `2024.xlsx`) with inconsistent formats (merged suffixes, multiple table blocks per sheet, mixed date formats). Only stations with at least 5 years of coverage for both pollutants were kept (13 stations).
- **Meteorological data**: Daily temperature, precipitation, wind speed/direction, humidity and pressure (Open-Meteo, one `.csv` file per station), plus boundary layer height from the ERA5 reanalysis dataset (downloaded from the Open-Meteo API on first run), merged by station and date.

## Data cleaning and corrections

- Identified and corrected source errors in the PM2.5 files:
  - **Duplicated series**: PM2.5 for Obrenovac Centar (2019) and Popovac (2020) are copies of the Kosjeric series and were removed (659 rows).
  - **One-day shift**: PM2.5 in 2020 (from 1 March) appears to be shifted by one day relative to PM10. No explicit error was found in the dates of the source files, so the shift was inferred statistically (correlation with PM10 and with weather) and corrected; the 29 February 2020 PM2.5 value was dropped.
- Annual means of the corrected data agree with the SEPA annual reports for 2019 and 2020.
- Removed physically impossible readings (PM2.5 > PM10, PM2.5 = 0), isolated spikes, and flagged sensor dropouts.
- Filled short gaps (≤7 days) using forward-fill, while excluding longer gaps from training/evaluation to avoid introducing synthetic data into the targets.

## Feature Engineering

- Lag features (1, 3, 7 days) and rolling averages (3, 7 days) for PM10/PM2.5 and weather variables
- Cyclical encoding of month (sin/cos) and a heating-season flag (Oct–Mar), since pollution is strongly seasonal in Serbia
- Strict chronological train / validation / test split (train through 2021, validation 2022, test 2023–2024) with target-leakage checks across split boundaries

## Models

| Model | Notes |
|---|---|
| Naive & seasonal-naive baselines | Reference points for evaluating real skill |
| ARIMA / ARIMAX (SARIMAX) | Order selected via AIC search; ARIMAX includes weather variables |
| XGBoost | Trained across all stations jointly; hyperparameters tuned with Optuna |
| LSTM | Sequence-based deep learning model (7-day input window), with and without weather/calendar channels, trained with early stopping |

Feature importance was additionally analyzed with a Random Forest regressor, and an ablation study tested whether adding boundary layer height improves predictions.

## Results (test set, 2023–2024, all stations)

Results for 1-day-ahead forecasts:

| Pollutant | Model | MAE | RMSE | R² |
|---|---|---|---|---|
| PM10 | XGBoost (gamma objective) | 10.04 | 15.90 | 0.662 |
| PM2.5 | XGBoost (gamma objective) | 6.66 | 10.81 | 0.766 |

ARIMA and ARIMAX are fitted on a single station (Beograd Mostar). On the same 697 days there, all models perform similarly at the 1-day horizon (R² around 0.66–0.69). ARIMAX uses actual future weather values, so it is an optimistic upper bound and not directly comparable to the others. At longer horizons (3 and 7 days), XGBoost and LSTM, which learn across all stations, generalize better than the single-station models, and the naive baseline drops to R² ≈ 0 at 3 days.

The project also evaluates the models as a binary classifier for daily PM10 threshold exceedance (>50 µg/m³), reaching a precision of 0.72 and recall of 0.73 (F1 = 0.73).

## How to run

1. Install the dependencies: `pip install -r requirements.txt`
2. Keep all `.xlsx` and `.csv` input files in the same folder as the notebook.
3. Open the notebook in Jupyter and run all cells. Boundary layer height is downloaded from the Open-Meteo API on first run (internet connection needed). Execution may take a while (Optuna, LSTM).

## Tech Stack

Python, pandas, NumPy, scikit-learn, XGBoost, TensorFlow/Keras, statsmodels, Optuna, Matplotlib

## Note

The notebook is written in Serbian (code comments and markdown explanations), as it was developed for a university thesis. This README provides an English summary of the data, methodology and key results.
