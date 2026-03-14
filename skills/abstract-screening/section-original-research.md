# Abstract Screening — Original Research (Data Cleaning) Process

_Read and follow this file when `project_type: original_research`. For original research, Stage 5 is the **data cleaning and validation** stage — not abstract screening. Raw collected data (REDCap exports, survey responses, chart review spreadsheets, EHR extracts) must be cleaned, validated, and prepared before statistical analysis in Stage 6._

---

## Prerequisites (original_research)

- `03_inclusion_exclusion/study_protocol.md` must exist — contains variable definitions, inclusion/exclusion criteria for participants, and the data collection instrument structure.
- At least one raw data file must exist in `04_database_search/abstracts/` (this directory is repurposed for raw data uploads in original research projects) **or** in `06_data_extraction/raw_data/`. Check both locations.

If no raw data file is found, print and stop:

```
❌ No raw data file found.

For original research projects, Stage 5 expects your raw collected data.

Please place your data file (REDCap .csv export, survey export, chart review spreadsheet) in:
  [project_dir]/06_data_extraction/raw_data/

Accepted formats: .csv, .xlsx (will be read as .csv), .tsv
Column headers should match the variable names in your study_protocol.md.

When ready: "I've added the data file to raw_data/ — please continue"
```

---

## Step 1: Read and Profile the Raw Data

Read the raw data file(s) from `06_data_extraction/raw_data/` (preferred) or `04_database_search/abstracts/` (fallback). Also read `03_inclusion_exclusion/study_protocol.md` to obtain the variable list, variable types, and participant eligibility criteria.

Profile the dataset and write `05_screening/data_profile.md`:

```markdown
# Raw Data Profile

| Metric | Value |
|--------|-------|
| Source file(s) | [filename(s)] |
| Total rows (pre-cleaning) | [N] |
| Total columns | [N] |
| Expected variables (from study_protocol.md) | [N] |
| Variables present in data | [N] |
| Variables missing from data | [list, or "none"] |
| Extra columns not in protocol | [list, or "none"] |

## Per-Variable Summary
| Variable | Type | Non-null | Null | Null% | Notes |
|---------------|------|----------------|------------|--------|-------|
| [var] | [numeric/categorical/date/text] | [N] | [N] | [%] | [any flags] |

## Eligibility Screening Summary (pre-cleaning)
- Participants loaded: [N]
- Participants meeting all inclusion criteria: [N] (estimated — confirmed in Step 2)
- Participants excluded: [N] (estimated)
```

---

## Step 2: Dispatch Data Extractor for Cleaning and Eligibility Filtering

Dispatch a **single Data Extractor** agent for data cleaning (no triple-redundancy — this is a deterministic cleaning task, not an interpretation task):

```
[Insert Data Extractor system prompt from agent-roles.md]

TASK: Clean and validate a raw dataset for original research analysis.

STUDY PROTOCOL:
[Read and insert 03_inclusion_exclusion/study_protocol.md]

RAW DATA FILE:
[Insert file path to raw data — agent will read it]

DATA PROFILE:
[Insert the data_profile.md content generated in Step 1]

CLEANING INSTRUCTIONS:

1. ELIGIBILITY FILTERING — Apply participant-level inclusion/exclusion criteria from study_protocol.md:
   - For each participant row, evaluate each inclusion and exclusion criterion.
   - Remove rows that meet any exclusion criterion. Record the exclusion reason.
   - Track counts: enrolled → eligible → analyzed.
   - Write excluded participants to: 05_screening/excluded_participants.csv
     Columns: row_id, exclusion_criterion, exclusion_reason
   - Retain eligible participants for cleaning.

2. MISSING DATA AUDIT — For each variable:
   - Count and record null/blank values per column.
   - Flag any variable with >20% missingness as HIGH MISSINGNESS in a comment column.
   - Do NOT impute missing data at this stage — imputation is a Stage 6 Statistician decision.
   - For dates: standardize to ISO format (YYYY-MM-DD); flag implausible dates (future dates, dates before study start).

3. DUPLICATE DETECTION — Identify duplicate participant entries:
   - Exact duplicates (all columns identical): remove, retaining first occurrence; record removed row_id in 05_screening/duplicates_removed.csv.
   - Potential duplicates (same participant ID or highly similar demographic entries — same age, sex, DOB, site): flag as POSSIBLE_DUPLICATE in a flag column; do NOT remove — these require human review.

4. RANGE AND CONSISTENCY CHECKS — For each numeric variable defined in study_protocol.md with a plausible range:
   - Flag values outside the specified range as OUT_OF_RANGE in a flag column.
   - Flag logical inconsistencies (e.g., age < 0, follow-up duration > study period, conflicting diagnosis codes).
   - Do NOT modify or delete flagged values — mark them for Statistician review.

5. VARIABLE STANDARDIZATION:
   - Recode categorical variables to match the protocol-specified coding scheme (e.g., Male=1, Female=2; Yes=1, No=0).
   - Standardize text fields: trim whitespace, convert to consistent case.
   - Rename any columns that differ from the protocol variable names (document all renames in the cleaning_log).

6. OUTPUT:
   - Write cleaned dataset to: 05_screening/cleaned_data.csv
   - Write a cleaning log to: 05_screening/cleaning_log.md documenting every change made (what, why, row IDs affected).

CLEANING LOG FORMAT:
```markdown
# Data Cleaning Log

## Summary
- Rows pre-cleaning: [N]
- Participants excluded (eligibility): [N]
- Duplicate rows removed (exact): [N]
- Possible duplicates flagged (kept): [N]
- Rows in cleaned_data.csv: [N]

## Eligibility Exclusions
| Row ID | Criterion | Reason |
|--------|-----------|--------|
| [id] | [criterion name] | [value that triggered exclusion] |

## Duplicates Removed
| Removed Row ID | Retained Row ID | Basis |
|----------------|-----------------|-------|
| [id] | [id] | [all fields identical / same subject ID] |

## Variable Changes
| Variable | Change | Rows Affected | Reason |
|----------|--------|---------------|--------|
| [var] | [renamed/recoded/flagged] | [N] | [reason] |

## Data Quality Flags (in cleaned_data.csv flag columns)
| Variable | Flag Type | Count | Notes |
|----------|-----------|-------|-------|
| [var] | OUT_OF_RANGE | [N] | [range: expected X–Y] |
| [var] | HIGH_MISSINGNESS | — | [X% missing] |
| [var] | POSSIBLE_DUPLICATE | [N] | [based on: ...] |
```

SELF-CHECK before finalizing your output:
1. Row count: cleaned_data.csv row count + excluded_participants.csv row count = raw data row count (before exact duplicate removal). ✗✗ if the numbers do not add up.
2. Eligibility: Every excluded participant has a specific exclusion criterion cited from study_protocol.md — no generic "does not meet criteria".
3. Flags: Every OUT_OF_RANGE and POSSIBLE_DUPLICATE flag cites the specific variable and threshold or basis.
4. No imputation: No missing values have been filled or estimated — only original or standardized values are present.
5. Protocol alignment: Every column in cleaned_data.csv corresponds to a variable in study_protocol.md (or is a flag column added by this cleaning process). Extra columns from the raw data not in the protocol are retained but documented in the cleaning_log.
6. Cleaning log completeness: Every structural change to the data (exclusion, recode, rename, duplicate removal) is logged with row IDs.
Correct any violations before writing output files.
```

---

## Step 3: Quick Review

Initialize `review_iteration = 0`. Dispatch Quick Review per `references/reviewer-protocol.md` on the cleaning log and cleaned_data.csv.

**Review criteria for original_research data cleaning:**
- Does `excluded_participants.csv` row count + `cleaned_data.csv` row count account for all raw data rows (minus exact duplicates)?
- Does each exclusion row cite a specific criterion from `study_protocol.md`?
- Are data quality flags (OUT_OF_RANGE, POSSIBLE_DUPLICATE) present for implausible values?
- Is there evidence of imputation (non-NR values in fields that should be missing)? This is a REJECT trigger — no imputation at this stage. ✗✗
- Is the cleaning log complete with per-variable change documentation?

On **REVISE:** Return cleaning_log.md and the specific findings to the Data Extractor. Re-run Steps 2–3 on the flagged issues only. Do not re-clean the full dataset. Increment `review_iteration`. Maximum 2 iterations per reviewer-protocol.md Quick Review rules.

---

## Step 4: Update project.yaml + Print Summary

Set `stages.abstract_screening: completed`.

Print:

```
✅ Data cleaning complete. Cleaned dataset ready for analysis.

📊 Participant Flow (for CONSORT/STROBE diagram):
- Participants in raw data file: [N]
- Exact duplicates removed: [N]
- Excluded (eligibility criteria): [N]
  [List each criterion with count: e.g., "  • Age < 18: [N]"]
- Possible duplicates flagged (retained — human review needed): [N]
- Participants in cleaned_data.csv: [N]

⚠️  Data quality flags requiring Statistician review:
[List any HIGH_MISSINGNESS or OUT_OF_RANGE flags, or "None — dataset is clean"]

📂 Output files:
  05_screening/cleaned_data.csv         — cleaned dataset for Stage 6 analysis
  05_screening/excluded_participants.csv — exclusion log (for participant flow diagram)
  05_screening/cleaning_log.md          — complete audit trail of all changes
  05_screening/data_profile.md          — raw data profile

[If POSSIBLE_DUPLICATE > 0:]
⚠️  MANUAL REVIEW RECOMMENDED: [N] possible duplicate participants are flagged in
  cleaned_data.csv (flag column: POSSIBLE_DUPLICATE). Review these rows before
  proceeding to analysis. Confirm or deny duplicates, then say "continue to data extraction".

When ready: Run /data-extraction to begin Stage 6 statistical analysis.
```
