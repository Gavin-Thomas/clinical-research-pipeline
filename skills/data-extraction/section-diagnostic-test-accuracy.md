# Data Extraction — diagnostic_test_accuracy

DTA reviews require fundamentally different extraction fields, quality tools, and synthesis methods.
Use this path when `project_type: diagnostic_test_accuracy`.

## Step 1: Dispatch Statistician for DTA Template Design

Same as the review-types path, but the template must include these DTA-specific fields in addition
to standard study identification:

**Accuracy data (from 2×2 contingency table):**
- `tp`, `fp`, `tn`, `fn` — raw cell counts. If the 2×2 table is not reported explicitly, reconstruct:
  TP = round(sensitivity × n_diseased); FP = round((1−specificity) × n_non_diseased).
  Document reconstruction in `source_notes`.
- `sensitivity`, `sensitivity_ci_lower`, `sensitivity_ci_upper` — as proportions (0–1), not percentages
- `specificity`, `specificity_ci_lower`, `specificity_ci_upper` — as proportions
- `threshold_used` — exact cut-off value applied in the study (critical for threshold effect analysis)

**Index test / reference standard fields:**
- `blinding_index` (Y/N/NR) — was the index test result interpreted blinded to the reference standard?
- `blinding_reference` (Y/N/NR) — was the reference standard applied blinded to the index test?
- `partial_verification` (Y/N/NR) — did all participants receive the reference standard? (N = verification bias)
- `recruitment` (consecutive | random | convenience) — drives spectrum bias assessment

**QUADAS-2 (7 domains, scored Low | Unclear | High for risk of bias):**
- `q2_patient_selection` — inappropriate exclusions avoided? consecutive/random sampling used?
- `q2_index_test` — index test interpreted blinded to reference standard?
- `q2_reference_standard` — reference standard classifies condition correctly? interpreted blinded?
- `q2_flow_timing` — appropriate interval between tests? all patients received same reference standard?
- `q2_app_patient_selection` — do patients match the review question population?
- `q2_app_index_test` — does the index test match the review question?
- `q2_app_reference_standard` — does the reference standard match the review question?

## Step 2: Partition Papers Across 3 Data Extractors

Same as review-types path. Add these items to the Data Extractor **SELF-CHECK**:
1. 2×2 table present (TP/FP/TN/FN) — or reconstruction method and formula documented in `source_notes`
2. All 7 QUADAS-2 domains scored (Low/Unclear/High) — no blank domains
3. Sensitivity and specificity recorded as proportions (0–1 scale), not percentages
4. `partial_verification` documented for every study

## Step 3: Merge Extracted Data

Combine all 3 extractors into `06_data_extraction/extracted_data.csv` (same as review-types).

## Step 4: Full Review with Retry Logic

Same as review-types path. **Add to review criteria:**
- Are TP/FP/TN/FN values internally consistent with reported sensitivity, specificity, and N?
  (Cross-check: sensitivity = TP/(TP+FN); specificity = TN/(TN+FP))
- Is QUADAS-2 complete (all 7 domains) for every study?
- Is partial verification documented?

## Step 5: Dispatch Statistician for DTA Synthesis

**Statistician agent prompt (DTA-specific):**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Synthesize diagnostic test accuracy data and generate executable R analysis code.

EXTRACTED DATA: [Read and insert 06_data_extraction/extracted_data.csv]
RESEARCH QUESTION (PICOS): [Read 02_research_question/research_question.md]

Produce:

1. STUDY CHARACTERISTICS TABLE: Study ID, country, setting, N, disease prevalence (%),
   index test threshold, reference standard, QUADAS-2 overall rating per domain.

2. QUADAS-2 TRAFFIC LIGHT TABLE: 7 domains × each study, colour-coded Low/Unclear/High.
   Summarise: proportion of studies at low risk per domain.

3. THRESHOLD EFFECT TEST: Calculate Spearman correlation between logit(sensitivity) and
   logit(1−specificity) across studies. If r > 0.6 and p < 0.05 → threshold effect likely
   → use HSROC model. Otherwise → use bivariate model.

4. META-ANALYSIS (if n_included ≥ 4):
   a. BIVARIATE MODEL (default): Fit Reitsma et al. using R mada::reitsma(). Report pooled
      sensitivity and specificity with 95% CIs and 95% confidence ellipse on SROC plane.
   b. HSROC CURVE (if threshold effect detected): Fit Rutter & Gatsonis using mada::hsroc().
      Overlay HSROC curve on SROC plane with individual study operating points.
   c. HETEROGENEITY: I² separately for sensitivity and specificity (logit-transformed DL estimates).
   d. PUBLICATION BIAS (n ≥ 10): Deeks' funnel plot (log DOR vs. 1/√ESS); Deeks' asymmetry test.
   e. SENSITIVITY ANALYSIS: Re-run primary bivariate model excluding studies with High risk of bias
      on q2_patient_selection. Compare pooled estimates.
   If n_included < 4: narrative synthesis only; state pooling not appropriate.

5. GRADE-DTA: Rate certainty of pooled sensitivity and specificity separately (High/Moderate/Low/
   Very Low) across 5 GRADE-DTA domains: risk of bias (QUADAS-2 summary), indirectness,
   inconsistency (I²), imprecision (95% CI width), publication bias (Deeks' p < 0.10).

CODE REQUIREMENTS:
- Write complete, runnable R script: 06_data_extraction/analysis_scripts/dta_analysis.R
- Libraries: mada, ggplot2, meta (for I² estimates)
- Save to 06_data_extraction/analysis_scripts/outputs/:
    sensitivity_forest.png
    specificity_forest.png
    sroc_curve.png          (SROC plane with 95% confidence ellipse)
    quadas2_traffic_light.png (heatmap of 7 domains × all studies)
    deeks_funnel.png        (if n_included ≥ 10)
- Include inline comments explaining each analytical step.

OUTPUT: (1) Synthesis report in markdown; (2) executable R script written to disk.
```

Write report to `06_data_extraction/synthesis_report.md`.
Write R script to `06_data_extraction/analysis_scripts/dta_analysis.R`.

## Step 5b: Programmer Code Review (DTA)

Dispatch Programmer agent to validate `dta_analysis.R`.

**Programmer agent prompt:**

```
[Insert Programmer system prompt from agent-roles.md]

TASK: Review and validate the DTA analysis R script.

SCRIPT: [Read 06_data_extraction/analysis_scripts/dta_analysis.R]
DATA COLUMNS: [Read first 3 lines of 06_data_extraction/extracted_data.csv]

DTA-specific checks (in addition to standard code review):
1. `library(mada)` is loaded; `reitsma()` and/or `hsroc()` are called correctly
2. 2×2 table inputs (TP, FP, FN, TN) are extracted from the correct CSV columns
3. Studies with NR values in TP/FP/FN/TN columns are filtered out before model fitting
4. Sensitivity and specificity are on the 0–1 scale (not 0–100) in all calculations
5. SROC ellipse uses `ROCellipse()` or equivalent — not a manually drawn circle
6. Deeks' funnel plot uses effective sample size (ESS) on the x-axis, not raw N
7. I² is calculated separately for sensitivity and specificity (logit-transformed)
8. Forest plot code produces paired plots (sensitivity forest + specificity forest)
9. All output PNGs saved to analysis_scripts/outputs/ with descriptive filenames
10. QUADAS-2 traffic light plot code reads the correct q2_* columns from the CSV

Fix any issues in-place and document changes.

OUTPUT: Code review report + corrected script if needed.
```

Write code review to `06_data_extraction/analysis_scripts/code_review.md`.

After Programmer review, dispatch Full Review per `references/reviewer-protocol.md`. Follow the same
iteration/escalation logic as the systematic review path (initialize `review_iteration = 0`).

## Step 6: Update project.yaml

Set `stages.data_extraction: completed`.
