# Adverse Drug Reaction Prediction

**Authors:** Arshad Ahamed Shajahan, Sai Shasank Dandu  
**Supervisor:** Dr. Davood Roshan

---

## Overview

Adverse drug reactions are one of the leading causes of patient harm worldwide, yet detecting them early — before a drug reaches widespread use — remains a genuinely difficult problem. The challenge is not just biological; it is mathematical. The datasets involved are highly sparse, the positive-to-negative class ratio is severely imbalanced, and the feature spaces involved (genomic targets, chemical structure) are large relative to the number of known drugs.

This project approaches ADR prediction as a matrix completion problem. Given a partially observed drug-side-effect matrix, the goal is to predict which side effects an unseen drug is likely to cause, using two sources of prior information: the drug's interactions with human genes, and its chemical fingerprint. We implemented and compared several models — from a simple frequency-based baseline up to weighted Non-negative Matrix Factorization with kernel regression — and assessed each using five-fold cross-validation across 973 drugs.

The project also extends an earlier published dataset by roughly 25%, adds methods not covered in the original research, and delivers a working web application for single-drug ADR prediction.

---

## Motivation

When a new drug enters clinical trials, its side-effect profile is largely unknown. Experimental approaches for discovering ADRs are slow and expensive. Computational prediction using publicly available genomic and chemical data offers a cheaper, faster first pass — flagging likely adverse effects before they are observed in patients.

Two practical obstacles make this harder than a standard classification task:

- **Sparse data.** The drug-side-effect matrix has a sparsity of around 0.977. Most entries are unknown, not negative.
- **Class imbalance.** Among the entries that are known, positive cases (confirmed ADRs) are heavily outnumbered.

The models chosen here were specifically selected or adapted to handle both issues.

---

## Datasets

All datasets were converted to binary matrix format before use.

| Dataset | Shape | Source | Notes |
|---|---|---|---|
| Drug–Gene Interaction | 973 x 3647 | STITCH / DGIdb | Binary interaction matrix |
| Chemical Fingerprints | 973 x 1024 | PubChem (via PubChemPy + RDKit) | Morgan fingerprints, binary |
| ADR Matrix | 5765 x 973 | SIDER | Sparsity: 0.977 |

**Train/test split:** 730 drugs for training, 243 held out for testing.

Chemical fingerprints were retrieved using SMILES strings fetched from PubChem via the PubChemPy library. RDKit was then used to generate 1024-bit Morgan fingerprints for each drug.

The dataset was extended by approximately 25% compared to the original research this project builds on, by pulling additional drug entries from the same public sources.

---

## Models

### Naive Baseline

A frequency-only model that ignores chemical and genomic features entirely. It predicts a side effect for every drug based purely on how often that side effect appears in the training set. This serves as a sanity check — any meaningful model should outperform it on precision-recall metrics.

### Kernel Regression

A similarity-based approach. For each test drug, a similarity score (between 0 and 1) is computed against every training drug using either a Gaussian (RBF) or linear kernel applied to the drug-gene or chemical fingerprint features. The predicted side-effect profile is then a weighted combination of the training drugs' known profiles.

### Support Vector Machine (Linear and RBF)

Rather than predicting per drug, SVM operates per side effect — for each ADR, it trains a binary classifier to distinguish drugs that cause it from those that do not. Two kernel variants were tested: linear and RBF, where the RBF kernel projects the data into a higher-dimensional space before fitting the decision boundary.

### Weighted NMF with Kernel Regression (VKR Weighted NMF)

Non-negative Matrix Factorization decomposes the drug-side-effect matrix into two lower-rank matrices: a side-effect profile matrix and a weight matrix. The number of latent features controls the compression. To handle class imbalance, a 1:10 weighting scheme penalises missed positive cases more heavily than false positives. The factorisation runs for 500 iterations. At prediction time, kernel regression maps a new drug's feature vector into the learned latent space, and the side-effect profile is reconstructed from there.

This model trades AUROC for precision-recall performance, which is the more meaningful metric given the extreme class imbalance.

### VKR NMF and VKR SVD

Variants of the above using standard (unweighted) NMF and singular value decomposition respectively, included for comparison.

---

## Results

Five-fold cross-validation across all 973 drugs. AUROC measures overall discrimination; AUPR is more informative here given the sparsity and imbalance.

| Model | AUROC Mean (SD) | AUPR Mean (SD) |
|---|---|---|
| Naive Baseline | 0.9101 (0.0024) | 0.3584 (0.0075) |
| Kernel Regression | 0.8745 (0.0051) | 0.4155 (0.0051) |
| SVM Linear | 0.7955 (0.0097) | 0.3122 (0.0124) |
| SVM RBF | 0.9001 (0.0041) | 0.3919 (0.0068) |
| VKR NMF | 0.9183 (0.0027) | 0.4230 (0.0033) |
| VKR SVD | 0.9098 (0.0033) | 0.4341 (0.0049) |
| VKR Weighted NMF | 0.8551 (0.0035) | **0.5001 (0.0022)** |

The Weighted NMF model achieves the highest AUPR at 0.5001, a notable improvement over the naive baseline's 0.3584. This confirms that applying class weighting during factorisation meaningfully improves the model's ability to rank true positive ADRs, even at the cost of some overall AUROC. VKR NMF achieves the highest AUROC (0.9183) but lags on AUPR, which reinforces why AUPR is the better metric to optimise in this setting.

---

## Application

A working web application was built to allow single-drug ADR prediction through a browser interface.

- **Backend:** FastAPI — handles model inference via REST endpoints
- **Frontend:** Streamlit — interactive dashboard for drug input and result display
- **Current deployment:** Localhost (cloud deployment via Streamlit Cloud is supported)

The user enters a drug name, the application retrieves its SMILES string from PubChem, generates chemical fingerprints, and returns a ranked list of predicted adverse reactions with associated confidence scores.

---

## Technical Stack

**Languages:** Python, R

**Key libraries:**

| Library | Purpose |
|---|---|
| RDKit | Chemical fingerprint generation from SMILES |
| PubChemPy | Drug and SMILES retrieval from PubChem |
| NumPy / Pandas | Data processing and matrix operations |
| FastAPI | Backend API server |
| Streamlit | Frontend dashboard |

---

## Limitations and Future Work

The current system has a few known constraints worth being upfront about:

- Prediction is limited to drugs that already exist in PubChem. New or rare compounds with no database record cannot be processed.
- Only single-drug ADR prediction is supported. Drug-drug interaction effects are out of scope.

Planned extensions include integration with live pharmacological databases and multi-drug interaction modelling.

---

## Repository Structure
