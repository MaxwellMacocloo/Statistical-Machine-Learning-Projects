# Detecting Shill Bidding: Hand-Built Logistic Regression and a Kernel Classifier

**Language:** R  **Date:** 2022  **Type:** Binary classification, numerical optimization, kernel methods

## Overview

Shill bidding is when a seller, or an accomplice, bids on their own auction to push the price up. This project builds two classifiers for spotting it **from first principles** instead of calling a ready-made model function:

1. **Logistic regression fitted by direct optimization.** The negative log-likelihood is written by hand and minimized numerically. Standard errors and Wald tests come from the Hessian matrix.
2. **A "primitive LDA" kernel classifier.** Each new point goes to the class whose training points it is most similar to on average, with similarity measured by a polynomial kernel (the kernel trick).

## Data

| Item | Detail |
|---|---|
| Source | UCI Machine Learning Repository, *Shill Bidding Dataset* (eBay auctions) |
| Size | 6,321 bidder-auction records |
| Predictors (9) | Bidder tendency, bidding ratio, successive outbidding, last bidding, auction bids, starting price average, early bidding, winning ratio, auction duration |
| Response | `Class`, coded −1 = normal, +1 = shill |
| Class balance | 5,646 normal (89.3%) vs 675 shill (10.7%) |
| Missing values | None |

The three ID columns (record, auction, bidder) were removed before modeling.

## Methodology

**Partitioning.** Random 2 : 1 : 1 split into **training D1 = 3,189**, **validation D2 = 1,662** and **test D3 = 1,470**.

**Part A: logistic regression as an optimization problem**
- Wrote the negative log-likelihood for ±1-coded labels. For each record it is the sum of log(1 + exp(−y·xᵀβ)).
- Minimized it on D1 ∪ D2 with two general-purpose optimizers: **BFGS** (quasi-Newton) and **conjugate gradient (CG)**.
- Inverted the numerical Hessian to get the variance-covariance matrix, then standard errors, Wald z-statistics and *p*-values.
- Checked the hand-built estimates against R's built-in `glm()`.
- Classified D3 with a 0.5 probability cut-off.

**Part B: the kernel trick**
- Standardized all predictors using **training-set** means and standard deviations only.
- Decision rule: a point *z* gets the class whose training points have the higher average kernel similarity to *z*, with a bias term to balance the two classes.
- Started with a linear kernel (polynomial, degree 1). Then tuned the polynomial degree from 1 to 20 on validation data and applied the best degree to the test set.

## Results

### Part A: optimizer comparison

| Optimizer | Converged? | Minimized negative log-likelihood |
|---|---|---|
| **BFGS** | **Yes** | 247.01 |
| Conjugate gradient | No (hit iteration limit) | 247.97 |

The BFGS estimates match `glm()` to four decimal places: intercept −10.620, SE 0.796, and the same for every slope. So the hand-built likelihood and Hessian standard errors are correct.

**Significant predictors (α = 0.05)**

| Predictor | Estimate | z | *p* |
|---|---|---|---|
| Successive outbidding | 10.62 | 16.10 | < 0.001 |
| Winning ratio | 5.43 | 8.38 | < 0.001 |
| Last bidding | 1.49 | 1.96 | 0.0495 |

The other six predictors were not significant once these were in the model.

**Test-set performance (D3, n = 1,470): 98.57% accuracy**

| | Actual normal | Actual shill |
|---|---|---|
| Predicted normal | 1,309 | 8 |
| Predicted shill | 13 | 140 |

Sensitivity is 94.6% (140 / 148 shill bids caught). Specificity is 99.0%.

### Part B: kernel classifier

| Setting | Data evaluated | Accuracy |
|---|---|---|
| Linear kernel (degree 1), trained on D1 | D2 | 76.5% |
| Polynomial degree tuned 1 → 20 | D2 | peak **95.3% at degree 4**, declining to 90.3% at degree 20 |
| Polynomial, degree 4 | D3 | **96.5%** |

Accuracy climbs sharply from degree 1 (77.3%) to degree 4 (95.3%), then slowly falls. Higher degrees overfit.

## Key Findings

- **Behavior beats price.** Successive outbidding (repeatedly bidding just above others) and a high winning ratio separate shill bidders from normal ones. Price-related and timing variables add little once these two are known.
- **Optimizer choice matters.** BFGS converged cleanly, while CG stopped at a worse point without converging. With ill-conditioned problems like this one (the intercept is near −10.6), quasi-Newton methods are the safer default.
- **A plain kernel rule comes close to logistic regression.** A classifier with no fitted weights, just averaged kernel similarities, reached about 96% test accuracy once the kernel degree was tuned. The linear version reached only 77%, which shows how much a non-linear feature map adds.

## Notes on the Original Analysis

- **Baseline:** 89% of records are normal, so a model that always predicts "normal" already scores 89% accuracy. Sensitivity (94.6% for the logistic model) is the more informative number.
- The original write-up listed *early bidding* as significant. The output shows **winning ratio** is significant and early bidding is not (*p* = 0.11). The table above follows the output.
- **The kernel classifier's test accuracy is optimistic.** In the degree-tuning step, and again in the final test, each class's average similarity was computed from the same data being scored, using that data's true labels. A clean estimate would compute those averages from training data only. The 96.5% figure should be treated as an upper bound.

## Tools

R: `optim` (BFGS / CG), `kernlab` (polynomial kernels, kernel matrices), base R for likelihood, Hessian-based inference and validation.
