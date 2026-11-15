# Air Quality Forecasting in Serbia — PM10 & PM2.5 Prediction

Bachelor's thesis project: forecasting daily PM10 and PM2.5 concentrations across Serbian air quality monitoring stations (2013–2024) using classical time series models, gradient boosting, and deep learning.

## Overview

Air pollution (particulate matter PM10 and PM2.5) poses a serious risk to public health, and short-term forecasting can support early-warning systems for vulnerable populations. This project builds and compares forecasting models for 1-, 3-, and 7-day-ahead predictions of PM10 and PM2.5 concentrations, combining official air quality measurements with meteorological data.

## Data

- **Air quality data**: Daily PM10 and PM2.5 measurements (2013–2024) from Serbian monitoring stations, originally distributed across yearly Excel files with inconsistent formats (merged suffixes, multiple table blocks per sheet, mixed date formats). Only stations with at least 5 years of coverage for both pollutants were kept.
- **Meteorological data**: Daily temperature, precipitation, wind speed/direction, humidity and pressure (Open-Meteo), plus boundary layer height from the ERA5 reanalysis dataset, merged by station and date.
- **Data cleaning**: Identified and corrected source errors (duplicated series between stations, a one-day timestamp shift in part of the 2020 data), removed physically impossible readings (e.g. PM2.5 > PM10), flagged sensor dropouts, and filled short gaps (≤7 days) using forward-fill while excluding longer gaps from training/evaluation to avoid introducing synthetic data into the targets.

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

## Results (test set, 2023–2024)

Best-performing models by forecast horizon (1 day ahead):

| Pollutant | Model | MAE | RMSE | R² |
|---|---|---|---|---|
| PM10 | ARIMAX + weather | 8.26 | 11.81 | 0.689 |
| PM2.5 | XGBoost (gamma objective) | 6.66 | 10.81 | 0.766 |

XGBoost and LSTM generalize better than ARIMA/ARIMAX at longer horizons (3 and 7 days ahead), since they learn across all stations simultaneously rather than per station. The project also evaluates the models as a binary classifier for daily PM10 threshold exceedance (>50 µg/m³), reaching a precision of 0.72 and recall of 0.73 (F1 = 0.73).

## Tech Stack

Python, pandas, NumPy, scikit-learn, XGBoost, TensorFlow/Keras, statsmodels, Optuna, Matplotlib

## Note

The notebook is written in Serbian (code comments and markdown explanations), as it was developed for a university thesis. This README provides an English summary of the data, methodology and key results.
