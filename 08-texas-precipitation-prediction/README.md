# Predicting Daily Precipitation in Texas (Lufkin Weather Station)

**Language:** R  **Date:** July 2022  **Type:** Regression, time-based validation, regularization and ensembles

## Overview

Precipitation controls how much surface water and groundwater is available for drinking, irrigation and industry, and heavy precipitation is the main cause of floods. This project models **daily precipitation at the Lufkin, Texas weather station** from other same-day weather measurements. It compares five regression methods and asks whether a smaller set of predictors does as well as all twelve.

## Data

| Item | Detail |
|---|---|
| Source | U.S. Environmental Protection Agency, *Meteorological Data – Texas* |
| Station | Lufkin, TX |
| Size | 10,957 daily records (1961 – 1990) |
| Response | Precipitation (cm/day) |
| Predictors (12) | Pan evaporation, mean temperature, wind speed, solar radiation, FAO short-grass reference evapotranspiration, daylight station pressure, daylight relative humidity, daylight opaque sky cover, daylight temperature, daylight broadband aerosol, daylight prevailing wind speed, daylight prevailing wind direction |

### Exploratory findings

- Precipitation has only **weak, mostly negative** correlations with individual predictors such as pan evaporation, temperature and solar radiation. Rainy days are cooler, cloudier and less evaporative.
- Several predictors are skewed and contain outliers: temperature, pan evaporation and daylight wind speed. The response itself has many outliers, typical of rainfall data (mostly dry days with occasional heavy rain).

## Methodology

- **Time-based split:** trained on **1961–1989** and tested on **all of 1990**. This mirrors real forecasting, where the model never sees the future.
- **Predictor selection:** backward selection, validated with a hold-out set and cross-validation, reduced the 12 predictors to **8**: pan evaporation, temperature, wind speed, solar radiation, station pressure, relative humidity, daylight temperature and daylight wind speed.
- Each model was fitted on both the **full (12)** and **reduced (8)** predictor sets.

| Model | Tuning |
|---|---|
| Multiple linear regression | n/a |
| Ridge regression | λ grid from 10¹⁰ to 10⁻², 100 values, chosen by CV |
| LASSO regression | Same λ grid, chosen by CV |
| Bagging | All predictors tried at each split, 500 trees |
| Random Forest | `mtry` = 6 (full) / 2 (reduced), 500 trees |

**Metric:** test mean squared error (MSE) on the 1990 data.

## Results

| Model | Test MSE (12 predictors) | Test MSE (8 predictors) |
|---|---|---|
| Multiple linear regression | 1.318 | 1.317 |
| Ridge regression (λ = 0.037) | 1.328 | 1.329 |
| LASSO regression (λ ≈ 0) | 1.318 | 1.318 |
| Bagging | 1.005 | 0.982 |
| **Random Forest** | 1.000 | **0.977** |

### Sample predictions, best model (Random Forest, 8 predictors)

| Date | Actual (cm) | Predicted (cm) |
|---|---|---|
| 1990-01-04 | 0.05 | 0.05 |
| 1990-01-05 | 1.70 | 1.75 |
| 1990-01-03 | 0.61 | 2.20 |
| 1990-12-28 | 0.00 | 1.80 |

## Key Findings

- **Random Forest is the best model** on both predictor sets. Its lowest test MSE (0.977) is about **26% below** linear regression (1.317).
- **Tree ensembles beat linear models by a wide margin, while regularization barely helped.** Ridge and LASSO matched plain least squares (cross-validation chose λ ≈ 0 for LASSO). The weak point is not overfitting but the **non-linear** relationship between weather and rain, which trees capture and linear models cannot.
- **Fewer predictors, slightly better accuracy.** Dropping 4 of 12 predictors *lowered* the test error for both ensembles. The dropped variables (aerosol, sky cover, wind direction, reference evapotranspiration) appear to add little beyond the remaining eight.
- **The model gets amounts right on some days and misses the timing on others.** It matches some days closely (5 January: 1.70 actual vs 1.75 predicted) but predicts rain on dry days (28 December: 0 vs 1.80). Deciding *whether* it rains is the harder part.

## Limitations and Next Steps

- **Zero-inflated response.** Most days are dry. A **two-part model** (classify rain or no rain, then predict the amount on rainy days) usually suits rainfall better than a single regression.
- **No baseline reported.** Comparing against a "predict the training mean" model would show how much of the variance is actually explained.
- **Single test year.** Validating across several years (rolling-origin evaluation) would show whether 1990 was typical.
- Adding **lagged weather** (the previous day's humidity and pressure) and seasonal terms would likely improve accuracy.

## References

1. U.S. EPA. *Meteorological Data – Texas.* https://www.epa.gov/ceam/meteorological-data-texas
2. U.S. EPA. *Climate Change Indicators: U.S. and Global Precipitation.* https://www.epa.gov/climate-indicators/climate-change-indicators-us-and-global-precipitation
3. National Geographic Society. *Types of Precipitation.* https://education.nationalgeographic.org/resource/types-precipitation

## Tools

R: penalized regression (ridge, LASSO), bagging and Random Forest, backward stepwise selection, and scatter, correlation and box plots for EDA.
