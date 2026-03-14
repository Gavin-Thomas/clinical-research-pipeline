---
name: data-extraction
description: |
  Use when the user wants to extract data from papers, create extraction templates,
  synthesize study data, or perform data analysis for a review.
  Also triggers for case presentation structuring and data analysis in original research.
  Trigger on phrases like "extract data", "data extraction", "pull data from papers",
  "synthesize the data", "create extraction form", or "analyze the data".
---

# Data Extraction & Synthesis (Stage 6)

Extract structured data from full-text papers and synthesize findings.

## Prerequisites

- `05_screening/screened_results.csv` must exist (except for case reports)
- Full-text PDFs must exist in `06_data_extraction/full_texts/`
- Read `project.yaml` for `project_type`, `review_config`

## Edge Cases (Check Before Processing)

### Missing Full Texts

Before Step 2, compare PDFs in `06_data_extraction/full_texts/` against IDs in `05_screening/screened_results.csv` (INCLUDE decisions only):

- Generate `06_data_extraction/missing_full_texts.md` listing any expected PDFs not found:
  ```
  | # | Author | Year | DOI | PMID | Suggested retrieval |
  |---|--------|------|-----|------|---------------------|
  ```
- In `extracted_data.csv`, add a row for missing papers with all fields set to `"Full text unavailable"` and `quality_assessment` set to `"Cannot assess"`
- Print a warning listing missing papers and count; continue extraction with available PDFs
- Do NOT abort the stage due to missing texts

### PDF Readability Pre-Check

Before starting extraction, quickly test-read page 1 of each PDF to identify unreadable files upfront:

1. For each PDF in `06_data_extraction/full_texts/`:
   - Read page 1 using: `Read(file_path, pages="1")`
   - If the returned text is empty, very short (<50 characters), or garbled: flag as likely scanned/image-only
2. Write a readability report to `06_data_extraction/pdf_readability.md`:
   ```
   | Filename | Pages | Readable | Issue |
   |----------|-------|----------|-------|
   | Smith_2022_dupilumab.pdf | 12 | Yes | — |
   | Chen_2019_biologics.pdf | 8 | No | Scanned image — no text layer |
   ```
3. If any PDFs fail the pre-check, print immediately:
   ```
   ⚠️ [N] PDF(s) appear to be scanned images without a text layer:
     - [filename] — scanned/image-only (no extractable text)

   These cannot be read by the pipeline. Options:
   1. Run OCR (Adobe Acrobat → Recognize Text, or free: ocrmypdf)
   2. Replace with a text-based version from the publisher
   3. Extract data manually and add to extracted_data.csv

   Remaining [N] readable PDFs will be processed now.
   ```
4. Continue extraction with readable PDFs only — do not wait for user to fix unreadable ones

### Unreadable or Corrupted PDFs

If the Read tool fails on a PDF during extraction (encrypted, corrupted, or missed by pre-check):
- Add the paper to `extraction_progress.yaml` under `failed_reads: [filename, reason]`
- Set extracted row values to `"PDF unreadable — [reason]"`
- Continue with remaining papers; report failed reads in the stage summary

### Conflicting Data Between Studies

During Step 5 (Statistician synthesis), if studies report contradictory findings on the same outcome:
- The Statistician must flag these explicitly in `synthesis_report.md` under a **"Discrepant Findings"** subsection
- For each discrepancy, document: which studies conflict, the magnitude of disagreement, and the most likely methodological explanations (different populations, outcome definitions, follow-up duration, risk of bias)
- Do NOT average or dismiss conflicting values — present both findings with commentary
- In meta-analyses, include a sensitivity analysis excluding the outlier study

## Batching

If more than 10 papers to extract, batch at 10 per invocation. Track in `06_data_extraction/extraction_progress.yaml`.

## Process — Review Types (systematic_review, meta_analysis)

### Step 1: Dispatch Statistician for Template Design

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Design a data extraction template for this review.

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

PROJECT TYPE: [from project.yaml]
REPORTING STANDARD: [from project.yaml]
META-ANALYSIS: [from project.yaml review_config.meta_analysis]

Design a CSV extraction template with these column categories:
1. Study identification: author, year, title, journal, DOI, country
2. Study design: type, setting, follow-up duration
3. Population: sample size, age, sex distribution, condition severity
4. Intervention/exposure: description, dosage, duration, comparison
5. Outcomes: primary outcome, secondary outcomes, measurement tools
6. Results: effect sizes, confidence intervals, p-values, raw counts
7. Quality assessment: [appropriate tool based on study designs expected]

If meta_analysis is true, ensure the template captures sufficient quantitative data for pooling (effect sizes, standard errors, sample sizes per group).

If `review_config.clinical_coding` is `true` in project.yaml, add these columns after the population fields:
- `diagnosis_icd10`: ICD-10-CM code(s) for the primary condition studied (pipe-separated if multiple, e.g., "L40.0|L40.1")
- `diagnosis_icd11`: ICD-11 code(s) for the same condition (for interoperability)
- `procedure_cpt`: CPT code(s) for any procedure-based interventions (NR if not applicable)
- `snomed_population`: SNOMED CT concept ID(s) for the population descriptor
- `clinical_coding_notes`: Free text — original disease terminology used in the paper, any mapping uncertainties

OUTPUT: CSV header row + example row showing expected data format.
```

Write template to `06_data_extraction/extraction_template.csv`.

### Step 2: Partition Papers Across 3 Data Extractors

List all PDFs in `06_data_extraction/full_texts/`. Divide into 3 roughly equal groups. Dispatch 3 Data Extractor agents in parallel, each assigned their partition.

**Data Extractor agent prompt:**

```
[Insert Data Extractor system prompt from agent-roles.md]

TASK: Extract data from the following papers using the provided template.

EXTRACTION TEMPLATE:
[Insert extraction_template.csv header]

PAPERS ASSIGNED TO YOU:
[List of PDF filenames in this partition]

For each paper:
1. Read the PDF using the Read tool (for papers >20 pages, read in chunks: pages="1-20", then pages="21-40", etc.)
2. Extract every field in the template
3. Mark fields as "NR" (not reported) if the data is not available
4. Note the exact page/table/figure number where each value was found

SELF-CHECK (perform before outputting CSV rows):
1. Every paper in your assigned partition has exactly one output row — no paper is missing or duplicated
2. No template column is blank — use "NR" only for data genuinely absent from the paper, not for data that was hard to find
3. Effect sizes, confidence intervals, and p-values are transcribed character-for-character from the source; verify each against its page/table reference before writing
4. Every row's `source_notes` column contains a specific page or table number for at least the primary outcome values
5. If `review_config.clinical_coding` is `true`: every diagnosis/condition column has an ICD-10-CM code assigned; uncertain mappings use `[UNSPECIFIED]` with an explanatory note in `clinical_coding_notes` — no coding column is left blank
Apply any corrections inline before outputting.

OUTPUT: CSV rows matching the extraction template, one row per study. Include a "source_notes" column with page references.
```

### Step 3: Merge Extracted Data

Combine all 3 extractors' outputs into `06_data_extraction/extracted_data.csv`.

### Step 4: Full Review with Retry Logic

Initialize `review_iteration = 0`. Select a random sample (~20% of papers, minimum 3). Provide the sample papers + extracted data rows to reviewers. Dispatch Full Review per `references/reviewer-protocol.md`.

On **REVISE** verdict:
1. Build a REVISION REQUIRED block: a table of reviewer findings (finding | affected paper IDs | column names)
2. Re-dispatch new Data Extractor agents for the flagged rows only, prepending the REVISION REQUIRED block to the original prompt; each finding must be addressed explicitly
3. Update `extracted_data.csv` with the corrected rows
4. Increment `review_iteration`; re-dispatch reviewer with the revision history appended to context

On **APPROVE**: proceed to Step 5.

On **REJECT** or `review_iteration ≥ 2` without APPROVE: trigger full escalation per reviewer-protocol.md (print escalation banner, list all unresolved findings, pause pipeline).

**Review criteria for this stage:**
- Does the extracted data match what's in the source papers?
- Are "NR" fields genuinely not reported, or was data missed?
- Are effect sizes and confidence intervals transcribed correctly?
- Is the extraction template being followed consistently?

### Step 5: Dispatch Statistician for Synthesis

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Synthesize the extracted data.

EXTRACTED DATA:
[Read and insert 06_data_extraction/extracted_data.csv]

PROJECT TYPE: [from project.yaml]
META-ANALYSIS: [from project.yaml review_config.meta_analysis]

Produce:
1. STUDY CHARACTERISTICS TABLE: Summary of included studies (Table 1 format)
2. NARRATIVE SYNTHESIS: Organized by outcome, describe the findings across studies
3. RISK OF BIAS SUMMARY: Based on quality assessment scores in the data

If meta_analysis is true, this synthesis is descriptive only — meta-analytic pooling happens in the next sub-stage.

SELF-CHECK (before writing synthesis_report.md to disk):
1. Study characteristics table accounts for every study in `extracted_data.csv` — no study silently omitted
2. Narrative synthesis addresses each outcome named in the research question — no primary or secondary outcome is skipped
3. Risk of bias summary is grounded in quality scores from the extracted data, not generic statements ("most studies had low bias")
4. Any discrepant or contradictory findings between studies are flagged in a **"Discrepant Findings"** subsection with documented magnitude and methodological explanation
5. No statistics are invented — every value cited traces back to a specific column in `extracted_data.csv`
6. Heterogeneity in findings is noted even when meta-analysis is not being performed
Apply corrections before writing to disk.

OUTPUT: Structured synthesis report in markdown.
```

Write to `06_data_extraction/synthesis_report.md`.

### Step 6 (meta_analysis only): Meta-Analytic Pooling

If `project_type` is `meta_analysis` or `review_config.meta_analysis` is `true`:

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Perform meta-analytic pooling of the extracted data AND generate executable analysis code.

EXTRACTED DATA:
[Read and insert 06_data_extraction/extracted_data.csv]

Perform and document:
1. POOLED ESTIMATES: Random-effects model (REML) for each primary outcome; also report fixed-effects estimate for comparison
2. FOREST PLOTS: Markdown table (study, effect size, 95% CI, weight%) AND executable Python/R code to generate a publication-quality forest plot saved as a PNG
3. HETEROGENEITY: I², Q statistic with p-value, tau², 95% prediction interval
4. PUBLICATION BIAS: Funnel plot (code to generate) + Egger's test (if ≥10 studies); interpret trim-and-fill results
5. SENSITIVITY ANALYSES: Leave-one-out table; pre-specified subgroup analyses from project.yaml
6. GRADE CERTAINTY: Complete GRADE assessment table for each primary outcome
7. SUMMARY: Plain-language interpretation of pooled results for non-statisticians

CODE REQUIREMENTS:
- Write a complete, runnable Python script (preferred) or R script for the full analysis
- Script must read from: 06_data_extraction/extracted_data.csv
- Script must save outputs (forest plot PNG, funnel plot PNG, results CSV) to: 06_data_extraction/analysis_scripts/outputs/
- Include all imports and inline comments
- Save the script to: 06_data_extraction/analysis_scripts/meta_analysis_primary.py (or .R)

NOTE: Present forest plots as both markdown tables (for the report) and executable code (for actual figure generation).

OUTPUT: (1) Complete meta-analysis results report in markdown, (2) executable analysis script written to disk.
```

Write report to `06_data_extraction/meta_analysis_results.md`.
Write analysis script to `06_data_extraction/analysis_scripts/meta_analysis_primary.py`.

### Step 6a-verify: Programmer Code Review

After the Statistician writes analysis scripts, dispatch a Programmer agent to validate them before proceeding.

**Programmer agent prompt:**

```
[Insert Programmer system prompt from agent-roles.md]

TASK: Review and validate the analysis script(s) written by the Statistician.

SCRIPTS TO REVIEW:
[List all .py and .R files in 06_data_extraction/analysis_scripts/]

EXTRACTION DATA (for column name validation):
[Read first 3 lines of 06_data_extraction/extracted_data.csv]

PROJECT CONFIGURATION:
[Read project.yaml — check meta_analysis, network_meta_analysis, subgroup_analyses, sensitivity_analyses]

Review each script for:
1. All imports present and correct (no missing packages)
2. Input file path matches: 06_data_extraction/extracted_data.csv
3. Output paths exist or are created: 06_data_extraction/analysis_scripts/outputs/
4. Column names in code match actual CSV column headers — flag any mismatches
5. Statistical method matches project.yaml specification (REML vs DL, fixed vs random)
6. Effect size formulas are correct for the data type (OR from counts, MD from means±SD, etc.)
7. Edge cases handled: NR values filtered before numeric operations, single-study subgroups, zero events
8. Random seed set for any stochastic operations
9. Output figures saved with descriptive filenames
10. Final stdout summary prints key results

If any issue is found:
- Fix the script in place (write the corrected version to the same path)
- Document every change in a code review report

OUTPUT: Code review report + corrected script(s) if needed.
```

Write code review report to `06_data_extraction/analysis_scripts/code_review.md`.

### Step 6b (network_meta_analysis only): Network Meta-Analysis

If `review_config.network_meta_analysis` is `true` in project.yaml, also dispatch:

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Perform network meta-analysis (NMA).

EXTRACTED DATA:
[Read and insert 06_data_extraction/extracted_data.csv]

PAIRWISE RESULTS (if available):
[Read 06_data_extraction/meta_analysis_results.md if it exists — skip if absent]

Produce and document:
1. NETWORK GEOMETRY: Enumerate all treatment nodes and direct comparison edges; count studies
   per edge; flag sparse edges (single-study comparisons). Generate a network diagram
   (node size ∝ total participants; edge width ∝ study count).
2. CONSISTENCY CHECK: Global design-by-treatment interaction test + local node-splitting for
   the 3-5 clinically most important pairwise comparisons. If global p < 0.1 or any
   node-split p < 0.1, report and investigate sources (population, outcome definition,
   risk-of-bias differences between direct and indirect evidence).
3. POOLED NMA ESTIMATES: League table of all pairwise NMA estimates (point estimate, 95% CrI).
   Distinguish direct vs. indirect evidence (shade differently in the table).
   Present all-vs-reference-treatment forest plot.
4. RANKING: SUCRA (frequentist) or P-score (Bayesian) for each treatment; mean rank and full
   rank probability histogram.
5. HETEROGENEITY: Between-study variance (tau²) for the overall network model; I².
6. GRADE-NMA: Standard GRADE 5 domains plus:
   - Indirectness: downgrade if estimate relies predominantly on indirect evidence
   - Incoherence: downgrade if global or local inconsistency was detected
7. SENSITIVITY: Repeat NMA excluding high-RoB studies; compare SUCRA/P-score rankings to
   confirm treatment hierarchy stability.

CODE REQUIREMENTS:
- Preferred: R with `netmeta` (frequentist) or `gemtc`/`BUGSnet` (Bayesian)
- For Bayesian NMA: ≥4 chains, ≥10,000 iterations; report R-hat < 1.01 and ESS > 1,000
- Generate and save to 06_data_extraction/analysis_scripts/outputs/:
    network_geometry.png    — network diagram
    nma_forest.png          — all vs. reference forest plot
    sucra_ranking.png       — SUCRA/P-score ranking plot
    nma_league_table.csv    — full pairwise comparison matrix
- Save script to: 06_data_extraction/analysis_scripts/nma_analysis.R (or .py)

OUTPUT: Complete NMA results in markdown + executable analysis script written to disk.
```

Write to `06_data_extraction/nma_results.md`.
Write analysis script to `06_data_extraction/analysis_scripts/nma_analysis.R`.

### Step 6b-verify: Programmer Code Review (NMA)

Dispatch Programmer agent to review the NMA script using the same prompt template as Step 6a-verify, but targeting `nma_analysis.R` (or `.py`). Additional NMA-specific checks:

- Correct network geometry construction (treatment nodes, comparison edges)
- Consistency model specification (consistency vs. inconsistency model)
- For Bayesian: chain count ≥ 4, iterations ≥ 10,000, convergence diagnostics (R-hat, ESS) are computed and printed
- SUCRA/P-score calculations use the correct posterior samples or frequentist estimates
- League table output is a proper pairwise matrix (symmetric, diagonal = reference)

Append NMA review to `06_data_extraction/analysis_scripts/code_review.md`.

### Step 7: Update project.yaml

Set `stages.data_extraction: completed`.

## Process — case_report

Read `skills/data-extraction/section-case-report.md` and follow it exactly.

## Process — case_series

Read `skills/data-extraction/section-case-report.md` and follow it exactly. The section-case-report.md file handles both single case reports and case series — it detects `project_type: case_series` from project.yaml and adds cross-case comparison steps.

## Process — scoping_review

Read `skills/data-extraction/section-scoping-review.md` and follow it exactly. Scoping reviews use data charting (thematic categorization), not standard data extraction for pooling.

## Process — rapid_review

Read `skills/data-extraction/section-rapid-review.md` and follow it exactly.

## Process — original_research

Read `skills/data-extraction/section-original-research.md` and follow it exactly.

## Process — qualitative_synthesis

Read `skills/data-extraction/section-qualitative-synthesis.md` and follow it exactly.

## Process — diagnostic_test_accuracy

Read `skills/data-extraction/section-diagnostic-test-accuracy.md` and follow it exactly.
