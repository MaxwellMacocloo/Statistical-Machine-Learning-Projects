# Modeling Jaw-Bone Growth: Parametric vs Non-Parametric Non-Linear Regression

**Language:** R  **Date:** 2022  **Type:** Non-linear regression, smoothing, model comparison

## Overview

Jaw-bone length grows quickly early in life and then levels off. This curved, saturating relationship is the kind a straight line cannot capture. This project fits it six ways and compares their prediction error on held-out data:

- one **parametric** non-linear model (an asymptotic exponential growth curve), and
- five **non-parametric** smoothers: k-nearest neighbors, kernel regression, local polynomial regression, natural cubic regression splines and smoothing splines.

## Data

| Item | Detail |
|---|---|
| Source | `jaws` dataset |
| Size | 54 observations |
| Variables | `age` (0–50.6) and `bone` (jaw-bone length) |
| Split | Random 2/3 : 1/3, **training D1 = 36**, **test D2 = 18** |

The training set spans the full age range (0–50.6), while the test set spans a narrower range. So test predictions never **extrapolate** beyond the data the models learned from.

## Methodology

### Parametric: asymptotic exponential model

- **Full model:** bone = β₁ − β₂ · e^(−β₃·age), fitted by non-linear least squares.
- **Hypothesis test:** H₀: β₁ = β₂. This means the curve starts at zero length at age 0, giving the **reduced model** bone = β₁ · (1 − e^(−β₃·age)).
- The two models were compared with an F-test, AIC and BIC.

### Non-parametric

| Method | Tuning |
|---|---|
| k-nearest-neighbor regression | k chosen by 5-fold CV over k = 1–10 → **k = 5** |
| Kernel regression | Global plug-in bandwidth → **h = 7.82** |
| Local cubic polynomial | Epanechnikov kernel, bandwidth 8 |
| Natural cubic regression spline | 5 degrees of freedom |
| Smoothing spline | Smoothness chosen by generalized cross-validation (GCV) → **df = 5.66** (spar = 0.707) |

All six methods were compared on **prediction mean squared error (PMSE)** on the test set.

## Results

### Parametric models

| Model | β₁ | β₂ | β₃ | Residual SE | AIC | BIC |
|---|---|---|---|---|---|---|
| Full (3 parameters) | 114.77 | 116.59 | 0.126 | 13.00 | 291.69 | 298.02 |
| **Reduced (2 parameters)** | **114.93** | (= β₁) | **0.124** | **12.81** | **289.73** | **294.48** |

- F-test of H₀: β₁ = β₂ gives **p = 0.85**. There is no evidence against the simpler model, and AIC and BIC both prefer it.
- **Interpretation:** jaw-bone length approaches a ceiling of about **115 units**. It reaches half of that by age ln 2 / 0.124 ≈ **5.6**, and about 95% of it by age ≈ 24.
- The natural cubic spline fits the training data well too (R² = 0.88).

### Test-set comparison (as reported)

| Method | Test PMSE |
|---|---|
| k-nearest neighbors (k = 5) | 35.7 † |
| Local cubic polynomial | 170.8 ‡ |
| **Reduced exponential model** | **187.7** |
| Natural cubic regression spline | 270.6 |
| Smoothing spline (GCV) | 1,081.4 † |
| Kernel regression | 1,344.1 † |

† and ‡: see the notes below. These figures are not comparable with the others.

## Key Findings

- **The 2-parameter growth curve gives the best value.** Among the cleanly evaluated methods, it has the second-lowest error. It also offers what no smoother can: interpretable parameters (an adult size ceiling and a growth rate).
- **More flexibility did not help here.** With only 36 training points, flexible smoothers chase noise. The 5-df regression spline predicted worse than the 2-parameter curve.
- **The theoretical constraint holds.** Forcing the curve through the origin (zero length at birth) cost no accuracy and removed a parameter.

## Notes on the Original Analysis

The original report concluded that KNN was best. A closer look at how each number was produced changes that conclusion:

- **† KNN (35.7):** the neighbor search used *both* `age` and `bone` as inputs, so each test point was matched partly on the very value being predicted (target leakage). The cross-validation used to pick k also scored on the test set and predicted `age` instead of `bone`. The true KNN error is very likely much higher.
- **† Smoothing spline (1,081.4):** the prediction step did not receive the test ages, so it returned fitted values at the *training* ages, which were then compared with test outcomes. The number does not measure test error.
- **† Kernel regression (1,344.1):** predictions fell in a narrow band (about 90–110) while true values ranged from about 10 to 140. This suggests the predictions were not evaluated at the test ages. It should be rechecked.
- **‡ Local polynomial (170.8):** fitted on the full dataset, *including* the test points, so its error is somewhat optimistic.

**Next step:** rerun every method with a single, shared train/test pipeline, ideally with repeated cross-validation given only 54 observations.

## Tools

R: `nls` (non-linear least squares), `FNN` (KNN regression), `lokern` (plug-in kernel regression), `locpol` (local polynomial), `splines` (natural cubic splines), `smooth.spline`, `ggplot2`.
