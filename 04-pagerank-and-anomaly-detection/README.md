# PageRank and Multivariate Anomaly Detection

**Language:** R  **Date:** 2022  **Type:** Graph ranking, unsupervised outlier detection

## Overview

This project has two parts:

1. **PageRank.** Ranks seven linked web pages by importance using Google's PageRank algorithm.
2. **Anomaly detection.** Finds two known defective parts among 902 manufactured high-tech parts, each measured on 88 tests. Three approaches are compared: robust Mahalanobis distance (MCD), the Local Outlier Factor (LOF) and Isolation Forest.

---

## Part 1: PageRank on a 7-page web graph

### Setup

- Built a 7×7 link matrix for pages A–G, following the textbook convention that entry (i, j) = 1 means page *j* links to page *i*.
- The links are: A→C, A→D · B→A · C→D · D→B, D→E · E→B, E→D, E→G · F→C, F→D, F→G. G has no outgoing links.
- Converted the matrix to a directed graph and computed PageRank with damping factor **0.85**, using the PRPACK solver.

### Results

| Rank | Page | PageRank score | Incoming links |
|---|---|---|---|
| 1 | **D** | **0.260** | 4 (A, C, E, F) |
| 2 | A | 0.186 | 1 (B) |
| 3 | B | 0.182 | 2 (D, E) |
| 4 | E | 0.142 | 1 (D) |
| 5 | C | 0.119 | 2 (A, F) |
| 6 | G | 0.080 | 2 (E, F) |
| 7 | F | 0.031 | 0 |

### Findings

- **D ranks first** because it receives the most links, and some come from well-ranked pages.
- **A outranks C and G, even though they have more incoming links.** A's only link comes from B, but B links to nothing else, so A receives B's whole vote. PageRank measures the *quality* of incoming links, not just how many there are.
- **F scores lowest** because nothing links to it. Its score comes entirely from the random-jump ("teleport") term.

---

## Part 2: Finding defective parts

### Data

| Item | Detail |
|---|---|
| Source | `HTP` dataset (R package `ICSOutlier`): high-tech parts |
| Size | 902 parts × 88 numeric test measurements |
| Ground truth | Parts **581** and **619** are known to be defective |

### Methods

| Method | Configuration |
|---|---|
| **Robust Mahalanobis distance** | Minimum Covariance Determinant (MCD) estimates of center and covariance, using 75% of the data (about 25% breakdown point). Each part's distance is compared with two cut-offs: the 97.5% chi-square quantile (115.8) and the Green–Martin finite-sample cut-off (149.9). |
| **Local Outlier Factor (LOF)** | Density compared with the k = 10 nearest neighbors. The top 1% are flagged. |
| **Isolation Forest** | Random isolation trees score how easy each part is to isolate. The top 1% are flagged. |

### Results

| Method | Part 619 | Part 581 | Separation from other parts |
|---|---|---|---|
| Robust Mahalanobis (MCD) | Flagged; one of only two extreme points (distance > 20,000, with part 303) | Flagged above both cut-offs, but not extreme | Weak. In 88 dimensions many parts exceed the cut-offs. |
| **LOF (k = 10)** | **Highest score** | **Second highest** | **Clear.** The two defects stand well apart. |
| Isolation Forest | Highest score | In top 1% | Moderate. Mixed in with 642, 460, 852, 644, 69, 303 and 428. |

### Findings

- **LOF worked best.** It put the two known defects at the very top, clearly separated from everything else.
- **Isolation Forest caught both defects** but ranked them among about nine candidates, so an inspector would have to check several healthy parts too.
- **The robust distance flags too many parts at this dimensionality.** With 88 variables and 902 parts, even the more accurate Green–Martin cut-off flags many parts. The distance is still useful for *ranking*, and it put 619 at the very top.
- **Part 303 deserves a look.** All three methods flagged it, so it may be a third defect or a measurement problem.

## Notes on the Original Analysis

- The section heading mentions a one-class SVM, but no one-class SVM was fitted. The comparison covers MCD, LOF and Isolation Forest only.

## Tools

R: `igraph` (graph construction, PageRank), `robustbase` (MCD), `CerioliOutlierDetection` (Green–Martin cut-off), `Rlof` (LOF), `IsolationForest`.
