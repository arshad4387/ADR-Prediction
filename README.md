This project predicts Adverse Drug Reactions (ADRs) using Non-negative Matrix Factorization (NMF), Weighted NMF, and Kernel-based machine learning models. The approach integrates:
Drug–Gene Interaction features
Chemical fingerprints
ADR labels
The goal is to predict missing ADRs for drugs using low-rank latent factor models combined with kernel regression.

Dataset Summary
Drug–Gene Interaction Matrix
Shape: 973 × 3647
Rows: Drugs
Columns: Genes
Binary interaction matrix
Chemical Fingerprints
Shape: 973 × 1024
Source: PubChem
Binary fingerprint representation
ADR Matrix
Shape: 5765 × 973
Rows: ADRs
Columns: Drugs
Sparsity: 0.977 (Highly sparse)
Train/Test split:
Train drugs: 730
Test drugs: 243
