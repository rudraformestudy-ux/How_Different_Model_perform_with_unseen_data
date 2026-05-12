# Model Robustness Under Distribution Shift: Experimental Analysis
**Dataset:** UCI Credit Card Default | **Models Evaluated:** 7 | **Experiments:** 2

---

## Overview

This analysis evaluates how seven machine learning models behave when the data they are asked to predict on differs from what they were trained on — a condition known as **distribution shift**. Two distinct experiments were conducted to stress-test model robustness from different angles.

---

## Experimental Design

| Experiment | What Was Removed | What It Simulates |
|---|---|---|
| Cluster Removal | A geometric subgroup of samples | An unseen population segment |
| Feature-Region Removal | The 40th–60th percentile of top features | A void in critical decision logic |

The second experiment is substantially harder. Removing a cluster displaces samples. Removing the mid-range of the most predictive features destroys the information the models rely on most to discriminate between classes.

---

## Experiment 1 — Cluster Removal

All models saw a moderate, roughly comparable drop in accuracy when a cluster was withheld.

| Model | ID Accuracy | Interpolation Accuracy | Drop | Calibration Gap |
|---|---|---|---|---|
| Gradient Boosting | 0.740 | 0.632 | ↓ 0.108 | +0.045 |
| Random Forest | 0.737 | 0.615 | ↓ 0.122 | +0.007 |
| XGBoost | 0.738 | 0.615 | ↓ 0.123 | +0.063 |
| LightGBM | 0.739 | 0.610 | ↓ 0.129 | +0.084 |
| Decision Tree | 0.727 | 0.610 | ↓ 0.117 | +0.098 |
| Logistic Regression | 0.725 | 0.607 | ↓ 0.118 | +0.246 |
| KNN | 0.721 | 0.581 | ↓ 0.140 | +0.071 |

**Key observation:** The accuracy drops across models are narrow — ranging from 0.108 to 0.140. If this were the only experiment, a reasonable but incorrect conclusion would be that all seven models are similarly robust. The calibration gap tells a different story. Logistic Regression's confidence exceeded its accuracy by 0.246, meaning it reported roughly 85% confidence while achieving only 60.7% accuracy on unseen clusters.

---

## Experiment 2 — Feature-Region Removal

Removing the 40th–60th percentile of the most important features caused severe divergence between models. The spread in interpolation accuracy widened from 0.051 (Experiment 1) to 0.203 (Experiment 2).

| Model | ID Accuracy | Interpolation Accuracy | Drop | Rank |
|---|---|---|---|---|
| Decision Tree | 0.709 | 0.670 | ↓ 0.039 | 1 |
| Gradient Boosting | 0.739 | 0.659 | ↓ 0.080 | 2 |
| LightGBM | 0.743 | 0.644 | ↓ 0.099 | 3 |
| XGBoost | 0.737 | 0.642 | ↓ 0.095 | 4 |
| Random Forest | 0.733 | 0.596 | ↓ 0.137 | 5 |
| Logistic Regression | 0.711 | 0.473 | ↓ 0.238 | 6 |
| KNN | 0.714 | 0.467 | ↓ 0.247 | 7 |

Logistic Regression and KNN both fell below 0.500 — worse than random guessing on a binary classification problem.

---

## Core Findings by Model

### Gradient Boosting — Most Robust Overall

Gradient Boosting recorded the smallest accuracy drop in Experiment 1 (↓ 0.108) and the second-smallest in Experiment 2 (↓ 0.080). More notably, its interpolation accuracy actually improved from 0.632 under cluster removal to 0.659 under the harder feature-region removal. Its confidence under feature removal sat at 0.651 against an accuracy of 0.659 — a calibration gap of −0.008, meaning it was marginally underconfident. This is the safest failure mode: the model hedges rather than overcommits.

### Decision Tree — Most Surprising Result

The simplest model in the set produced the most stable result under feature-region removal, dropping only 0.039 points compared to a 0.247 drop for KNN. Its confidence under feature removal was 0.693 against accuracy of 0.670 — a gap of just 0.023. The likely explanation is that a depth-constrained decision tree creates coarse, broad partitions. When test points fall into the missing region, they route into wide leaves that cover adjacent space rather than into highly specific regions that no longer exist.

### Logistic Regression — Confidently Wrong

Under feature removal, Logistic Regression's accuracy collapsed to 0.473 while its confidence held at 0.662 — a gap of 0.189. The model extended its global linear boundary straight through the feature void without recognizing the data had changed. This is the most operationally dangerous failure mode: the model does not degrade gracefully. It continues to report high confidence while performing below chance.

### KNN — Worst Overconfidence

KNN produced the largest calibration gap in the experiment: confidence 0.745 against accuracy 0.467 under feature-region removal — a gap of 0.278. The mechanism is clear. KNN requires nearby training points to make predictions. By removing the 40th–60th percentile of top features, the training neighborhood for those test points was eliminated. The model located the geometrically nearest neighbors, but those neighbors belonged to structurally incorrect class regions. High confidence, near-random accuracy.

### Random Forest — Stable but Not Robust

Random Forest maintained reasonable calibration across both experiments (gaps of +0.007 and +0.035). However, its interpolation accuracy under feature removal (0.596) was notably below Gradient Boosting (0.659), XGBoost (0.642), and LightGBM (0.644). Averaging over many trees improves smoothness relative to KNN but does not produce the additive boundary correction that boosting provides. It falls in the middle: better than local methods, worse than sequential ensembles.

### XGBoost and LightGBM — Reliable, Slightly Aggressive

Both performed closely. XGBoost: interpolation 0.642, calibration gap +0.018. LightGBM: interpolation 0.644, calibration gap +0.015. Their sharper optimization gives stronger in-distribution performance (LightGBM ID: 0.743, highest in the set) but produces marginally less smooth boundaries than vanilla Gradient Boosting, which costs a small amount of robustness in the feature void.

---

## The Central Diagnostic: ID Accuracy Hides Failure

| Model | ID Accuracy (Exp. 2) | Interpolation Accuracy | Difference |
|---|---|---|---|
| Gradient Boosting | 0.739 | 0.659 | 0.080 |
| LightGBM | 0.743 | 0.644 | 0.099 |
| KNN | 0.714 | 0.467 | 0.247 |
| Logistic Regression | 0.711 | 0.473 | 0.238 |

All four models above have ID accuracy within 0.032 of each other. A standard train/test evaluation would report them as near-equivalent. The interpolation experiment reveals a 0.192-point gap between the best and worst performers in the missing region. Standard evaluation metrics would completely conceal this divergence.

---

## Calibration Summary

| Model | Interp Confidence | Interp Accuracy | Gap | Risk Assessment |
|---|---|---|---|---|
| KNN | 0.745 | 0.467 | +0.278 | Critical |
| Logistic Regression | 0.662 | 0.473 | +0.189 | High |
| Decision Tree | 0.693 | 0.670 | +0.023 | Low |
| Gradient Boosting | 0.651 | 0.659 | −0.008 | Very Low |
| Random Forest | 0.631 | 0.596 | +0.035 | Low |
| XGBoost | 0.660 | 0.642 | +0.018 | Very Low |
| LightGBM | 0.659 | 0.644 | +0.015 | Very Low |

Models with a large positive gap are not just inaccurate — they are inaccurate while reporting high confidence. In a deployed system, these models provide no signal that anything has gone wrong.

---

## Final Ranking

| Rank | Model | Interpolation (Exp. 2) | Calibration Gap | Assessment |
|---|---|---|---|---|
| 1 | Gradient Boosting | 0.659 | −0.008 | Best robustness and calibration |
| 2 | Decision Tree | 0.670 | +0.023 | Unexpectedly stable under feature void |
| 3 | LightGBM | 0.644 | +0.015 | Strong and well-calibrated |
| 4 | XGBoost | 0.642 | +0.018 | Strong and well-calibrated |
| 5 | Random Forest | 0.596 | +0.035 | Moderate robustness |
| 6 | Logistic Regression | 0.473 | +0.189 | Fails silently and confidently |
| 7 | KNN | 0.467 | +0.278 | Most dangerous under shift |

---

## Inferences

**1. Feature-region voids are more destructive than population voids.** Accuracy spread across models was 0.051 under cluster removal and 0.203 under feature removal. The type of shift matters more than the presence of shift.

**2. Standard test accuracy is insufficient for robustness evaluation.** All models recorded ID accuracy between 0.709 and 0.743 — a 0.034 range. Their interpolation accuracy ranged from 0.467 to 0.670 — a 0.203 range. The same models that looked equivalent on standard metrics diverged dramatically under shift.

**3. Model architecture determines failure mode.** Linear models fail because their global boundary extends through the void with no adjustment. Distance-based models fail because their local neighborhood disappears. Boosted ensembles partially compensate using secondary features. The inductive bias of the model directly predicts how it breaks.

**4. Calibration failure compounds accuracy failure.** A model that drops to 0.467 accuracy and reports 0.467 confidence is informative — it signals uncertainty. A model that drops to 0.467 accuracy and reports 0.745 confidence is dangerous — it actively misleads. KNN and Logistic Regression exhibited the second pattern.

**5. For this dataset, the mid-range of top features carries most of the predictive signal.** Removing the 40th–60th percentile of features like PAY_0 was sufficient to reduce the two worst models to below-random performance. This confirms that the UCI Credit dataset's discriminatory power is concentrated in a narrow band of its most important features — not distributed uniformly.
