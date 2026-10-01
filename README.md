# Statistical Machine Learning Projects

Eight applied machine-learning projects in **R**, covering regression, classification, unsupervised learning and anomaly detection on real-world data from medicine, e-commerce, HR, meteorology, text and manufacturing.

Each folder has a full project report: problem, data, method, results tables, findings and an honest review of limitations. **The reports describe the analysis, not the code.**

## Projects

| # | Project | Problem | Methods | Headline result |
|---|---|---|---|---|
| 01 | [Early-Stage Diabetes Risk](01-diabetes-risk-regularized-logistic-regression/) | Diagnose diabetes from 16 symptom questions | SEMMA workflow, chi-square / Wilcoxon screening, **SCAD, MCP, LASSO** logistic regression | 5-question SCAD model matches the 13-variable models: **test AUC 0.956** |
| 02 | [Shill Bidding Detection](02-shill-bidding-optimization-and-kernel-trick/) | Flag fraudulent auction bidders | Logistic regression **fitted by hand** (BFGS vs CG, Hessian-based inference), **kernel-trick** classifier | **98.6%** test accuracy, 94.6% of shill bids caught |
| 03 | [Kernel PCA & Association Rules](03-kernel-pca-and-association-rules/) | Compress digit images; mine word patterns in 31k Bible verses | **PCA vs RBF kernel PCA**, **Apriori** with confidence / lift / conviction | 32 of 61 PCs for 90% variance; rules recover grammar (*shalt ⇒ thou*, lift 8.1) |
| 04 | [PageRank & Anomaly Detection](04-pagerank-and-anomaly-detection/) | Rank web pages; find 2 defective parts among 902 | **PageRank**, robust **MCD** distance, **LOF**, **Isolation Forest** | LOF ranks both known defects #1 and #2 |
| 06 | [Employee Attrition](06-employee-attrition-prediction/) | Predict which of 15k employees leave | Logistic (LASSO), **GAM**, **MARS**, **PPR**, **Random Forest** | Random Forest **AUC 0.992** vs 0.822 linear |
| 06 | [Hepatitis C Detection](07-hepatitis-c-classification-ann-svm/) | Detect HCV from 10 blood markers | MICE imputation, 10-fold stratified CV, **neural networks**, **SVMs**, RF, MARS, LASSO | Random Forest **AUC 0.979**, 2.9% error |
| 07 | [Texas Precipitation](08-texas-precipitation-prediction/) | Predict daily rainfall from weather data | Linear, **ridge, LASSO**, **bagging**, **Random Forest**, time-based split | Random Forest cuts test MSE **26%** vs linear |

## Skills Demonstrated

**Statistical modeling:** logistic and penalized regression (LASSO, ridge, SCAD, MCP) · non-linear least squares · GAM · MARS · projection pursuit · splines and kernel smoothing

**Machine learning:** Random Forest · bagging · neural networks · support vector machines (linear and RBF) · KNN · kernel methods

**Unsupervised learning:** PCA and kernel PCA · association rule mining · PageRank · anomaly detection (MCD, LOF, Isolation Forest)

**Workflow:** EDA · missing-data imputation (MICE) · variable screening · train/validation/test and time-based splits · stratified k-fold cross-validation · hyperparameter tuning · ROC/AUC with confidence intervals · odds-ratio interpretation · numerical optimization (BFGS, CG)

## Recurring Lessons

Across these projects, a few themes came up again and again:

1. **Simpler models often tie complex ones.** A 5-variable model matched a 13-variable one (01).
2. **When a linear model loses, it is usually from missing non-linearity rather than overfitting.** Regularization barely moved the error, while trees, GAMs and MARS improved it sharply (05, 07).
3. **The evaluation pipeline matters as much as the model.** 

## Author

**Maxwell Kwesi Mac-Ocloo**
