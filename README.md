# HRfH-2026-Team-4-Task-2
# RESULT
## Final Result
<img width="580" height="363" alt="image" src="https://github.com/user-attachments/assets/85f756de-8f85-4fdd-b2e9-5ed6955f0356" />

## Sensitivity Analysis
<img width="1488" height="413" alt="image" src="https://github.com/user-attachments/assets/c33599e6-ca77-4032-8312-034dced4659e" />

To validate that our main model\'s robustness is not an artifact of 
feature engineering or simulator cleanliness, we ran three perturbation 
experiments:

| Experiment | Modification | Mean accuracy drop at 75% missingness |
|---|---|---|
| Main (Phase A) | (control) | ~5% |
| A: Sensitive features only | 1 feature | ~XX% |
| B: Fragile features only | Drop max/mean | ~XX% |
| C: Noisy data | +500 step std | ~XX% |

These experiments show that the main model\'s robustness depends on 
(1) the breadth of summary features and (2) clean step count data. 
In noisy clinical settings, we expect degradation closer to 
Experiment C, suggesting a more cautious accuracy estimate of ~XX% 
under realistic missingness.

# METHOD
## Training DATA+ Validation DATA+ Missing DATA
## 13 Missing Data Datasets + 2 Complete Training/Testing Sets



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

| Done | Dataset | File Name    | Missingness Level | Mechanism | Gap Length | Early Effect | BMI/Age | Low-Activity Sensitivity | Time Window |
| ---- | ------- | ------------ | ----------------- | --------- | ---------- | ------------ | ------- | ------------------------ | ----------- |
| v    | 1       | 1_0_complete | 0%                | —         | —          | —            | —       | —                        | —           |
| ☐    | 2       | 2_10MCAR     | 10%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐    | 3       | 3_10MAR      | 10%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐    | 4       | 4_10MNAR     | 10%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐    | 5       | 5_25MCAR     | 25%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐    | 6       | 6_25MAR      | 25%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐    | 7       | 7_25MNAR     | 25%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐    | 8       | 8_50MCAR     | 50%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐    | 9       | 9_50MAR      | 50%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐    | 10      | 10_50MNAR    | 50%               | MNAR      | —          | —            | —       | 70                       | 30 days     |
| ☐    | 11      | 11_75MCAR    | 75%               | MCAR      | 50         | —            | —       | —                        | —           |
| ☐    | 12      | 12_75MAR     | 75%               | MAR       | —          | 70           | 50      | —                        | —           |
| ☐    | 13      | 13_75MNAR    | 75%               | MNAR      | —          | —            | —       | 70                       | 30 days     |


