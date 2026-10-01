# Predicting Employee Attrition: Linear vs Non-Linear Classifiers

**Language:** R  **Date:** 2022  **Type:** Binary classification, HR analytics, model comparison

## Overview

Why do employees leave, and can we tell in advance who will? This project explores an HR dataset of about 15,000 employees and compares five supervised learning methods for predicting attrition. They range from a fully linear model to flexible non-linear ones:

**Logistic regression (LASSO-guided)** → **Generalized Additive Model (GAM)** → **Multivariate Adaptive Regression Splines (MARS)** → **Projection Pursuit Regression (PPR)** → **Random Forest**

## Data

| Item | Detail |
|---|---|
| Source | HR Analytics employee-retention dataset (`HR_comma_sep.csv`) |
| Size | 14,999 employees, 9 predictors + 1 response |
| Predictors | Satisfaction level, last evaluation score, number of projects, average monthly hours, years at company, work accident (0/1), promotion in last 5 years (0/1), department (10 levels), salary (low < medium < high, ordinal) |
| Response | `left`: 1 = left the company (23.8%), 0 = stayed (76.2%) |
| Missing values | None |

## Exploratory Findings

- **Satisfaction separates the groups.** Leavers cluster at low-to-mid satisfaction (about 0.25–0.5), while stayers become more common as satisfaction rises. Average satisfaction across all staff is 0.61.
- **Salary drives attrition.** Within each salary band:

  | Salary | Attrition rate |
  |---|---|
  | Low | **29.7%** |
  | Medium | 20.4% |
  | High | **6.6%** |

- **Tenure varies by department.** Management staff stay longest.
- **Workload interacts with satisfaction.** Employees on only 2 projects mostly report satisfaction below 0.5, while those on 4–5 projects are mostly above it.
- Goodman–Kruskal τ was used to measure association across the mixed numeric and categorical variables.

## Methodology

- **Split:** random 67% / 33%, giving **training = 10,049** and **test = 4,950**.
- **Metric:** area under the ROC curve (AUC) on the test set.

| Model | Configuration |
|---|---|
| Logistic regression | LASSO path with 10-fold CV. The one-standard-error λ (0.0089) kept 12 non-zero terms. A standard logistic model was then fitted for interpretable odds ratios. |
| Random Forest | `mtry` tuned by out-of-bag error → 4. 200 trees. |
| GAM | Smoothing spline on satisfaction and loess on last evaluation; other terms linear. Stepwise GAM search over smooth/linear/dropped forms. |
| MARS | Degree-2 interactions, binomial GLM link |
| PPR | Supersmoother ridge functions. 4 terms selected (from up to 10), bass = 5. |

## Results

### Model comparison (test set, n = 4,950)

| Rank | Model | Test AUC | 95% CI |
|---|---|---|---|
| 1 | **Random Forest** | **0.992** | n/a |
| 2 | MARS | 0.979 | 0.973 – 0.984 |
| 3 | PPR | 0.972 | 0.966 – 0.978 |
| 4 | GAM | 0.913 | 0.904 – 0.922 |
| 5 | Logistic regression | 0.822 | 0.809 – 0.834 |

The Random Forest's out-of-bag error rate was **1.03%**: 0.26% for stayers and 3.5% for leavers.

### What drives attrition: logistic-regression odds ratios

| Factor | Odds ratio | Reading |
|---|---|---|
| Satisfaction level | 0.016 per unit (**0.66 per +0.1**) | Each 0.1 rise in satisfaction cuts the odds of leaving by about a third |
| Years at company | 1.32 per year | Odds rise 32% with each extra year |
| Last evaluation | 1.86 per unit | Higher-rated employees are *more* likely to leave |
| Number of projects | 0.73 per project | |
| Work accident | 0.22 | |
| Promotion in last 5 years | 0.20 | A promotion cuts the odds of leaving by about 80% |
| Salary (linear trend) | 0.27 | Strong protective effect of higher pay |
| Department: R&D / Management | 0.54 / 0.59 | Lower attrition than the reference department (accounting) |

### Variable importance (consistent across RF and MARS)

1. **Satisfaction level**
2. **Number of projects**
3. Years at company · average monthly hours · last evaluation

Department, salary, work accident and promotion add comparatively little once these five are known. MARS used only these five predictors in its final model.

## Key Findings

- **Non-linearity is the main story.** AUC rose from 0.82 (linear logistic) to 0.91 just by letting *two* variables bend in the GAM, and to 0.98–0.99 with fully flexible models.
- **The relationships are U-shaped.** Partial-dependence plots and MARS hinge points (at 3 projects and satisfaction 0.38) show that both under-loaded *and* over-loaded employees leave. A linear model cannot represent this.
- **High performers leave too.** The positive effect of last evaluation suggests the company loses strong employees, not just weak ones. This is a retention risk worth flagging to HR.
- **Practical levers:** satisfaction, sensible workload (about 3–5 projects), promotion paths and pay. Each has a large, consistent effect.

## Notes on the Original Analysis

- The "LASSO" row in the comparison is the AUC of a standard logistic regression on all predictors. LASSO was used to choose and justify the variables, not to produce the final predictions.
- The models were compared on a single split. With 15,000 rows the estimates are fairly stable, but repeated cross-validation would also give confidence intervals for the Random Forest.

## Tools

R: `glmnet` (LASSO), `randomForest`, `gam` (GAM and stepwise GAM), `earth` (MARS), `ppr`, `vip` and `pdp` (importance and partial dependence), `GoodmanKruskal`, `questionr` (odds ratios), `cvAUC` and `verification` (ROC/AUC), `ggplot2`.
