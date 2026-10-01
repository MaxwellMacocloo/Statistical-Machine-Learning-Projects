# Kernel PCA on Handwritten Digits and Association Rules in the Bible

**Language:** R  **Date:** 2022  **Type:** Unsupervised learning, dimension reduction, frequent-pattern mining

## Overview

This project has two unsupervised-learning studies:

1. **Dimension reduction.** Compares ordinary principal component analysis (PCA) with **kernel PCA** (RBF kernel) on 64-pixel images of handwritten digits, and checks whether components learned on training images carry over to unseen test images.
2. **Association rule mining.** Treats every verse of the King James Bible as a "shopping basket" of words and mines it with the **Apriori** algorithm. Rules are ranked by confidence, lift and conviction, and the strengths and weaknesses of each measure are discussed.

---

## Part 1: PCA vs kernel PCA on handwritten digits

### Data

| Item | Detail |
|---|---|
| Source | UCI Machine Learning Repository, *Optical Recognition of Handwritten Digits* |
| Format | 8×8 grid of pixel counts (0–16) = 64 features, plus the digit label 0–9 |
| Size | Training set 3,823 images; test set 1,797 images |
| Missing values | None |

### Methodology

- Removed the label and all constant (unary) pixel columns. Pixels at the image border are always blank, so this left **61 features**.
- Standardized the features (mean 0, SD 1).
- Visualized the raw data with parallel boxplots and heatmaps of images sorted by digit.
- **Ordinary PCA:** scree plot, cumulative variance and PC1 vs PC2 scatter labelled by digit.
- **Kernel PCA:** Gaussian RBF kernel (σ = 0.001), first 25 kernel components, then the same scatter plot.
- Projected the test images onto both sets of components to check how well they generalize.

### Results

| Measure | Ordinary PCA |
|---|---|
| Variance explained by PC1 | 11.8% |
| Variance explained by PC1 + PC2 | 22.5% |
| Components needed for ≈ 50% | 8 |
| Components needed for ≥ 90% | **32** (of 61) |

- The pixel features are far from normal: they are heavily skewed, with many zeros.
- **No dominant direction.** Variance is spread thinly across many components, so a 2-D PCA view keeps only about a fifth of the information.
- In the PC1 vs PC2 plot, images of the same digit form partly separated groups, but many overlap heavily near the origin. Digit 7 shows the widest spread.
- **Kernel PCA** gave visibly tighter per-digit groups in its first two components than linear PCA, though the groups were still not fully separated. The leading kernel eigenvalues (0.0123, 0.0113, 0.0086) decay slowly, so here too no small set of components dominates.
- Test-set projections reproduce the cluster layout of the training projections for both PCA and kernel PCA, with slightly more spread. The learned components generalize to new handwriting.

---

## Part 2: Association rules in the King James Bible

### Data

| Item | Detail |
|---|---|
| Transactions | 31,101 verses |
| Items | 13,978 distinct words |
| Density | 0.085% (very sparse) |
| Verse length | median 11 words, mean 11.9, max 36 |
| Most frequent words | *lord* (6,555 verses, 21.1%), *thou* (3,827), *god* (3,814), *said* (3,532), *thy* (2,996) |

### Methodology

- Apriori with **minimum support 0.01** (a word set must appear in at least 311 verses), **minimum confidence 0.6** and **maximum rule length 5**.
- 222 words passed the support threshold, giving **18 rules** (8 of length two, 10 of length three).
- Ranked the rules by confidence, by lift and, after lowering confidence to 0.09 (213 rules), by **conviction**.

### Results

**Top 5 by confidence**

| Rule | Support | Confidence | Lift |
|---|---|---|---|
| {shalt, thee} ⇒ {thou} | 0.012 | 1.000 | 8.13 |
| {shalt, thy} ⇒ {thou} | 0.015 | 1.000 | 8.13 |
| {shalt} ⇒ {thou} | 0.038 | 0.999 | 8.12 |
| {lord, shalt} ⇒ {thou} | 0.012 | 0.997 | 8.10 |
| {art} ⇒ {thou} | 0.014 | 0.989 | 8.04 |

**Top 5 by lift**

| Rule | Support | Confidence | Lift |
|---|---|---|---|
| {lord, thus} ⇒ {saith} | 0.014 | 0.859 | **22.50** |
| {thus} ⇒ {saith} | 0.015 | 0.651 | 17.07 |
| {she} ⇒ {her} | 0.013 | 0.602 | 16.40 |
| {shalt, thee} ⇒ {thou} | 0.012 | 1.000 | 8.13 |
| {shalt, thy} ⇒ {thou} | 0.015 | 1.000 | 8.13 |

**Top by conviction:** {shalt, thee} ⇒ {thou} (∞), {shalt, thy} ⇒ {thou} (∞), {shalt} ⇒ {thou} (1,045), {lord, shalt} ⇒ {thou} (324), {art} ⇒ {thou} (78), {hast} ⇒ {thou} (52).

### Why conviction?

| Measure | Weakness |
|---|---|
| **Confidence** P(B \| A) | Ignores how common B already is. Any rule ending in *lord* looks strong simply because *lord* appears in 21% of verses. |
| **Lift** P(A,B) / P(A)P(B) | Corrects for B's frequency but is **symmetric** (lift(A⇒B) = lift(B⇒A)), so it cannot say which way the implication runs. |
| **Conviction** (1 − P(B)) / (1 − conf(A⇒B)) | Accounts for B's frequency **and** is directional. It measures how often the rule would be wrong if A and B were independent, relative to how often it is actually wrong. A rule that never fails has infinite conviction. |

### Key findings

- **The rules recover Early Modern English grammar.** *shalt*, *art* and *hast* almost always come with *thou*, the subject they conjugate with. The rule mining "learned" verb agreement with no linguistic input.
- **The highest-lift rule is a prophetic formula.** {lord, thus} ⇒ {saith} reflects the stock phrase *"Thus saith the LORD"*. Its lift of 22.5 means the pattern occurs 22 times more often than chance.
- **Conviction picks out the truly deterministic rules.** It separates the rules that essentially never fail from rules that are merely frequent.

## Notes on the Original Analysis

- The original text said "the last 30 PCs" explain 90% of variance. The output shows the **first 32** components are needed to reach 90%. The table above follows the output.
- The test set was standardized with its own mean and SD. Strictly, the training-set statistics should be reused, so both sets sit on the same scale.

## Tools

R: `prcomp` (PCA), `kernlab` (kernel PCA), `arules` (Apriori, interest measures).
