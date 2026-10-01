# Early-Stage Diabetes Risk: SEMMA with Regularized Logistic Regression

**Language:** R  **Date:** September 2022  **Type:** Binary classification, variable selection

## Overview

This project follows the SEMMA framework (Sample, Explore, Modify, Model, Assess) from start to finish. The task is to predict a positive diabetes diagnosis from questionnaire answers about symptoms. Three penalized logistic regression methods (SCAD, MCP and LASSO) compete. Two things are compared: which predictors each method keeps, and how well each resulting model ranks unseen patients.

## Data

| Item | Detail |
|---|---|
| Source | UCI Machine Learning Repository, *Early Stage Diabetes Risk Prediction* |
| Origin | Direct questionnaires, patients of Sylhet Diabetes Hospital, Bangladesh |
| Size | 520 patients, 16 predictors + 1 binary response |
| Predictors | Age (numeric); Gender; 14 yes/no symptoms (Polyuria, Polydipsia, sudden weight loss, weakness, Polyphagia, genital thrush, visual blurring, itching, irritability, delayed healing, partial paresis, muscle stiffness, alopecia, obesity) |
| Response | `class`: Positive (320, 61.5%) / Negative (200, 38.5%) |
| Missing values | None |

The classes are moderately imbalanced (about 62/38). The imbalance is not severe enough to need resampling.

## Methodology

1. **Explore.** Inspected variable types, checked for missing values and wrong records, and checked the class balance.
2. **Screen variables.** Tested each predictor's marginal association with the response, using a deliberately liberal threshold (α = 0.25) to drop only clearly useless predictors.
   - Age (continuous): Wilcoxon rank-sum test, *p* = 0.012, so kept.
   - Categorical symptoms: Pearson chi-square test of independence.
   - **Dropped:** Itching (*p* = 0.83) and Delayed healing (*p* = 0.33).
   - Strongest associations: Polyuria (χ² = 227.9), Polydipsia (χ² = 216.2), Gender (χ² = 103.0), sudden weight loss (χ² = 97.3), partial paresis (χ² = 95.4).
3. **Partition.** Random 2/3 : 1/3 split. **Training D1 = 346** and **test D2 = 174**.
4. **Model.** Fitted penalized logistic regression with three penalties, each tuned by 5-fold cross-validation on D1.
5. **Refit.** Refitted an ordinary logistic regression on the variables each penalty selected, for interpretation.
6. **Assess.** Measured ROC curves and AUC on the held-out D2, with 95% confidence intervals.

## Results

### Variable selection

| Penalty | CV-optimal λ | Predictors selected | AIC of refit model |
|---|---|---|---|
| **SCAD** | 0.0248 | **5:** Gender, Polyuria, Polydipsia, Irritability, Alopecia | 154.26 |
| MCP | 0.0022 | 13 (all remaining predictors) | 146.42 |
| LASSO | 0.0066 | 13 (same set as MCP) | 146.42 |

### Refit SCAD model (5 predictors), all terms significant

| Predictor | Coefficient | Odds ratio | *p*-value |
|---|---|---|---|
| Gender = Male | −3.41 | 0.03 | < 0.001 |
| Polyuria = Yes | +3.78 | 43.9 | < 0.001 |
| Polydipsia = Yes | +3.63 | 37.6 | < 0.001 |
| Irritability = Yes | +2.49 | 12.0 | < 0.001 |
| Alopecia = Yes | −1.21 | 0.30 | 0.011 |

In the 13-variable MCP/LASSO refit, only 7 terms were significant at 5%: Gender, Polyuria, Polydipsia, Polyphagia, Irritability, partial paresis and genital thrush.

### Test-set performance (D2, n = 174)

| Model | AUC | 95% CI |
|---|---|---|
| SCAD (5 predictors) | 0.956 | 0.924 – 0.988 |
| MCP (13 predictors) | 0.956 | 0.927 – 0.986 |
| LASSO (13 predictors) | 0.956 | 0.927 – 0.986 |

## Key Findings

- **Parsimony wins.** SCAD used 5 predictors, the others 13, and all three reached the same test AUC (0.956). For a screening questionnaire, five questions matter far more in practice than thirteen.
- **The classic symptoms dominate.** Polyuria (excessive urination) and Polydipsia (excessive thirst) are by far the strongest signals. Each multiplies the odds of a positive diagnosis by roughly 40, holding the others fixed.
- **Gender effect.** Male patients have much lower odds of a positive result *in this sample*. This most likely reflects how the hospital population was sampled, not a general biological effect, and should be read with care.
- **SCAD's sparsity is useful here.** SCAD and MCP are non-convex penalties built to shrink large coefficients less than LASSO does. Here SCAD produced the sparsest model, while MCP and LASSO converged to the same 13-variable solution.

## Notes on the Original Analysis

- The original write-up gave the SCAD refit's AIC as 140.22, but the model output shows **154.26**. The table above uses the output value. With that value, SCAD has a *higher* AIC than MCP/LASSO (146.42). It is still the preferred model, because its test AUC is equal and it uses 8 fewer predictors.
- The *p*-values come from refitting an unpenalized model after selection. Post-selection *p*-values are optimistic, so they should be read as descriptive.
- Performance rests on a single train/test split. Repeated cross-validation would give a steadier estimate, and reporting sensitivity and specificity at a chosen cut-off would make the result clinically more useful.

## Tools

R: `ncvreg` (SCAD/MCP/LASSO paths and CV), `Hmisc` (chi-square screening), `verification` and `cvAUC` (ROC/AUC with confidence intervals).
