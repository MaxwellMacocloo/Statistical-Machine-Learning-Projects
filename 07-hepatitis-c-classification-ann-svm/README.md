# Hepatitis C Detection from Blood Tests: Neural Networks, SVMs and Ensembles

**Language:** R  **Date:** 2022  **Type:** Medical binary classification, cross-validated model comparison

## Overview

Can routine blood measurements tell healthy blood donors apart from patients with Hepatitis C and its later stages (fibrosis, cirrhosis)? This project cleans and imputes a laboratory dataset, explores which markers matter, and compares **nine classifiers** with 10-fold cross-validation. The classifiers include logistic regression, Random Forest, MARS, three artificial neural network designs and three support vector machines.

## Data

| Item | Detail |
|---|---|
| Source | UCI Machine Learning Repository, *HCV data* (`hcvdat0.csv`) |
| Size | 615 subjects, 12 predictors |
| Predictors | Age, Sex, and 10 blood markers: ALB (albumin), ALP (alkaline phosphatase), ALT, AST (liver enzymes), BIL (bilirubin), CHE (cholinesterase), CHOL (cholesterol), CREA (creatinine), GGT (gamma-glutamyl transferase), PROT (total protein) |
| Response | Recoded to binary: **0** = blood donor or suspect donor (540, 87.8%), **1** = Hepatitis C, fibrosis or cirrhosis (75, 12.2%) |

### Data preparation

- **Missing values** in predictors: ALP 18 (2.9%), CHOL 10 (1.6%), and ALB, ALT, PROT 1 each. None in the response.
- **Imputation:** multiple imputation by chained equations (MICE), run **without** the response variable to avoid leaking label information into the predictors.
- Sex was converted to a dummy variable. For the neural networks and SVMs, predictors were standardized **inside each fold** using training-fold statistics only.

## Exploratory Findings

| Marker | Correlation with disease |
|---|---|
| **AST** | **+0.62** |
| GGT | +0.44 |
| BIL | +0.40 |
| CHOL | −0.27 |
| CHE | −0.23 |
| ALB | −0.18 |

- **AST is the single strongest marker.** AST and GGT are also correlated with each other (r = 0.49), consistent with both reflecting liver damage.
- **Sex is not significantly associated with disease** (chi-square *p* = 0.10). The raw rates were 14.1% for males and 9.2% for females.
- AST, GGT and BIL are strongly right-skewed: most people have normal values, with a long tail of very high readings in patients.
- An **Isolation Forest** trained on healthy donors was used to score the patient group by how unusual each patient looks compared with healthy profiles. One patient (observation 71) stood out as the most extreme.

## Methodology

- **Validation:** 10-fold cross-validation, **stratified** by class so every fold has its share of the 75 positive cases.
- **Metrics:** mean AUC and mean misclassification rate (cut-off 0.5) across folds.

| Family | Model | Configuration |
|---|---|---|
| Linear | Logistic regression | LASSO (10-fold CV, one-standard-error λ) for selection, then logistic regression |
| Tree ensemble | Random Forest | 500 trees. `mtry` tuned by out-of-bag error in every fold. |
| Splines | MARS | Up to 3-way interactions, binomial GLM |
| Neural nets | ANN (3) | 1 hidden layer, 3 neurons |
| | ANN (3, 4) | 2 hidden layers, 3 and 4 neurons |
| | ANN (7, 4) | 2 hidden layers, 7 and 4 neurons |
| | | All: logistic activation, cross-entropy loss, resilient backpropagation |
| Kernel | SVM, linear | Inner repeated 10×3 CV |
| | SVM, linear with cost grid | C ∈ {0.01 … 5} |
| | SVM, radial (RBF) | Inner repeated 10×3 CV, `tuneLength` = 10 |

## Results

### 10-fold cross-validated comparison

| Classifier | Mean AUC | Mean misclassification |
|---|---|---|
| **Random Forest** | **0.979** | **0.029** |
| Logistic regression (LASSO) | 0.960 | 0.045 |
| ANN (7, 4) | 0.956 | 0.044 |
| ANN (3) | 0.949 | 0.049 |
| MARS | 0.943 | see note |
| ANN (3, 4) | 0.919 | 0.061 |
| SVM, radial | 0.878 * | 0.043 |
| SVM, linear | 0.861 * | 0.043 |
| SVM, linear with cost grid | 0.857 * | 0.046 |

\* SVM AUCs were computed from hard 0/1 predictions, not scores, which understates AUC. See the notes below.

### Interpretable model

A logistic regression on the four predictors LASSO kept, fitted on one training fold:

| Marker | Odds ratio (per unit) | 95% CI | *p* |
|---|---|---|---|
| AST | 1.061 | 1.041 – 1.084 | < 0.001 |
| BIL | 1.067 | 1.026 – 1.112 | 0.001 |
| GGT | 1.012 | 1.003 – 1.022 | 0.009 |
| CHOL | 0.558 | 0.389 – 0.790 | 0.001 |

Each additional unit of AST raises the odds of disease by about 6%. Higher cholesterol is *protective*, which is consistent with the liver's reduced ability to make cholesterol in advanced disease.

### Variable importance (Random Forest and MARS agree)

**AST** ≫ ALT ≈ ALP > GGT > BIL. Sex and albumin carry almost no information.

## Key Findings

- **Random Forest is the most accurate and most stable classifier.** It reached a mean AUC of 0.979 with only about 3% of cases misclassified.
- **A 4-marker logistic model comes close.** At AUC 0.96 it is almost as good, and much more transparent for clinicians: AST, BIL, GGT and CHOL.
- **Bigger networks were not reliably better.** ANN (7, 4) beat the single-layer net, but ANN (3, 4) was the weakest network. With 615 rows and 75 positives, architecture choices are noisy.
- **AST is the key liver-damage marker** in every analysis: correlation, odds ratios, Random Forest and MARS.

## Notes on the Original Analysis

- **The comparison table was corrected.** In the original table, the misclassification values for the ANN and SVM rows were listed in a different order from the row labels, so each ANN row showed an SVM figure and vice versa. The table above pairs each value with its own model.
- **The MARS misclassification rate (0.177 in the original) is invalid.** Inside the MARS loop it was computed from the Random Forest's predictions by mistake. On the one fold where MARS's own predictions were scored, its misclassification was 0.055.
- **SVM AUCs are understated.** They were computed from predicted class labels (0/1) instead of predicted probabilities. Refitting the SVMs with probability output would give a fair AUC.
- **About class imbalance:** 87.8% of subjects are healthy, so a model that always predicts "healthy" scores 12.2% misclassification. All models beat this baseline comfortably. Even so, sensitivity on the 75 positive cases is the clinically important figure, and it should be reported.
- The neural network folds were assigned randomly, while the other models used stratified folds. The fold-to-fold variation (for example AUC 0.81–1.00) is large given only about 7 positives per fold.

## Tools

R: `mice` (imputation), `glmnet` (LASSO), `randomForest`, `earth` (MARS), `neuralnet` (ANNs), `caret` + `kernlab` (SVMs), `isofor` (Isolation Forest), `vip` (importance), `cvAUC` and `verification` (ROC/AUC), `ggplot2`, `reshape2`.
