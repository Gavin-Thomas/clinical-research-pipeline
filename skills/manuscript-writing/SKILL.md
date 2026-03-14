---
name: manuscript-writing
description: |
  Use when the user wants to write a manuscript, draft a paper, format for a journal,
  create a publication, or produce the final written output of a research project.
  Trigger on phrases like "write the manuscript", "draft the paper", "format for journal",
  "write up the results", "prepare for submission", or "write the discussion".
---

# Manuscript Writing & Formatting (Stage 7)

Draft a publication-ready manuscript formatted to a specific journal's guidelines.

## Prerequisites

- All prior stage outputs must exist (varies by project type)
- Read `project.yaml` for `target_journal`, `journal_guidelines_url`, `project_type`, `review_config`

## Edge Cases

Before dispatching any agents, perform these pre-flight checks. Stop immediately and report to the user if critical conditions are unmet.

**Required file check**

Read `project_type` from `project.yaml` first, then apply the project-type-specific checks below.

For **`case_report`** or **`case_series`** projects, check:
- `06_data_extraction/case_presentation.md` — if missing → STOP: "Case presentation not found. Please complete Stage 6 (Data Extraction) for your case report before running manuscript writing. Stage 6 for case reports produces `06_data_extraction/case_presentation.md`."
- Skip `screened_results.csv`, `extracted_data.csv`, and `synthesis_report.md` checks — these files are not produced for case reports.

For **`original_research`** projects, check:
- `06_data_extraction/synthesis_report.md` — if missing → STOP: "Analysis report not found. Please complete Stage 6 (Data Extraction) before running manuscript writing."
- `06_data_extraction/cleaned_data.csv` — if missing → STOP: "Cleaned dataset not found. Data collection and cleaning (Stage 6) must be complete before manuscript writing."
- Skip `screened_results.csv` check — original research projects do not have an abstract screening stage.

For **all other project types** (systematic_review, scoping_review, rapid_review, meta_analysis, qualitative_synthesis, diagnostic_test_accuracy), check:
- `05_screening/screened_results.csv` — if missing → STOP: "Abstract screening output not found. Please complete Stage 5 before running manuscript writing."
- `06_data_extraction/extracted_data.csv` — if missing → STOP: "Data extraction output not found. Please complete Stage 6."
- `06_data_extraction/synthesis_report.md` — if missing → STOP: "Synthesis report missing. Stage 6 may have been interrupted — re-run Data Extraction."
- For `qualitative_synthesis` projects: also check `06_data_extraction/extracted_quotes.csv` and `06_data_extraction/casp_appraisal.md`. If either is missing → STOP with the same guidance.

**Included study count**

_Skip this check for `case_report`, `case_series`, and `original_research` — it applies only to review types with a screening stage._

Count rows where `final_decision = INCLUDE` in `screened_results.csv`. Call this `n_included`.

- `n_included = 0` → **STOP**: "No studies passed screening. A manuscript cannot be drafted with zero included studies. Review inclusion/exclusion criteria or expand the database search before re-running."
- `n_included = 1` AND `review_config.meta_analysis: true` in project.yaml → Auto-set `meta_analysis = false` for this run. Print: "Only 1 study included — pooling not possible. Proceeding as single-study narrative synthesis." Direct Writers to add to Limitations: "A meta-analysis was not conducted as only one study met the inclusion criteria."
- `n_included = 2` AND `review_config.meta_analysis: true` → Auto-set `meta_analysis = false`. Print: "Only 2 studies included — random-effects pooling is unreliable with 2 estimates. Proceeding as narrative synthesis." Direct Writers to add to Limitations: "Statistical pooling was not performed due to the small number of included studies (n=2); a narrative synthesis was conducted instead."
- `n_included < 5` AND `review_config.network_meta_analysis: true` → Auto-set `network_meta_analysis = false`. Print: "Only [n_included] studies available — NMA requires a minimum network of 5 studies with adequate connectivity. Downgrading to pairwise synthesis."

**Journal guidelines check**

If `journal_guidelines_url` is blank, "TBD", or absent from project.yaml: note that Step 1 will require the user to manually paste guidelines. Do not stop — continue and prompt the user at Step 1.

## Process

> **Project-type routing:** Check `project_type` in `project.yaml` before proceeding:
>
> | project_type | Instructions |
> |---|---|
> | `qualitative_synthesis` | Read `skills/manuscript-writing/section-a-qualitative.md` and follow it exactly |
> | `case_report` | Read `skills/manuscript-writing/section-c-case-report.md` and follow it exactly |
> | `case_series` | Read `skills/manuscript-writing/section-c-case-report.md` and follow it exactly |
> | `diagnostic_test_accuracy` | Read `skills/manuscript-writing/section-d-dta.md` and follow it exactly |
> | `original_research` | Read `skills/manuscript-writing/section-e-original-research.md` and follow it exactly |
> | all others (systematic_review, scoping_review, rapid_review, meta_analysis) | Read `skills/manuscript-writing/section-b-quantitative.md` and follow it exactly |

Read the appropriate sub-file now, then execute its steps in full.
