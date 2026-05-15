# Phase 3: 13 Missing Data Datasets + 2 Complete Training/Testing Sets

## Design Rationale

This three-phase validation strategy uses **15 datasets total**:

**Phase 1: Train and Lock Model**
- **Dataset 1 (Training):** `full_recovery_clinical_curves_2026-05-14_complete.csv`
  - 300 participants, 365 days, 0% missing
  - Used to train K-means model and standardise features
  - Accuracy: 58.3%, ARI: 0.352

**Phase 2: External Validation**
- **Dataset 2 (Testing):** `(2)recovery_clinical_curves_2026-05-14_complete.csv`
  - 300 participants (different cohort), 365 days, 0% missing
  - Tests model generalisation on new complete data
  - Accuracy: 64.0%, ARI: 0.413

**Phase 3: Missing Data Sensitivity Analysis**
- **Datasets 3-15 (13 missing data variations):**
  - Assess how clustering performance degrades under different missingness scenarios
  - Design: 4 missingness levels (10%, 25%, 50%, 75%) × 3 mechanisms (MCAR, MAR, MNAR) = 12 datasets
  - Plus 1 additional complete dataset (0%) as Phase 3 baseline
  - Total: 13 Phase 3 datasets

### Why This Design?

1. **Missingness Levels:** 10%, 25%, 50%, 75% cover realistic clinical scenarios
   - 10%: mild data collection issues
   - 25%: moderate compliance problems
   - 50%: severe tracking failures
   - 75%: critical data scarcity

2. **Three Mechanisms:** Each represents different missing data patterns
   - **MCAR** (Missing Completely At Random): Random sensor failures, unrelated to activity
   - **MAR** (Missing At Random): Concentration in early recovery phase (realistic for post-op compliance)
   - **MNAR** (Missing Not At Random): Systematic bias toward missing low-activity days (most harmful)

3. **Constant Parameters:** Isolate effect of missingness level alone
   - Seed, days, cohorts, age, BMI, and mechanism parameters held constant
   - Only missing data level varies within each mechanism type
   - Ensures clean comparison of accuracy degradation patterns

---

## Full Summary Table

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

> **Note on 100% missing:** Not generated as a dataset. With no data, feature extraction is impossible — the model has no information to cluster on. Theoretical accuracy = random baseline = 1/3 = **33.3%**. Reported as a reference point in the degradation curve, not as an actual evaluation.

^i82vk2


---

## Grouped by Mechanism (Constant Parameters)

### Complete Dataset

| Done | Dataset | File Name    | Missingness |
| ---- | ------- | ------------ | ----------- |
| v    | 1       | 1_0_complete | 0%          |

### MCAR (Random Gaps) — Gap Length = 50

| Done | Dataset | File Name | Missingness |
| ---- | ------- | --------- | ----------- |
| ☐    | 2       | 2_10MCAR  | 10%         |
| ☐    | 5       | 5_25MCAR  | 25%         |
| ☐    | 8       | 8_50MCAR  | 50%         |
| ☐    | 11      | 11_75MCAR | 75%         |
| ☐    | 14      | 14_90MCAR | 90%         |
| ☐    | 17      | 17_95MCAR | 95%         |
| ☐    | 20      | 20_99MCAR | 99%         |

### MAR (More Missing Early) — Early Effect = 70, BMI/Age = 50

| Done | Dataset | File Name | Missingness |
| ---- | ------- | --------- | ----------- |
| ☐    | 3       | 3_10MAR   | 10%         |
| ☐    | 6       | 6_25MAR   | 25%         |
| ☐    | 9       | 9_50MAR   | 50%         |
| ☐    | 12      | 12_75MAR  | 75%         |
| ☐    | 15      | 15_90MAR  | 90%         |
| ☐    | 18      | 18_95MAR  | 95%         |
| ☐    | 21      | 21_99MAR  | 99%         |

### MNAR (Low-Activity Biased) — Low-Activity Sensitivity = 70, Time Window = 30 days

| Done | Dataset | File Name  | Missingness |
| ---- | ------- | ---------- | ----------- |
| ☐    | 4       | 4_10MNAR   | 10%         |
| ☐    | 7       | 7_25MNAR   | 25%         |
| ☐    | 10      | 10_50MNAR  | 50%         |
| ☐    | 13      | 13_75MNAR  | 75%         |
| ☐    | 16      | 16_90MNAR  | 90%         |
| ☐    | 19      | 19_95MNAR  | 95%         |
| ☐    | 22      | 22_99MNAR  | 99%         |

---

## Detailed Configuration

### Complete Dataset

**Dataset 1: 0% Missing (Baseline)**
- Missingness Level: 0%
- Mechanism: None
- Seed: 42, Days: 365, Cohorts: 300 each
- Age: 68, BMI: 28

### 10% Missing Datasets

**Dataset 2: 10% MCAR**
- Missingness Level: 10%
- Mechanism: Random gaps (MCAR)
- Gap length distribution: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 3: 10% MAR**
- Missingness Level: 10%
- Mechanism: More missing early in recovery (MAR)
- Early recovery effect: 70
- BMI and age effect: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 4: 10% MNAR**
- Missingness Level: 10%
- Mechanism: Low-activity days more likely missing (MNAR)
- Low-activity sensitivity: 70
- Time window: 30 days
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

### 25% Missing Datasets

**Dataset 5: 25% MCAR**
- Missingness Level: 25%
- Mechanism: Random gaps (MCAR)
- Gap length distribution: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 6: 25% MAR**
- Missingness Level: 25%
- Mechanism: More missing early in recovery (MAR)
- Early recovery effect: 70
- BMI and age effect: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 7: 25% MNAR**
- Missingness Level: 25%
- Mechanism: Low-activity days more likely missing (MNAR)
- Low-activity sensitivity: 70
- Time window: 30 days
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

### 50% Missing Datasets

**Dataset 8: 50% MCAR**
- Missingness Level: 50%
- Mechanism: Random gaps (MCAR)
- Gap length distribution: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 9: 50% MAR**
- Missingness Level: 50%
- Mechanism: More missing early in recovery (MAR)
- Early recovery effect: 70
- BMI and age effect: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 10: 50% MNAR**
- Missingness Level: 50%
- Mechanism: Low-activity days more likely missing (MNAR)
- Low-activity sensitivity: 70
- Time window: 30 days
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

### 75% Missing Datasets

**Dataset 11: 75% MCAR**
- Missingness Level: 75%
- Mechanism: Random gaps (MCAR)
- Gap length distribution: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 12: 75% MAR**
- Missingness Level: 75%
- Mechanism: More missing early in recovery (MAR)
- Early recovery effect: 70
- BMI and age effect: 50
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

**Dataset 13: 75% MNAR**
- Missingness Level: 75%
- Mechanism: Low-activity days more likely missing (MNAR)
- Low-activity sensitivity: 70
- Time window: 30 days
- Seed: 42, Days: 365, Cohorts: 300 each, Age: 68, BMI: 28

---

## Parameter Rationale

**MCAR (Gap length = 50)**
- Balanced between short and long gaps
- Represents truly random sensor failures

**MAR (Early effect = 70, BMI/Age = 50)**
- Strong concentration in early recovery phase (realistic for post-op compliance issues)
- Medium demographic effect (age and BMI naturally influence activity patterns)

**MNAR (Sensitivity = 70, Time window = 30 days)**
- Moderate bias towards missing low-activity days
- 30-day windows detect within-month patterns of missing data

---

## Generation Workflow

1. Go to Shiny simulator: https://hrfh-hackathon-2026.shinyapps.io/hackathon2026/
2. Set constant parameters: Seed=42, Days=365, Cohorts=300, Age=68, BMI=28
3. For each dataset, set the sliders according to this table
4. Download all 16 CSV files
5. Create folder: `simulator_outputs/`
6. Move all 16 CSV files into `simulator_outputs/`
7. Run Phase 3 evaluation loop in notebook

---

## Expected Results

| Missingness Level | MCAR | MAR | MNAR | Notes |
|---|---|---|---|---|
| 0% | Baseline | Baseline | Baseline | Complete data accuracy ~58-64% |
| 10% | ~55-58% | ~50-55% | ~45-50% | Slight degradation |
| 25% | ~45-50% | ~35-45% | ~25-35% | Noticeable drop |
| 50% | ~30-40% | ~20-30% | ~10-20% | Significant degradation |
| 75% | ~15-25% | ~10-15% | ~5-10% | Severe degradation, near random |

**Patterns:**
- Accuracy decreases with missingness level
- MNAR causes worse degradation than MAR
- MAR causes worse degradation than MCAR (at same level)
- At 75% missing, model becomes unreliable
