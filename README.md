# HRfH-2026-Team-4-Task-2 — Fantastic 4 (Group 4)
https://health-research-from-home.github.io/HRfH-Hackathon-2026/
## The Task

We were asked to develop an analysis approach that uses daily step counts to identify post-treatment recovery trajectory groups (fast, moderate, slow), and to assess how recovery group identification changes under different missingness settings.

An online simulator (Shiny) [https://hrfh-hackathon-2026.shinyapps.io/hackathon2026/] that generates 365-day daily step count trajectories with configurable missingness mechanisms and severities was provided. We were given no underlying simulator code, only the output CSV files.

The four guiding questions from the brief:

1. How well can the recovery groups be identified when there is no missingness or very little missingness?
2. How does performance change as the amount of missingness increases?
3. Does performance differ when missingness occurs in different ways (MCAR, MAR, MNAR)?
4. What level or type of missingness appears to make recovery group identification unreliable?

---

## Repository Structure 

| File | Purpose |
|---|---|
| `Hackathon_task_2.ipynb` | **Main analysis** — Phase A (6-feature model) + Blooper 1 + Diagnostic |
| `BLooper1.ipynb` | Standalone version of the initial 8-feature attempt (for inspection) |
| `BLooper2.ipynb` | Data leakage demonstration (train on same cohort as test) |
| `Sensitivity_Experiments.ipynb` | Three perturbation experiments (A: sensitive features, B: fragile features, C: noisy data) |
| `README.md` | This file |
| `FINAL_SUMMARY.md` | Detailed bilingual final summary |
| `PHASE3_DATASETS_16_COMBINATIONS.md` | Dataset specification (23 files) |
| `TASK2 DATA/` | All datasets from the simulator |

---

## Methods 

### Dataset Setup 

We develop a three-phase validation design using **23 datasets** generated from the simulator:

- **2 complete datasets** (0% missingness) for training (seed=50) and external validation (seed=42), generated with different random seeds to create independent cohorts
- **21 missing-data datasets** (seed=42) = 7 missingness levels × 3 mechanisms

The 7 missingness levels are: 10%, 25%, 50%, 75%, 90%, 95%, 99%. The 3 mechanisms are:

| Mechanism | Behaviour | Realistic interpretation |
|---|---|---|
| **MCAR** (Missing Completely At Random) | Random sensor failures, with `gap_length=50` | Device battery dying, wearer removing the device for grooming |
| **MAR** (Missing At Random) | Concentrated in early recovery (`early_effect=70`, `BMI/age=50`) | Post-operative compliance is worse in the first weeks |
| **MNAR** (Missing Not At Random) | Biased toward low-activity days (`sensitivity=70`, `time_window=30 days`) | Sedentary patients are less likely to wear the device |

We hold all other simulator parameters constant (seed within each cohort, 365 days, 300 patients per cohort, age=68, BMI=28) so that we are measuring the effect of missingness alone.


#### Full Dataset List

| Done | Dataset | File Name | Missingness Level | Mechanism | Gap Length | Early Effect | BMI/Age | Low-Activity Sensitivity | Time Window |
|---|---|---|---|---|---|---|---|---|---|
| ✓ seed=42 | 1-1 | 1-1_0_complete | 0% | — | — | — | — | — | — |
| ✓ seed=50 (training) | 1-2 | 1-2_0_complete | 0% | — | — | — | — | — | — |
| ✓ seed=42 | 2 | 2_10MCAR | 10% | MCAR | 50 | — | — | — | — |
| ✓ | 3 | 3_10MAR | 10% | MAR | — | 70 | 50 | — | — |
| ✓ | 4 | 4_10MNAR | 10% | MNAR | — | — | — | 70 | 30 days |
| ✓ | 5 | 5_25MCAR | 25% | MCAR | 50 | — | — | — | — |
| ✓ | 6 | 6_25MAR | 25% | MAR | — | 70 | 50 | — | — |
| ✓ | 7 | 7_25MNAR | 25% | MNAR | — | — | — | 70 | 30 days |
| ✓ | 8 | 8_50MCAR | 50% | MCAR | 50 | — | — | — | — |
| ✓ | 9 | 9_50MAR | 50% | MAR | — | 70 | 50 | — | — |
| ✓ | 10 | 10_50MNAR | 50% | MNAR | — | — | — | 70 | 30 days |
| ✓ | 11 | 11_75MCAR | 75% | MCAR | 50 | — | — | — | — |
| ✓ | 12 | 12_75MAR | 75% | MAR | — | 70 | 50 | — | — |
| ✓ | 13 | 13_75MNAR | 75% | MNAR | — | — | — | 70 | 30 days |
| ✓ | 14 | 14_90MCAR | 90% | MCAR | 50 | — | — | — | — |
| ✓ | 15 | 15_90MAR | 90% | MAR | — | 70 | 50 | — | — |
| ✓ | 16 | 16_90MNAR | 90% | MNAR | — | — | — | 70 | 30 days |
| ✓ | 17 | 17_95MCAR | 95% | MCAR | 50 | — | — | — | — |
| ✓ | 18 | 18_95MAR | 95% | MAR | — | 70 | 50 | — | — |
| ✓ | 19 | 19_95MNAR | 95% | MNAR | — | — | — | 70 | 30 days |
| ✓ | 20 | 20_99MCAR | 99% | MCAR | 50 | — | — | — | — |
| ✓ | 21 | 21_99MAR | 99% | MAR | — | 70 | 50 | — | — |
| ✓ | 22 | 22_99MNAR | 99% | MNAR | — | — | — | 70 | 30 days |
| — | — | (100% = N/A) | 100% | any | — | — | — | — | — |

### Clustering Method 

We use **K-means clustering** with K=3 (matching the simulator's 3 recovery groups), `random_state=42` for reproducibility, and `n_init=10` to mitigate sensitivity to initialisation.

Why K-means?

- **Interpretable**: cluster centroids correspond to "typical" patients of each group
- **Reproducible**: deterministic given seed
- **Computationally cheap**: ~300 participants × 6 features finishes in milliseconds
- **Standard**: well-understood and accepted by clinical reviewers

Since K-means is unsupervised, we use the **Hungarian-style label permutation** approach: K-means produces labels 0/1/2 with no inherent meaning, so we try all 3! = 6 possible mappings to the true labels (fast/moderate/slow) and select the mapping that maximises accuracy. This is standard practice for evaluating unsupervised clustering against known ground truth.

**Feature engineering**: each participant's 365-day trajectory is compressed into **6 summary features**:

| Feature | Computation | Why |
|---|---|---|
| `early_speed_log` | `np.log1p(slope of days 1–60)` | How fast they start recovering (log to control skewness) |
| `mid_plateau` | Mean steps on days 61–180 | Stabilisation level |
| `late_level` | Mean steps on days 181–365 | Final plateau |
| `max_steps` | Peak daily value | Activity ceiling |
| `variability` | Day-to-day standard deviation | Consistency |
| `recovery_consistency` | Correlation with ideal sigmoid | Trajectory shape quality |

After standardisation (StandardScaler fitted on training only), K-means is trained on the resulting 300×6 matrix.

<img width="1590" alt="Recovery trajectories by group" src="https://github.com/user-attachments/assets/05526507-11df-4c4a-b49f-bbb378f40423" />

### Testing Methodology 

We use a **strict cross-cohort design** to avoid data leakage:

| Phase | Dataset | Purpose | Cohort |
|---|---|---|---|
| Phase 1 | `1-2_0_complete.csv` | Train K-means and lock model | Cohort C (seed 50) |
| Phase 2 | `1-1_0_complete.csv` | External validation on complete data | Cohort B (seed 42) |
| Phase 3 | 21 missing-data CSV files | Sensitivity to missingness | Cohort B with missingness applied |

Critical: the **training cohort (C) is different from the testing cohort (B)**. The Phase 3 missing-data files are derived from Cohort B, so they share the same underlying patients as Phase 2 (just with data removed), but never appear in the training data.

We **lock the model after training** using `joblib.dump()`. All subsequent uses (Phases 2 and 3) use `joblib.load()` followed by `scaler.transform()` and `kmeans.predict()` — never `fit_transform()` or `fit_predict()`. This guarantees the model never sees test data during training.

<img width="467" alt="Training accuracy" src="https://github.com/user-attachments/assets/6d375c9f-84c6-46d7-acd8-24fe1f1c254f" />

<img width="472" alt="External validation accuracy" src="https://github.com/user-attachments/assets/8d140111-0c71-42ce-9819-1e270658c996" />

<img width="1200" alt="Phase A overall flow" src="https://github.com/user-attachments/assets/df893d31-a3d5-4c41-9bb3-feaf475e1f82" />

### Sensitivity Analysis 

To verify that our main result's robustness reflects real model quality rather than artifacts, we develop three perturbation experiments in `Sensitivity_Experiments.ipynb`:

| Experiment | Modification | Purpose |
|---|---|---|
| **A: Sensitive features only** | Use only `early_speed_log` (1 feature instead of 6) | Test whether robustness depends on feature redundancy |
| **B: Fragile features only** | Drop `max_steps`, `mid_plateau`, `late_level` (the most robust features); keep `early_speed_log`, `variability`, `recovery_consistency` | Test which specific features are responsible for robustness |
| **C: Noisy data** | Add Gaussian noise (std=500 steps) to every step count | Test how robustness changes under realistic sensor noise |

Each experiment is run through the full pipeline (training, external validation, missing-data tests) and the degradation curves are compared against the main Phase A model.


<img width="1488" alt="Sensitivity experiments comparison" src="https://github.com/user-attachments/assets/3675e55c-3c5b-4f25-9e4d-5052bd83f4be" />

---

## Results: The Bloopers 

### Blooper 1: The 8-Feature Distribution-Shift Failure 

#### What looked OK but was actually broken 

We started with **8 features** chosen by clinical domain knowledge:

```
['early_speed', 'mid_plateau', 'late_level', 'recovery_slope',
 'max_steps', 'variability', 'recovery_consistency', 'data_completeness']
```

The training accuracy on the new cohort C looked acceptable (~93%), so we initially thought the model was fine. **The real problem only surfaced under missing data tests**: the 8-feature model degrades much faster than our improved 6-feature Phase A:

| Missingness | Blooper 1 (8 features) | Phase A (6 features) | Difference |
|---|---|---|---|
| 10% | ~93% | ~96% | -3% |
| 50% | ~87% | ~95% | **-8%** |
| 75% | ~79% | ~92% | **-13%** |
| 90% | ~67–72% | ~80–92% | **-13–25%** |
| 99% | ~61–70% | ~75–92% | **-14–22%** |

**Baseline accuracy was misleading.** The 8-feature model passed inspection on training data but failed badly under distribution shift (missing data).

#### How we detected it 

We developed a systematic diagnostic check on the features:

**Check 1: Post-standardisation standard deviation**

```python
print(X_scaled.std(axis=0))
# Output: [1. 1. 1. 1. 1. 1. 1. 0.]
#                                 ↑
#                          data_completeness std = 0!
```

`data_completeness` is constant=1.0 in complete training data, so its std after scaling is 0. The feature does no harm in training (everyone the same), but under missing data where it varies (0.33–0.91), the scaler produces extreme out-of-distribution values that distort the K-means distance calculation.

**Check 2: Feature monotonicity across true groups**

```python
for feature in feature_cols:
    means = features.groupby('true_group')[feature].mean()
```

`recovery_slope` is non-monotonic:

- fast: 12.10
- **moderate: 14.73** ← higher than fast!
- slow: 7.53

Moderate patients have higher recovery slope than fast patients (biologically implausible). The cause: `np.polyfit` with `fillna(0)` produces a misleadingly steep slope when zero-filled days are mixed with recovery days. Under missing data, this misleading signal is amplified.

**Check 3: Feature distribution skewness**

`early_speed` is severely skewed:

- fast: 95.19
- moderate: 1.73
- slow: 0.80

Fast group's early speed is **55x the moderate group**. After standardisation, fast becomes an extreme outlier and moderate + slow are squashed together. Under missing data noise, the fast/non-fast boundary becomes blurry.


#### How we corrected it

We **drop the two problematic features** and **log-transform the skewed one**:

```python
# Original 8 features (Blooper 1)
feature_cols = [..., 'recovery_slope', ..., 'data_completeness']

# Improved 6 features (Phase A)
features['early_speed_log'] = np.log1p(features['early_speed'])
feature_cols_v2 = ['early_speed_log', 'mid_plateau', 'late_level',
                   'max_steps', 'variability', 'recovery_consistency']
```

After this fix, training accuracy improved slightly (~93% → ~94%) but **missing-data robustness improved dramatically** (e.g. at 75% missingness, MAR jumped from 79% to 92%). Removing two "bad" features made the model not just more accurate but also more robust to distribution shift.

### Blooper 2: Data Leakage Demonstration

#### What we tested 

We test whether using the **same cohort for training and testing** would inflate accuracy through data leakage. We train on `1-1_0_complete.csv` (Cohort B) and test on the missing-data versions of Cohort B (which contain the same patients, just with data removed).


<img width="985" alt="Blooper 2 setup" src="https://github.com/user-attachments/assets/5f99d32e-508b-4c06-becf-243d97b044ee" />

#### What we found

Surprisingly, the **leakage advantage is essentially zero** (mean +0.1%, range -0.7% to +1.3%, std 0.6%). The model trained on Cohort B and the model trained on Cohort C produce nearly identical results on the same test data.

<img width="203" alt="Blooper 2 leakage results" src="https://github.com/user-attachments/assets/69965013-0ae2-4e63-84e2-4587b416ec02" />

#### How we interpret it 

We find that K-means with summary features does not memorise individual patients — it learns group-level cluster boundaries that are reproducible across cohorts. Additionally, missing data destroys patient-level fingerprints: even if the model "remembered" patient A, their 99%-missing feature vector looks like a different person.

This null result confirms that **our cross-cohort design (in the main analysis) is methodologically defensible** without overpaying for it.
---

## Final Findings 

### Performance on Complete Data

We develop our final 6-feature model (Phase A) and find:

| Metric | Phase A (6 features) | Blooper 1 (8 features) |
|---|---|---|
| Training accuracy (Cohort C) | ~94% | ~93% |
| External validation (Cohort B) | ~95% | ~93% |
| ARI (training) | 0.83 | 0.72 |
| ARI (external) | 0.85 | 0.77 |

<img width="536" alt="Phase A vs Blooper 1 comparison" src="https://github.com/user-attachments/assets/e6466a45-29a6-4d7a-8bcb-86b7919b8003" />

### Missing Data Sensitivity 

We find that Phase A is remarkably robust to missing data:

| Missingness | MAR | MCAR | MNAR |
|---|---|---|---|
| 10% | 96.3% | 96.0% | 97.0% |
| 25% | 96.3% | 96.0% | 96.3% |
| 50% | 94.7% | 94.0% | 95.7% |
| 75% | 92.3% | 87.7% | 95.0% |
| 90% | **79.3%** | 91.3% | 92.3% |
| 95% | **75.7%** | 90.3% | 92.7% |
| 99% | **75.3%** | 91.0% | 91.3% |

**Key findings:**

1. **MCAR and MNAR remain >87% accurate at all tested missingness levels (10–99%).** The model does not "collapse" under random or low-activity-biased missingness.

2. **MAR is the only mechanism that causes significant degradation.** Specifically, MAR accuracy drops sharply between 75% and 90% missingness (from 92% to 79%), and continues to degrade at 95%/99% (75%). MAR removes early-recovery days, which destroy the most diagnostic feature (`early_speed_log`).

3. **At 99% missingness with MAR, the model becomes unreliable** (~75% accuracy, but with substantially noisier ARI). For practical clinical use, **MAR is the missingness pattern to watch out for, especially during the first 60 days post-treatment**.

4. **There is no clean "collapse point"** within the tested range. To find one, you would need to test at >99% (where each patient has fewer than 4 data points), or to combine multiple stressors (e.g. MAR + noise).

<!-- <img width="958" alt="Phase A degradation curves" src="https://github.com/user-attachments/assets/2798e5fb-0444-4d50-beff-5631ef406b19" /> -->
<img width="958" alt="Phase A degradation curves" src="https://raw.githubusercontent.com/Foluwa/HRfH-2026-Team-4-Task-2/main/images/kmeans_recovery_image.png" />

<img width="762" alt="Phase A heatmap" src="https://github.com/user-attachments/assets/e3353e25-67e4-4d59-95a5-96dc8b1f48de" />

### Sensitivity Analysis: Why Is the Model So Stable?

We design three perturbation experiments to identify what makes the main model robust.

#### Experiment A: Using only `early_speed_log` (1 feature)

With only 1 sensitive feature:

- Baseline accuracy drops from 95% to 60–70%
- Under MAR at 99% missing, accuracy collapses to **~30–40%** (near random)

This proves that **the main model's robustness comes from feature redundancy** — multiple features compensating for each other when individual features fail.

#### Experiment B: Fragile features only (drop `max_steps`, `mid_plateau`, `late_level`)

Removing the three statistical "anchor" features:

- Baseline accuracy drops by 10–15%
- Missing-data degradation curves become much steeper across all mechanisms
- MCAR at 75% drops from 88% (main) to ~55–65%

This proves that **`max_steps`, `mid_plateau`, and `late_level` are the primary drivers of robustness**. They are statistical summaries over long windows, so they survive even severe random subsampling.

#### Experiment C: Noisy data (add Gaussian noise std=500)

Adding measurement noise:

- Baseline accuracy drops from 95% to ~80–88%
- Missingness curves are shifted down but still relatively stable
- The combination of noise + extreme missingness causes ~30–40% accuracy

This proves that **the model's apparent high accuracy depends partly on the simulator's clean data**. Real-world wearable data with measurement noise would likely produce 80–88% baseline accuracy with steeper degradation.

#### Which features matter most

We rank the features by their **contribution to robustness**:

| Rank | Feature | Why it matters |
|---|---|---|
| 1 | `max_steps` | Requires only 1 peak day to be observed → highly missingness-resistant |
| 2 | `mid_plateau` | Mean of 120 days → stable under MCAR/MNAR even at 75% missing |
| 3 | `late_level` | Mean of 185 days → similar to mid_plateau |
| 4 | `recovery_consistency` | Sigmoid correlation → degrades under all mechanisms but slowly |
| 5 | `variability` | Standard deviation → noisier estimates under missingness |
| 6 | `early_speed_log` | Slope of first 60 days → most diagnostic in complete data, but most fragile under MAR |

---

## Summary

We develop a K-means clustering pipeline with 6 engineered features for identifying post-treatment recovery trajectory groups. Our main result (Phase A) achieves ~95% accuracy on complete external validation data and remains >87% accurate under most missingness scenarios up to 99%, with the notable exception of MAR mechanism at high missingness (drops to ~75% at 90%+ MAR). We find that this robustness is genuine but contingent on (1) using statistical summary features over long windows (`max_steps`, `mid_plateau`, `late_level`), (2) the clean nature of simulated data, and (3) the well-separated cluster structure produced by the simulator. Our diagnostic methodology — checking post-standardisation std, group monotonicity, and feature skewness — caught three feature problems in our initial 8-feature attempt (Blooper 1) and improved both baseline accuracy and missingness robustness when corrected. Our cross-cohort design was empirically validated through a leakage demonstration (Blooper 2), which showed negligible inflation from same-cohort training, confirming that K-means with summary features does not memorise individual patients. For clinical translation, we recommend treating the reported >95% accuracy as an optimistic upper bound and budgeting for ~80–88% baseline in realistic noisy settings.

---

## How to Run 

### Requirements
- Python 3.10+
- `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `joblib`
- Jupyter Notebook environment

The first cell of each notebook installs missing packages automatically.

### Recommended Order

1. **Start with `Hackathon_task_2.ipynb`** — main analysis. Run cells top to bottom (`Kernel → Restart & Run All`).
2. Run `BLooper1.ipynb` to inspect the initial 8-feature attempt in isolation.
3. Run `BLooper2.ipynb` to verify the data-leakage finding.
4. Run `Sensitivity_Experiments.ipynb` to see how robustness depends on feature design and noise.

| Notebook | Output files |
|---|---|
| `Hackathon_task_2.ipynb` | `kmeans_model.pkl`, `scaler.pkl`, `kmeans_model_v2.pkl`, `scaler_v2.pkl`, `phase3_results.csv`, `phase3_results_v2.csv`, `figure_1_degradation_curves.png`, `figure_2_heatmap.png`, `figure_3_99pct_comparison.png` |
| `BLooper1.ipynb` | `kmeans_model_blooper1.pkl`, `scaler_blooper1.pkl`, `phase3_results_blooper1.csv` |
| `BLooper2.ipynb` | `kmeans_model_blooper.pkl`, `scaler_blooper.pkl`, `phase3_results_blooper.csv` |
| `Sensitivity_Experiments.ipynb` | In-memory only (no .pkl saved) |

All output files are independently named so notebooks do not overwrite each other.

---

## Team

Fantastic 4 (Group 4), HRfH 2026 Hackathon, University of Manchester.

## Contact

For questions, contact the organisers via `hrfh@manchester.ac.uk`.
