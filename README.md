# Model Robustness Under Distribution Shift
**Dataset:** UCI Credit Card Default | **Models:** 7 | **Experiments:** 3

---

## What This Project Does

This project stress-tests seven classical ML models under three progressively severe forms of distribution shift. The goal is to measure not just how much accuracy drops, but whether model confidence remains trustworthy when predictions are made outside the training distribution.

---

## Experiments

| # | Experiment | What Was Removed | Shift Type |
|---|---|---|---|
| 1 | Cluster Removal | A geometric subgroup of samples | Semantic / population |
| 2 | Feature-Region Removal | 40th–60th percentile of top features | Decision-feature |
| 3 | PCA Square Removal | A square region in principal component space | Structural / latent-space |

---

## Models Evaluated

Logistic Regression · K-Nearest Neighbors · Decision Tree · Random Forest · Gradient Boosting · XGBoost · LightGBM

---

## Key Results

All 7 models scored between **0.721–0.740** on standard in-distribution accuracy — close enough to look equivalent.

Under distribution shift, the picture changed:

- **Experiment 1** — Interpolation accuracy spread: **0.032** (models still look similar)
- **Experiment 2** — Interpolation accuracy spread: **0.203** (models diverge sharply)
- **Experiment 3** — Every model collapsed to **0.480–0.558**, near random chance

The accuracy divergence grew **6× wider** under shift compared to standard evaluation.

---

## Core Finding

As experiments got harder, average accuracy dropped from **0.12 → 0.14 → 0.16**. Average confidence drop stayed near **zero** across all three experiments.

> **Confidence encodes decision geometry, not data support. Models do not know when they are predicting in unsupported regions.**

---

## Worst Calibration Gaps (Confidence − Accuracy in Interpolation Zone)

| Model | Worst Gap | Experiment |
|---|---|---|
| Decision Tree | +0.485 | PCA Square |
| KNN | +0.293 | PCA Square |
| XGBoost | +0.253 | PCA Square |
| Logistic Regression | +0.246 | Cluster Removal |
| Random Forest | +0.190 | PCA Square |
| Gradient Boosting | +0.144 | PCA Square |

---

## Best Model

**Gradient Boosting** — highest interpolation accuracy in Experiments 1 and 2, smallest calibration gap (+0.144) in Experiment 3. Most reliable confidence-accuracy relationship across all three experiments.

---

## Main Takeaway

Standard test accuracy is insufficient for robustness evaluation. Models that appear equivalent on normal metrics can fail by completely different mechanisms and magnitudes under shift. Addressing this properly requires explicit uncertainty quantification — conformal prediction, Bayesian methods, or density-based OOD detection.
