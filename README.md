# HRfH-2026-Team-4-Task-2- Fantastic 4 (Group 4)

## The Task | 任務

We were asked to develop an analysis approach that uses daily step counts to identify post-treatment recovery trajectory groups (fast, moderate, slow), and to assess how recovery group identification changes under different missingness settings.

an online simulator (Shiny) [https://hrfh-hackathon-2026.shinyapps.io/hackathon2026/] that generates 365-day daily step count trajectories with configurable missingness mechanisms and severities was provided. We were given no underlying simulator code, only the output CSV files.

The four guiding questions from the brief:

1. How well can the recovery groups be identified when there is no missingness or very little missingness?
2. How does performance change as the amount of missingness increases?
3. Does performance differ when missingness occurs in different ways (MCAR, MAR, MNAR)?
4. What level or type of missingness appears to make recovery group identification unreliable?


我們的任務是發展一個分析方法，用每日步數識別治療後的恢復軌跡分組（fast 快、moderate 中、slow 慢），並評估在不同缺失資料情境下，組別識別的效能如何變化。

主辦方提供了一個 Shiny 線上模擬器 [https://hrfh-hackathon-2026.shinyapps.io/hackathon2026/]，產生 365 天的每日步數軌跡，可以設定不同的缺失機制與嚴重程度。我們只有 CSV 輸出檔，沒有模擬器原始碼。

要回答的四個問題：

1. 在沒有缺失或很少缺失的情況下，恢復組別可以被識別多準確？
2. 隨著缺失增加，效能怎麼變化？
3. 不同的缺失機制（MCAR、MAR、MNAR）對效能影響不同嗎？
4. 什麼程度或類型的缺失會讓組別識別變得不可靠？

---

## Methods | 方法

### Dataset Setup | 資料集設計

 We develop a three-phase validation design using **23 datasets** generated from the simulator:

- **2 complete datasets** (0% missingness) for training(seed=50) and external validation(seed=42), generated with different random seeds to create independent cohorts
- **21 missing-data datasets**(seed=42) = 7 missingness levels × 3 mechanisms

The 7 missingness levels are: 10%, 25%, 50%, 75%, 90%, 95%, 99%. The 3 mechanisms are:

| Mechanism | Behaviour | Realistic interpretation |
|---|---|---|
| **MCAR** (Missing Completely At Random) | Random sensor failures, with `gap_length=50` | Device battery dying, wearer removing the device for grooming |
| **MAR** (Missing At Random) | Concentrated in early recovery (`early_effect=70`, `BMI/age=50`) | Post-operative compliance is worse in the first weeks |
| **MNAR** (Missing Not At Random) | Biased toward low-activity days (`sensitivity=70`, `time_window=30 days`) | Sedentary patients are less likely to wear the device |

We hold all other simulator parameters constant (seed within each cohort, 365 days, 300 patients per cohort, age=68, BMI=28) so that we are measuring the effect of missingness alone.

 我們發展了一個**三階段驗證設計**，使用模擬器產生的 **23 個資料集**：

- **2 個完整資料集**（0% 缺失）用於訓練(seed=50) 與外部驗證(seed=42)，用不同 random seed 產生 → 兩個獨立 cohort
- **21 個缺失資料集** (seed=42)= 7 個缺失水平 × 3 個機制

7 個缺失水平：10%, 25%, 50%, 75%, 90%, 95%, 99%。3 個機制如上表。

我們**固定其他所有參數**（每個 cohort 內的 seed、365 天、300 人、年齡 68、BMI 28），只變動缺失設定，這樣才能單獨測量缺失的影響。
| Done                       | Dataset | File Name      | Missingness Level | Mechanism | Gap Length | Early Effect | BMI/Age | Low-Activity Sensitivity | Time Window |
| -------------------------- | ------- | -------------- | ----------------- | --------- | ---------- | ------------ | ------- | ------------------------ | ----------- |
| ☐seeding =42               | 1-1     | 1-1_0_complete | 0%                | —         | —          | —            | —       | —                        | —           |
| ☐seeding =50; for Training | 1-2     | 1-2_0_complete | 0%                |           |            |              |         |                          |             |
| ☐seeding =42               | 2       | 2_10MCAR       | 10%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐                          | 3       | 3_10MAR        | 10%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐                          | 4       | 4_10MNAR       | 10%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐                          | 5       | 5_25MCAR       | 25%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐                          | 6       | 6_25MAR        | 25%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐                          | 7       | 7_25MNAR       | 25%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐                          | 8       | 8_50MCAR       | 50%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐                          | 9       | 9_50MAR        | 50%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐                          | 10      | 10_50MNAR      | 50%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐                          | 11      | 11_75MCAR      | 75%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐                          | 12      | 12_75MAR       | 75%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐                          | 13      | 13_75MNAR      | 75%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐                          | 14      | 14_90MCAR      | 90%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐                          | 15      | 15_90MAR       | 90%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐                          | 16      | 16_90MNAR      | 90%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐                          | 17      | 17_95MCAR      | 95%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐                          | 18      | 18_95MAR       | 95%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐                          | 19      | 19_95MNAR      | 95%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐                          | 20      | 20_99MCAR      | 99%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐                          | 21      | 21_99MAR       | 99%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐                          | 22      | 22_99MNAR      | 99%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| —                          | —       | (100% = N/A)   | 100%              | any       | —          | —            | —       | —                        | —           |
### Clustering Method | 分群方法
** We use **K-means clustering** with K=3 (matching the simulator's 3 recovery groups), `random_state=42` for reproducibility, and `n_init=10` to mitigate sensitivity to initialisation.

Why K-means?

- **Interpretable**: cluster centroids correspond to "typical" patients of each group
- **Reproducible**: deterministic given seed
- **Computationally cheap**: ~300 participants × 6 features finishes in milliseconds
- **Standard**: well-understood and accepted by clinical reviewers

Since K-means is unsupervised, we use the **Hungarian-style label permutation** approach: K-means produces labels 0/1/2 with no inherent meaning, so we try all 3! = 6 possible mappings to the true labels (fast/moderate/slow) and select the mapping that maximises accuracy. This is standard practice for evaluating unsupervised clustering against known ground truth.

**Feature engineering**: each participant's 365-day trajectory is compressed into **6 summary features**:

| Feature                | Computation                    | Why                                                      |
| ---------------------- | ------------------------------ | -------------------------------------------------------- |
| `early_speed_log`      | `np.log1p(slope of days 1–60)` | How fast they start recovering (log to control skewness) |
| `mid_plateau`          | Mean steps on days 61–180      | Stabilisation level                                      |
| `late_level`           | Mean steps on days 181–365     | Final plateau                                            |
| `max_steps`            | Peak daily value               | Activity ceiling                                         |
| `variability`          | Day-to-day standard deviation  | Consistency                                              |
| `recovery_consistency` | Correlation with ideal sigmoid | Trajectory shape quality                                 |

After standardisation (StandardScaler fitted on training only), K-means is trained on the resulting 300×6 matrix.

 我們用 **K-means 分群**，K=3（對應 fast/moderate/slow 3 組），`random_state=42` 確保可重現，`n_init=10` 減少初始化敏感性。

為什麼選 K-means？

- **可解釋**：cluster centroids 對應「典型病人」
- **可重現**：給定 seed 是確定性的
- **計算便宜**：300 人 × 6 特徵幾毫秒
- **標準方法**：臨床審稿者熟悉

因為 K-means 是非監督的，cluster 標籤 0/1/2 沒有先天意義，所以我們試 3! = 6 種可能對應到真實標籤（fast/moderate/slow），選最大化 accuracy 的那個 permutation。

**特徵工程**：每個病人的 365 天軌跡壓縮成 **6 個摘要特徵**（如上表）。
<img width="1590" height="801" alt="image" src="https://github.com/user-attachments/assets/05526507-11df-4c4a-b49f-bbb378f40423" />

### Testing Methodology | 驗證方法
 We use a **strict cross-cohort design** to avoid data leakage:

| Phase | Dataset | Purpose | Cohort |
|---|---|---|---|
| Phase 1 | `1-2_0_complete.csv` | Train K-means and lock model | Cohort C (seed 50) |
| Phase 2 | `1-1_0_complete.csv` | External validation on complete data | Cohort B (seed 45) |
| Phase 3 | 21 missing-data CSV files | Sensitivity to missingness | Cohort B with missingness applied |

Critical: the **training cohort (C) is different from the testing cohort (B)**. The Phase 3 missing-data files are derived from Cohort B, so they share the same underlying patients as Phase 2 (just with data removed), but never appear in the training data.




We **lock the model after training** using `joblib.dump()`. All subsequent uses (Phases 2 and 3) use `joblib.load()` followed by `scaler.transform()` and `kmeans.predict()` — never `fit_transform()` or `fit_predict()`. This guarantees the model never sees test data during training.

 我們用**嚴格的跨 cohort 設計**避免 data leakage：

| Phase | Dataset | 目的 | Cohort |
|---|---|---|---|
| Phase 1 | `1-2_0_complete.csv` | 訓練 + lock 模型 | Cohort C (seed 50) |
| Phase 2 | `1-1_0_complete.csv` | 完整資料外部驗證 | Cohort B (seed 45) |
| Phase 3 | 21 個缺失資料檔 | Missing data 敏感性 | Cohort B 加缺失機制 |

**關鍵：訓練 cohort (C) 跟測試 cohort (B) 不同**。Phase 3 的缺失資料雖然是從 Cohort B 派生的，但跟訓練資料沒接觸過。

我們在訓練後**用 `joblib.dump()` lock 模型**，之後 Phase 2 和 Phase 3 都用 `joblib.load()` + `transform()` + `predict()`，**絕不再 `fit`**，確保模型不會偷偷再學測試資料。
<img width="467" height="65" alt="image" src="https://github.com/user-attachments/assets/6d375c9f-84c6-46d7-acd8-24fe1f1c254f" />

<img width="472" height="63" alt="image" src="https://github.com/user-attachments/assets/8d140111-0c71-42ce-9819-1e270658c996" />
<img width="1200" height="1600" alt="WhatsApp Image 2026-05-15 at 4 33 30 AM" src="https://github.com/user-attachments/assets/df893d31-a3d5-4c41-9bb3-feaf475e1f82" />


### Sensitivity Analysis | 敏感性分析
To verify that our main result's robustness reflects real model quality rather than artifacts, we develop three perturbation experiments in `Sensitivity_Experiments.ipynb`:

| Experiment | Modification | Purpose |
|---|---|---|
| **A: Sensitive features only** | Use only `early_speed_log` (1 feature instead of 6) | Test whether robustness depends on feature redundancy |
| **B: Fragile features only** | Drop `max_steps`, `mid_plateau`, `late_level` (the most robust features); keep `early_speed_log`, `variability`, `recovery_consistency` | Test which specific features are responsible for robustness |
| **C: Noisy data** | Add Gaussian noise (std=500 steps) to every step count | Test how robustness changes under realistic sensor noise |

Each experiment is run through the full pipeline (training, external validation, missing-data tests) and the degradation curves are compared against the main Phase A model.

為了驗證主結果的穩健性是真的，不是 artifact，我們在 `Sensitivity_Experiments.ipynb` 設計三個 perturbation 實驗（如上表）。

每個實驗都跑完整流程（訓練、外部驗證、缺失資料測試），並把降解曲線跟主 Phase A 對比。
<img width="1488" height="413" alt="image" src="https://github.com/user-attachments/assets/3675e55c-3c5b-4f25-9e4d-5052bd83f4be" />

---

## Results: The Bloopers | 結果：兩個 Bloopers

### Blooper 1: The 8-Feature Failure | 8-特徵失敗===>>>> NEED  TO FIX

#### What went wrong | 問題

 We started with **8 features** chosen by clinical domain knowledge:

```
['early_speed', 'mid_plateau', 'late_level', 'recovery_slope',
 'max_steps', 'variability', 'recovery_consistency', 'data_completeness']
```

We trained K-means on these 8 features and **achieved only 58% accuracy on training data** — suspiciously lower than the 64% on external validation. Normally training accuracy should be the highest because the model has "seen" that data.

我們最初用 domain knowledge 選了 **8 個特徵**（如上）。

訓練 K-means 之後**只達到 58% accuracy 在訓練資料上** —— 比外部驗證的 64% 還低，這非常異常。通常訓練 accuracy 應該最高，因為模型「看過」那些資料。


#### How we detected it | 怎麼發現

**English:** We develop a systematic diagnostic check on the features:

**Check 1: Post-standardisation standard deviation**

```python
print(X_scaled.std(axis=0))
# Output: [1. 1. 1. 1. 1. 1. 1. 0.]
#                                 ↑
#                          data_completeness std = 0!
```

`data_completeness` is constant=1.0 in complete training data, so its std after scaling is 0. A feature with std=0 contributes only noise during clustering.

**Check 2: Feature monotonicity across true groups**

```python
for feature in feature_cols:
    means = features.groupby('true_group')[feature].mean()
    print(f"{feature}: fast={means['fast']}, moderate={means['moderate']}, slow={means['slow']}")
```

We find that `recovery_slope` is **non-monotonic**:
- fast: 12.10
- moderate: **14.73** ← higher than fast!
- slow: 7.53

Moderate patients have higher recovery slope than fast patients, which is biologically implausible and actively misleads K-means.

**Check 3: Feature distribution skewness**

We find that `early_speed` is severely skewed:
- fast: 95.19
- moderate: 1.73
- slow: 0.80

Fast group's early speed is **55x the moderate group**. After standardisation, fast becomes an extreme outlier and moderate + slow are squashed together.

 我們發展了一套**系統性 diagnostic check** 來檢驗特徵：

**Check 1：標準化後的標準差**

```
Output: Std = [1. 1. 1. 1. 1. 1. 1. 0.]
                                    ↑
                              data_completeness 的 std=0!
```

完整訓練資料中 `data_completeness` 都是 1.0，標準化後 std=0。**標準差 = 0 的特徵在分群中只貢獻雜訊**。

**Check 2：特徵在 true groups 間的單調性**

`recovery_slope` 邏輯顛倒：
- fast: 12.10
- moderate: **14.73** ← 比 fast 還高！
- slow: 7.53

Moderate 比 fast 恢復斜率高，**生物學上不合理**，且會主動誤導 K-means。

**Check 3：特徵分佈偏度**

`early_speed` 極端偏斜：
- fast: 95.19, moderate: 1.73, slow: 0.80

Fast 組是 moderate 的 **55 倍**。標準化後 fast 變極端 outlier，把 moderate + slow 擠在一起。

#### How we corrected it | 怎麼修正

**English:** We **drop the two problematic features** and **log-transform the skewed one**:

```python
# Original 8 features (Blooper 1)
feature_cols = [..., 'recovery_slope', ..., 'data_completeness']

# Improved 6 features (Phase A)
features['early_speed_log'] = np.log1p(features['early_speed'])
feature_cols_v2 = ['early_speed_log', 'mid_plateau', 'late_level',
                   'max_steps', 'variability', 'recovery_consistency']
```

After this fix, training accuracy jumps from 58% to ~94%. Even more remarkably, the missing-data robustness also improves (see Final Findings below).

 我們**移除兩個問題特徵**，並對偏斜特徵做 **log 轉換**（如上程式碼）。

修正後，訓練 accuracy 從 58% 跳到 ~94%。更值得注意的是，**missing data 的穩健性也大幅改善**（見最終發現）。

### Blooper 2: Data Leakage Demonstration | Data Leakage 示範

#### What we tested | 我們測什麼

 We test whether using the **same cohort for training and testing** would inflate accuracy through data leakage. We train on `1-1_0_complete.csv` (Cohort B) and test on the missing-data versions of Cohort B (which contain the same patients, just with data removed).

 我們測試**訓練跟測試用同 cohort** 會不會因 data leakage 而虛胖 accuracy。訓練在 `1-1_0_complete.csv` (Cohort B)，測試在 Cohort B 的缺失版本（同樣 300 病人，只是部分資料被刪除）。
<img width="985" height="127" alt="image" src="https://github.com/user-attachments/assets/5f99d32e-508b-4c06-becf-243d97b044ee" />

#### What we found | 我們發現

 Surprisingly, the **leakage advantage is essentially zero** (mean +0.1%, range -0.7% to +1.3%, std 0.6%). The model trained on Cohort B and the model trained on Cohort C produce nearly identical results on the same test data.

出乎意料，**leakage 優勢幾乎是零**（平均 +0.1%，範圍 -0.7% 到 +1.3%，標準差 0.6%）。訓練在 Cohort B 跟訓練在 Cohort C 的模型在同樣測試資料上幾乎一樣。
<img width="203" height="421" alt="image" src="https://github.com/user-attachments/assets/69965013-0ae2-4e63-84e2-4587b416ec02" />


#### How we interpret it | 解讀

We find that K-means with summary features does not memorise individual patients — it learns group-level cluster boundaries that are reproducible across cohorts. Additionally, missing data destroys patient-level fingerprints: even if the model "remembered" patient A, their 99%-missing feature vector looks like a different person.

This null result confirms that **our cross-cohort design (in the main analysis) is methodologically defensible** without overpaying for it. It is a sanity check, not a smoking gun.

 我們發現 K-means + 統計摘要特徵**不會記住個別病人**，學的是「組別之間的邊界」，這個邊界在不同 cohort 間是可重現的。而且缺失資料破壞了病人的「身分證」：即使模型記得病人 A，他 99% 缺失的特徵向量看起來像不同人。

這個 null result 確認**主分析的跨 cohort 設計在方法學上是站得住腳的**，而且沒「過度設計」。

---

## Final Findings | 最終發現

### Performance on Complete Data | 完整資料表現

 We develop our final 6-feature model (Phase A) and find:

| Metric | Phase A (6 features) | Blooper 1 (8 features) |
|---|---|---|
| Training accuracy (Cohort C) | ~94% | ~92% |
| External validation (Cohort B) | ~95% | ~93% |
| ARI (training) | 0.83 | 0.72 |
| ARI (external) | 0.85 | 0.77 |

 我們發展的最終 6-feature 模型（Phase A）：

| 指標 | Phase A (6 features) | Blooper 1 (8 features) |
|---|---|---|
| 訓練 accuracy (Cohort C) | ~94% | ~92% |
| 外部驗證 (Cohort B) | ~95% | ~93% |
| ARI (訓練) | 0.83 | 0.72 |
| ARI (外部) | 0.85 | 0.77 |

<img width="536" height="326" alt="image" src="https://github.com/user-attachments/assets/e6466a45-29a6-4d7a-8bcb-86b7919b8003" />


### Missing Data Sensitivity | 缺失資料敏感性

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

我們發現 Phase A 對缺失資料極度穩健（如上表）。

**主要發現：**

1. **MCAR 和 MNAR 在所有測試的缺失水平 (10-99%) 都 >87%**。模型不會在隨機或低活動偏向的缺失下「崩潰」。

2. **MAR 是唯一造成顯著退化的機制**。MAR accuracy 在 75%-90% 之間急遽下降（92% → 79%），95%/99% 繼續退化到 75%。**MAR 移除早期日，破壞了診斷力最強的特徵 `early_speed_log`**。

3. **在 99% 缺失且 MAR 機制下，模型變得不可靠**（~75% accuracy，ARI 變動很大）。臨床應用上，**MAR 是要特別警覺的缺失模式，尤其是治療後前 60 天**。

4. 在測試範圍內**沒有明確的「崩潰點」**。要找的話需要測 >99%（每人少於 4 個資料點）或結合多個壓力源（如 MAR + noise）。

### Sensitivity Analysis: Why Is the Model So Stable? | 為什麼模型這麼穩定？

We design three perturbation experiments to identify what makes the main model robust.

#### Experiment A: Using only `early_speed_log` (1 feature)

We find that with only 1 sensitive feature:
- Baseline accuracy drops from 95% to 60–70%
- Under MAR at 99% missing, accuracy collapses to **~30–40%** (near random)

This proves that **the main model's robustness comes from feature redundancy** — multiple features compensating for each other when individual features fail.

#### Experiment B: Fragile features only (drop `max_steps`, `mid_plateau`, `late_level`)

We find that removing the three statistical "anchor" features:
- Baseline accuracy drops by 10–15%
- Missing-data degradation curves become much steeper across all mechanisms
- MCAR at 75% drops from 88% (main) to ~55–65%

This proves that **`max_steps`, `mid_plateau`, and `late_level` are the primary drivers of robustness**. They are statistical summaries over long windows, so they survive even severe random subsampling.

#### Experiment C: Noisy data (add Gaussian noise std=500)

We find that adding measurement noise:
- Baseline accuracy drops from 95% to ~80–88%
- Missingness curves are shifted down but still relatively stable
- The combination of noise + extreme missingness causes ~30–40% accuracy

This proves that **the model's apparent high accuracy depends partly on the simulator's clean data**. Real-world wearable data with measurement noise would likely produce 80–88% baseline accuracy with steeper degradation.

#### Which features matter most | 哪個特徵最重要

We rank the features by their **contribution to robustness**:

| Rank | Feature | Why it matters |
|---|---|---|
| 1 | `max_steps` | Requires only 1 peak day to be observed → highly missingness-resistant |
| 2 | `mid_plateau` | Mean of 120 days → stable under MCAR/MNAR even at 75% missing |
| 3 | `late_level` | Mean of 185 days → similar to mid_plateau |
| 4 | `recovery_consistency` | Sigmoid correlation → degrades under all mechanisms but slowly |
| 5 | `variability` | Standard deviation → noisier estimates under missingness |
| 6 | `early_speed_log` | Slope of first 60 days → most diagnostic in complete data, but most fragile under MAR |

我們設計三個 perturbation 實驗找出模型穩定性的來源。

**Experiment A**：只用 `early_speed_log`（1 個特徵）→ MAR 99% 缺失時 accuracy 崩到 30-40%（接近隨機）。
> 證明**主模型的穩健性來自特徵冗餘** —— 多個特徵互相補足。

**Experiment B**：移除穩健特徵 → 所有機制的退化曲線都變陡，MCAR 75% 從 88% 掉到 55-65%。
> 證明 **`max_steps`, `mid_plateau`, `late_level` 是穩健性的主要來源**。它們是長時間窗口的統計摘要，能在嚴重 subsampling 下存活。

**Experiment C**：加 Gaussian noise → baseline 從 95% 掉到 80-88%。
> 證明**模型的高 accuracy 部分依賴模擬器資料的乾淨**。真實 wearable 資料可能 baseline 只有 80-88%。

**哪個特徵最重要？**

我們排序特徵對穩健性的貢獻（如上表）：
1. `max_steps`（最穩，1 天高峰就夠）
2. `mid_plateau`（120 天平均）
3. `late_level`（185 天平均）
4. `recovery_consistency`
5. `variability`
6. `early_speed_log`（最有診斷力但最脆弱）

---

## Summary  | 一段話總結

 We develop a K-means clustering pipeline with 6 engineered features for identifying post-treatment recovery trajectory groups. Our main result (Phase A) achieves ~95% accuracy on complete external validation data and remains >87% accurate under most missingness scenarios up to 99%, with the notable exception of MAR mechanism at high missingness (drops to ~75% at 90%+ MAR). We find that this robustness is genuine but contingent on (1) using statistical summary features over long windows (`max_steps`, `mid_plateau`, `late_level`), (2) the clean nature of simulated data, and (3) the well-separated cluster structure produced by the simulator. Our diagnostic methodology — checking post-standardisation std, group monotonicity, and feature skewness — caught three feature problems in our initial 8-feature attempt (Blooper 1) and improved both baseline accuracy and missingness robustness when corrected. Our cross-cohort design was empirically validated through a leakage demonstration (Blooper 2), which showed negligible inflation from same-cohort training, confirming that K-means with summary features does not memorise individual patients. For clinical translation, we recommend treating the reported >95% accuracy as an optimistic upper bound and budgeting for ~80–88% baseline in realistic noisy settings.

我們發展了一個 K-means 分群 pipeline，用 6 個工程特徵來識別治療後恢復軌跡的組別。主結果（Phase A）在完整外部驗證資料上達到 ~95% accuracy，且在 99% 以下的大部分缺失情境下都維持 >87%，唯一例外是 MAR 在高缺失（90%+）會掉到 ~75%。我們發現這個穩健性是真實的，但依賴於 (1) 使用長時間窗口的統計摘要特徵（`max_steps`, `mid_plateau`, `late_level`）、(2) 模擬器資料的乾淨、(3) 模擬器產生的良好分離 cluster 結構。我們的 diagnostic methodology（檢查標準化後的 std、組間單調性、特徵偏度）在初始的 8-feature 嘗試（Blooper 1）發現了三個特徵問題，修正後不只 baseline accuracy 改善，缺失穩健性也改善了。我們的跨 cohort 設計透過 leakage 示範（Blooper 2）獲得實證驗證：同 cohort 訓練的虛胖效應幾乎為零，確認 K-means + 摘要特徵不會記住個別病人。臨床應用上我們建議把報告的 >95% accuracy 當作樂觀的上界，在真實雜訊環境下預期 ~80-88% baseline。



