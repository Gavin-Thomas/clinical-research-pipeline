# Data Extraction — original_research

For primary research projects, Stage 6 covers designing the data collection instrument, validating
collected data, and running pre-specified statistical analyses. This is distinct from extracting data
from existing papers.

## Step 1: Design Data Collection Form

Dispatch Statistician:

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Design a structured data collection form for this primary research study.

STUDY PROTOCOL:
[Read and insert 03_inclusion_exclusion/study_protocol.md]

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

Design a REDCap-compatible data dictionary with:
1. Participant identification: de-identified ID, enrollment date, site (if multi-center)
2. Demographic variables as specified in the protocol (age, sex, ethnicity, etc.)
3. Primary exposure or intervention variables
4. Primary outcome measure(s) with measurement timing and instrument name
5. Secondary outcomes
6. Confounders and covariates pre-specified in the protocol
7. Adverse events / safety outcomes (if applicable)
8. Data quality fields: missing data flags, out-of-range flags, query status

For each field specify: field_name, data_type (text/integer/date/radio/checkbox/calc),
validation_rule (range or permitted values), and instrument question text.

OUTPUT: Data dictionary as CSV + instrument structure suitable for REDCap import.
```

Write to `06_data_extraction/data_collection_form.csv`.

## Step 2: Data Validation and Cleaning

If raw data files exist in `06_data_extraction/raw_data/`:
- Dispatch Statistician to validate against the collection form (type checks, range checks,
  missing data pattern analysis)
- Generate `06_data_extraction/missing_data_report.md` (counts and patterns by variable and
  participant; flag variables >10% missing for PI attention)
- Produce cleaned dataset at `06_data_extraction/cleaned_data.csv` with an appended
  data cleaning log (what was changed, how many values affected)

If data collection is not yet complete, pause:

```
⏸️  DATA COLLECTION IN PROGRESS

Data collection form: 06_data_extraction/data_collection_form.csv
Place raw data files in: 06_data_extraction/raw_data/

When collection is complete: "I've placed the data in the raw_data folder — please continue"
```

## Step 3: Statistical Analysis

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Run all pre-specified statistical analyses for this primary research study and produce
an analysis report, executable analysis script, and all tables/figures needed for manuscript writing.

STUDY PROTOCOL:
[Read and insert 03_inclusion_exclusion/study_protocol.md]

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

DATA:
[Read and insert 06_data_extraction/cleaned_data.csv]

SUBGROUP ANALYSES: [from project.yaml review_config.subgroup_analyses — list each or "none pre-specified"]
SENSITIVITY ANALYSES: [from project.yaml review_config.sensitivity_analyses — list each or "none pre-specified"]
REPORTING STANDARD: [from project.yaml review_config.reporting_standard — CONSORT or STROBE]

Produce the following:

1. **Descriptive statistics and Table 1 (Participant Characteristics)**
   - Continuous variables: mean ± SD if normally distributed; median (IQR) if skewed — assess normality
   - Categorical variables: n (%)
   - Columns: by study group if applicable (RCT or case-control), plus Total column
   - Do NOT include p-values in Table 1 for RCTs (CONSORT 2010 requirement)
   - For observational studies: include standardized mean difference (SMD) column if comparing groups

2. **Missing data summary**
   - Count and % missing per variable for the primary outcome and all key predictors
   - State overall MCAR/MAR/MNAR assessment where possible
   - If >5% missing for the primary outcome or a key predictor: perform complete-case AND
     multiple imputation sensitivity analysis (5 imputations minimum); report both

3. **Primary analysis**
   - Run as pre-specified in study_protocol.md (regression model, survival analysis, t-test, etc.)
   - Report: point estimate, 95% CI, exact p-value (not "p < 0.05")
   - Report both unadjusted and adjusted estimates for observational designs (list covariates)
   - For RCTs: report absolute risk reduction and NNT/NNH in addition to relative measures (if binary outcome)
   - For time-to-event outcomes: Kaplan-Meier curves + log-rank test + Cox regression HR with 95% CI

4. **Secondary analyses**
   - One result block per secondary outcome (same format as primary)

5. **Pre-specified subgroup analyses** (if any in project.yaml)
   - Report interaction p-value for each subgroup variable — do NOT report individual group p-values
   - Include a forest plot of subgroup estimates (as executable code)

6. **Pre-specified sensitivity analyses** (if any in project.yaml)
   - State what was varied and how results changed vs. primary analysis
   - Flag any substantial differences (point estimate changes by >20% or CI shifts to cross null)

7. **Executable analysis script**
   - Write a complete, runnable Python (preferred) or R script
   - Python: use pandas, scipy.stats, statsmodels, lifelines (survival), matplotlib/seaborn for figures
   - R: use tidyverse, survival, ggplot2, tableone
   - Script must: read from `06_data_extraction/cleaned_data.csv`, run all analyses above in sequence,
     save all output figures as PNG to `06_data_extraction/analysis_scripts/outputs/`,
     save results tables as CSV to `06_data_extraction/analysis_scripts/outputs/`,
     print a human-readable summary of all key results to stdout
   - Save script to `06_data_extraction/analysis_scripts/primary_analysis.py` (or `.R`)
   - Include brief inline comments explaining each major analysis step

SELF-CHECK (required before writing any output to disk):
1. Table 1 accounts for all participants in cleaned_data.csv — total N matches row count
2. Primary outcome reported with both unadjusted AND adjusted estimates if observational — not just one
3. Every p-value is exact (not "p < 0.05" or "p = NS") — re-check all
4. Missing data pattern is described; if >5% missing for primary outcome or key predictor, sensitivity
   analysis with imputation is included
5. For RCTs: Table 1 does NOT contain p-values for baseline characteristics — remove any that appeared
6. Analysis script is syntactically complete — runs without manual editing; all imports present, all
   file paths use relative paths from project root
7. All pre-specified subgroup analyses (from project.yaml) are addressed — none skipped
8. All pre-specified sensitivity analyses (from project.yaml) are addressed — none skipped
Apply any corrections before writing outputs to disk.

OUTPUT: (1) Synthesis report in markdown written to 06_data_extraction/synthesis_report.md,
(2) executable analysis script written to disk.
```

Write synthesis report to `06_data_extraction/synthesis_report.md`.
Write analysis script to `06_data_extraction/analysis_scripts/primary_analysis.py` (or `.R`).

## Step 3b: Programmer Code Review

Dispatch Programmer agent to validate the primary analysis script before review.

**Programmer agent prompt:**

```
[Insert Programmer system prompt from agent-roles.md]

TASK: Review and validate the primary analysis script for this original research study.

SCRIPT: [Read 06_data_extraction/analysis_scripts/primary_analysis.py or .R]

STUDY PROTOCOL: [Read 03_inclusion_exclusion/study_protocol.md — for specified statistical methods]

DATA COLUMNS: [Read first 3 lines of 06_data_extraction/cleaned_data.csv]

Review for:
1. All imports/libraries present — script runs without installing unlisted packages
2. Input path matches: 06_data_extraction/cleaned_data.csv
3. Output paths: analysis_scripts/outputs/ — directory created if absent
4. Column names in code match actual CSV headers
5. Statistical test matches protocol specification (e.g., logistic regression if binary outcome, Cox if time-to-event)
6. For adjusted models: covariates listed match those in study_protocol.md
7. Missing data handling: NR/NA values handled before analysis, imputation method matches protocol if >5% missing
8. Table 1 generation: correct descriptive stats (mean±SD for normal, median(IQR) for skewed, n(%) for categorical)
9. No p-values in Table 1 for RCT designs
10. Subgroup forest plot code produces readable output with interaction p-values
11. All output figures saved as PNG; results CSVs saved; stdout summary printed

Fix any issues in-place and document changes.

OUTPUT: Code review report + corrected script if needed.
```

Write code review to `06_data_extraction/analysis_scripts/code_review.md`.

## Step 4: Full Review

Initialize `review_iteration = 0`. Dispatch Full Review per `references/reviewer-protocol.md` on `06_data_extraction/synthesis_report.md` and the analysis script(s) in `06_data_extraction/analysis_scripts/`.

**Review criteria for original_research Stage 6 (provide these to each reviewer in the dispatch prompt):**

Statistical Completeness:
- Does `synthesis_report.md` cover all pre-specified analyses from `study_protocol.md`? (primary, secondary, subgroup, sensitivity)
- Is Table 1 (participant characteristics) present and correctly formatted: continuous as mean ± SD or median (IQR); categorical as n (%)?
- For RCT: does Table 1 omit p-values for baseline comparisons (CONSORT requirement)?
- Are primary outcome results reported with BOTH unadjusted and adjusted estimates, each with 95% CI and exact p-value (not "p < 0.05")?
- For RCT binary outcomes: are absolute risk reduction and NNT or NNH reported alongside the relative measure?
- For observational: are confounders from `study_protocol.md` included in the adjusted model?

Missing Data and Assumptions:
- Is the missing data pattern described for the primary outcome and key predictors?
- If >5% missing for the primary outcome or any key predictor, is a sensitivity analysis with multiple imputation present alongside the complete-case analysis?

Subgroup and Sensitivity Analyses:
- Are all pre-specified subgroup analyses from `project.yaml` present with interaction p-values — not individual within-subgroup p-values?
- Are all pre-specified sensitivity analyses from `project.yaml` present with results compared to the primary analysis?
- Are any substantial differences (point estimate shifts >20% or CI crosses null) explicitly flagged?

Analysis Script:
- Does `analysis_scripts/primary_analysis.py` (or `.R`) exist?
- Does the script contain all required imports and no placeholder variables?
- Are file paths relative to the project root, referencing `06_data_extraction/cleaned_data.csv`?
- Are output figures saved to `analysis_scripts/outputs/`?
- Does the script print a readable summary of key results to stdout?

On **REVISE** verdict:
1. Build a REVISION REQUIRED table with all reviewer findings: finding | affected section | required action
2. Re-dispatch the Statistician for targeted corrections only — do NOT re-run the full analysis; address only the flagged findings
3. Update `synthesis_report.md` and analysis script(s) with corrections
4. Increment `review_iteration`; re-dispatch Full Review with revision history prepended

On **APPROVE**: proceed to Step 5.

On **REJECT** or `review_iteration ≥ 2` without APPROVE: trigger full escalation per `reviewer-protocol.md` (print escalation banner, list all unresolved findings, pause pipeline).

## Step 5: Update project.yaml

Set `stages.data_extraction: completed`.
