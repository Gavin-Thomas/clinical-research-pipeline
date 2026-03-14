# Changelog

## [1.0.48] — 2026-03-14

### Fixed

- **Area 7 (Agent Team Coordination) — Manuscript Writer B context injection gap: landscape report missing from all 4 section files; CERQual assessment missing from qualitative synthesis Writer B**

  **Problem — Writer B never receives the landscape report:**

  Every manuscript section file (section-a-qualitative.md, section-b-quantitative.md, section-d-dta.md, section-e-original-research.md) divides writing between Writer A (Introduction + Methods) and Writer B (Results + Discussion). Writer A in all four files correctly injects `01_literature_search/landscape_report.md` as a source material — needed for Introduction background and framing. However, Writer B in all four files was NOT given the landscape report.

  This matters because the Discussion section in every project type explicitly instructs Writer B to compare findings with existing literature "from the landscape report":
  - `section-b-quantitative.md` Writer B: "Compare with existing literature (reference landscape report)"
  - `section-a-qualitative.md` Writer B: "Comparison with existing quantitative and qualitative literature (from landscape report)"
  - `section-d-dta.md` Writer B: "Comparison with prior systematic reviews and individual accuracy studies (from landscape report)"
  - `section-e-original-research.md` Writer B: "Compare findings with the existing literature cited in the landscape report"

  Without the landscape report in context, Writer B agents must either: (a) fabricate comparative literature references, (b) omit the literature comparison entirely, or (c) produce a generic Discussion that ignores the specific literature gap identified in Stage 1. All three outcomes degrade manuscript quality — especially for the Discussion, where comparison with prior literature is a required item in PRISMA, STARD, CONSORT, and STROBE.

  **Problem — Qualitative synthesis Writer B lacks CERQual assessment:**

  `section-a-qualitative.md` Writer B is instructed to produce: "CERQual confidence rating (High / Moderate / Low / Very Low) with brief justification" for each analytical theme, and a "CERQual Summary of Qualitative Findings table (Table 2): one row per synthesized finding — finding statement, studies contributing, CERQual assessment, explanation." However, the CERQual assessment is formally produced in Stage 6 (Step 5 of section-qualitative-synthesis.md) and written to `06_data_extraction/cerqual_assessment.md` — which was never included in Writer B's source materials.

  Without `cerqual_assessment.md`, Writer B would need to reconstruct confidence ratings from the synthesis_report.md, which may not organize CERQual domain ratings in a structured, per-finding format. The formal CERQual document is the authoritative source for domain-by-domain ratings (methodological limitations, coherence, adequacy, relevance) and the overall confidence level per finding.

  **Changes:**

  **`skills/manuscript-writing/section-b-quantitative.md` — Writer B SOURCE MATERIALS:**
  - Added: `- Landscape report (for Discussion — existing literature comparison): [Read and insert 01_literature_search/landscape_report.md]`
  - Updated Discussion instruction: "Compare with existing literature (cite specific reviews and studies from landscape_report.md)"

  **`skills/manuscript-writing/section-a-qualitative.md` — Writer B SOURCE MATERIALS:**
  - Added: `- Landscape report (for Discussion — existing literature comparison): [Read 01_literature_search/landscape_report.md]`
  - Added: `- CERQual assessment: [Read 06_data_extraction/cerqual_assessment.md]`
  - Updated Discussion instruction: "Comparison with existing quantitative and qualitative literature (cite specific studies from landscape_report.md)"

  **`skills/manuscript-writing/section-d-dta.md` — Writer B SOURCE MATERIALS:**
  - Added: `- Landscape report (for Discussion — comparison with prior systematic reviews and accuracy studies): [Read 01_literature_search/landscape_report.md]`
  - Updated Discussion instruction: "Comparison with prior systematic reviews and individual accuracy studies (cite specific work from landscape_report.md)"

  **`skills/manuscript-writing/section-e-original-research.md` — Writer B SOURCE MATERIALS:**
  - Added: `- Landscape report (for Discussion — comparison with existing literature): [Read 01_literature_search/landscape_report.md]`
  - Updated Discussion instruction: "Compare findings with the existing literature cited in landscape_report.md (support or contradict? — cite specific studies)"

  **`plugin.json`** — version bumped to 1.0.48

  **Files changed:**
  - `skills/manuscript-writing/section-a-qualitative.md`
  - `skills/manuscript-writing/section-b-quantitative.md`
  - `skills/manuscript-writing/section-d-dta.md`
  - `skills/manuscript-writing/section-e-original-research.md`
  - `.claude-plugin/plugin.json`

---

## [1.0.47] — 2026-03-14

### Fixed

- **Area 5 (Quality & Proofreading) + Area 1 (Autonomy & Self-Correction) — `original_research` Stage 5 in `reviewer-protocol.md`: missing data-cleaning checklist added**

  **Problem — No Stage 5 original_research checklist in `reviewer-protocol.md`:**

  `skills/abstract-screening/section-original-research.md` Step 3 dispatches a Quick Review per `references/reviewer-protocol.md` on `cleaned_data.csv` and `cleaning_log.md`. But the Stage 5 checklist in `reviewer-protocol.md` covered only abstract screening outputs — four universal blocks (Completeness, Decision Quality, Conflict Resolution, Deduplication) that reference `screened_results.csv`, screening verdicts (INCLUDE/EXCLUDE/UNCERTAIN), conflict reconciliation, and DOI/fuzzy-title deduplication. None of these are applicable to a raw-data cleaning output.

  This was the last stage missing an `original_research` conditional block. The pattern was already established at Stages 3, 4, and 6:

  - Stage 3: `original_research` block added (1.0.45) — PICO Alignment / Operationalizability checks replaced with study protocol checklist
  - Stage 4: `original_research` block added (1.0.44) — search strategy checks replaced with data collection instrument checklist
  - Stage 6: `original_research` block added (1.0.46) — extraction completeness / synthesis quality checks replaced with statistical analysis checklist
  - Stage 5: ← **missing** (fixed in this version)

  **Consequences of the missing checklist:**

  1. **Reviewer applied nonsensical criteria to data cleaning outputs:** An Independent Reviewer dispatched for original_research Stage 5 would check: "Every abstract from the input file has exactly one decision row", "Three screener columns present (Screener 1, Screener 2, Screener 3) plus a Reconciled Decision column", "All 3-way disagreements are flagged as CONFLICT" — none of which exist in a cleaned dataset. The reviewer would either: (a) declare APPROVE because no screening rows were present to fail checks, or (b) produce a REJECT citing missing screener columns that will never exist. Both outcomes are wrong.

  2. **Silent participant loss went undetected:** The most critical data cleaning error — exclusion filtering removes too many or too few participants — was never caught because the reviewer had no row-count reconciliation check. A data cleaning run that silently dropped 50 eligible participants (wrong criterion applied, or criterion applied to the wrong column) would pass review if the reviewer had no reconciliation item to check.

  3. **No imputation detection:** Stage 5 for original research explicitly prohibits imputation (imputation is the Stage 6 Statistician's decision). A Data Extractor that imputed missing values during cleaning would pass the Stage 5 review unchallenged, because no checklist item covered this.

  4. **Data quality flags not verified:** OUT_OF_RANGE and POSSIBLE_DUPLICATE flags in `cleaned_data.csv` were never reviewed. Incorrectly applied or missing flags propagate silently to Stage 6 — the Statistician treats flagged values as requiring review but would miss unflagged out-of-range values entirely.

  **Changes:**

  **`references/reviewer-protocol.md` — Stage 5 original_research conditional block (~18 lines) inserted before the Qualitative Synthesis conditional block:**
  - Disclaimer: standard Completeness, Decision Quality, Conflict Resolution, and Deduplication checks do NOT apply for original_research
  - 9 binary pass/fail items with ✗✗ reject-threshold markers on critical ones:
    - Row count reconciliation: cleaned + excluded = raw − exact duplicates (✗✗)
    - Exclusion criterion specificity: named criterion from study_protocol.md required (✗✗)
    - No imputation in cleaned_data.csv: REJECT trigger (✗✗)
    - OUT_OF_RANGE flags present for variables with documented plausible ranges (✗✗)
    - HIGH_MISSINGNESS flag applied to variables with >20% missingness
    - POSSIBLE_DUPLICATE rows retained with flag column populated (✗✗)
    - Column names correspond to study_protocol.md variables or are documented as renamed (✗✗)
    - cleaning_log.md documents every structural change
    - data_profile.md lists missing variables explicitly

  **Why this matters:** Original research (chart reviews, cohort studies, retrospective studies, RCTs) is one of the 8 supported project types. The data cleaning stage is the only place where participant eligibility is enforced and data quality flags are applied before analysis. Without a Stage 5 reviewer that understands data cleaning (rather than abstract screening), a PI using the pipeline for original research would have no automated quality gate between raw data and the Stage 6 Statistician — who reads `cleaned_data.csv` as the authoritative, ready-for-analysis dataset.

  **Files changed:**
  - `references/reviewer-protocol.md` — Stage 5 original_research conditional block added (~18 items, ~25 lines)
  - `.claude-plugin/plugin.json` — version bumped to 1.0.47

---

## [1.0.46] — 2026-03-14

### Fixed

- **Area 1 (Autonomy & Self-Correction) + Area 5 (Quality & Proofreading) — `original_research` Stage 6 in `data-extraction/section-original-research.md`: missing Full Review step added; Stage 6 original_research checklist added to `reviewer-protocol.md`**

  **Problem — Missing Full Review in Stage 6 original_research path:**

  `skills/data-extraction/section-original-research.md` had no quality review between Step 3 (Statistician statistical analysis) and Step 4 (project.yaml update). Every other project type in Stage 6 dispatches a Full Review after synthesis (review_types path Step 4, rapid_review, qualitative_synthesis, DTA all have reviewer dispatches). The original_research path was the only Stage 6 path that wrote `synthesis_report.md` and analysis scripts to disk and then immediately set `stages.data_extraction: completed` — with no review.

  Consequences of the missing review:

  1. **Statistical errors propagate to manuscript writing:** The Statistician is instructed to run primary analysis, secondary analyses, subgroup analyses, sensitivity analyses, and generate executable code. Without a review, errors in these outputs (wrong Table 1 format, missing adjusted estimates, individual subgroup p-values instead of interaction p-values, analysis script with placeholder variables, missing imputation sensitivity analysis when >5% missing data) pass uncaught directly into `section-e-original-research.md` Stage 7, which reads `synthesis_report.md` as the authoritative source of all result values.

  2. **No verification of CONSORT/STROBE compliance at the analysis stage:** The Statistician is instructed to omit p-values from Table 1 for RCTs and to include absolute risk reduction + NNT/NNH for RCT binary outcomes. Without a reviewer verifying this, CONSORT violations in Table 1 flow silently into the manuscript and are only caught (if at all) in the Stage 7 Full Review — by which point two Manuscript Writers have already drafted text that may reference incorrect tables.

  3. **No verification that the analysis script is executable:** The Statistician is instructed to write `analysis_scripts/primary_analysis.py` (or `.R`). Without review, a script with missing imports, wrong file paths, or placeholder variables is committed to disk and the user is told "analysis complete" — the script will fail at runtime.

  4. **`review_iteration` variable never initialized:** The existing Step 4 had no initialization of `review_iteration = 0`, meaning the pipeline-orchestrator's resume logic had no iteration counter to track if Stage 6 was interrupted mid-review.

  **Problem — No Stage 6 original_research checklist in `reviewer-protocol.md`:**

  The Stage 6 checklist in `references/reviewer-protocol.md` covered `extracted_data.csv` and `synthesis_report.md` criteria that apply exclusively to literature review types. None of these items are applicable to a primary research analysis: "Every study in the included-studies list has exactly one row in extracted_data.csv" — original research has no included-studies list. An Independent Reviewer dispatched for original_research Stage 6 without a specific checklist would have applied irrelevant review-type criteria to a statistical analysis report, producing a meaningless review.

  **Changes:**

  **`skills/data-extraction/section-original-research.md` — new Step 4 (~45 lines) inserted before the former Step 4 (now Step 5):**
  - Initializes `review_iteration = 0`
  - Dispatches Full Review per `reviewer-protocol.md` on `synthesis_report.md` and analysis scripts
  - Review criteria block organized into 4 areas: Statistical Completeness (Table 1 format, RCT/observational-specific requirements, unadjusted+adjusted estimates, NNT/NNH), Missing Data (pattern description, imputation sensitivity analysis threshold), Subgroup and Sensitivity Analyses (interaction p-values, pre-specified analyses, flagging substantial differences), Analysis Script (file existence, imports, file paths, output directory, stdout summary)
  - REVISE path: builds REVISION REQUIRED table → re-dispatches Statistician for targeted corrections only (not full re-analysis) → increments review_iteration → re-dispatches Full Review with revision history
  - APPROVE path: proceeds to Step 5
  - REJECT / review_iteration ≥ 2 path: triggers full escalation banner per reviewer-protocol.md

  **`references/reviewer-protocol.md` — Stage 6 original_research conditional block (~14 items):**
  - Placed before the existing DTA conditional block, following the same pattern as Stage 3 and Stage 5 project-type-specific blocks
  - Includes disclaimer: standard Extraction Completeness and Synthesis Quality checks do NOT apply for original_research
  - 11 binary pass/fail items with ✗✗ reject-threshold markers on the critical ones:
    - All pre-specified analyses present (✗✗)
    - Table 1 format correct (✗✗)
    - RCT Table 1: no p-values for baseline comparisons (✗✗)
    - Both unadjusted and adjusted estimates with exact p-values (✗✗)
    - RCT binary: absolute risk reduction and NNT/NNH present (✗✗)
    - Observational: all confounders from protocol in adjusted model (✗✗)
    - Missing data handling (>5% missingness → imputation sensitivity analysis) (✗✗)
    - Subgroup analyses: interaction p-values, not individual group p-values (✗✗)
    - Analysis script exists and is syntactically complete (✗✗)
    - No review-type language in the analysis report (✗✗)

  **Why this matters:** Original research (chart reviews, cohort studies, cross-sectional surveys, RCTs) is one of the 8 supported project types and the most common type in a busy dermatology PI lab. Without a review at Stage 6, a PI using the pipeline for original research would receive a `synthesis_report.md` that could contain: (a) Table 1 with p-values (CONSORT violation), (b) only unadjusted estimates for an observational study (STROBE violation), (c) individual within-subgroup p-values instead of interaction p-values (a common but serious methodological error that implies within-subgroup effects are independently significant when they should be compared against each other), (d) an analysis script that fails at runtime. All of these errors would then be baked into the manuscript by Stage 7 writers who read `synthesis_report.md` as the authoritative source.

  **Files changed:**
  - `skills/data-extraction/section-original-research.md` — Step 4 (Full Review) inserted; former Step 4 renamed Step 5
  - `references/reviewer-protocol.md` — Stage 6 original_research conditional block added (~14 items, ~20 lines)
  - `.claude-plugin/plugin.json` — version bumped to 1.0.46

---

## [1.0.45] — 2026-03-14

### Fixed

- **Area 4 (Research Type Coverage) + Area 1 (Autonomy & Self-Correction) — `original_research` Stage 3 in `inclusion-exclusion/SKILL.md`: sparse stub replaced with comprehensive sub-file; reviewer checklist added**

  **Problem:** The `original_research` path in `skills/inclusion-exclusion/SKILL.md` had a 6-bullet Methodologist prompt with no SELF-CHECK, no structured output format, and no reviewer checklist — unlike every other project type in Stage 3. Specifically:

  1. **Sparse Methodologist prompt (no SELF-CHECK):** The prompt instructed the Methodologist to "include" 6 items but gave no field-level specifications, no structured output template, and no SELF-CHECK. The result was that agents would produce inconsistent protocols: missing primary outcome operationalization (time points, measurement instruments), confounders listed without justification, missing sample size calculations, incomplete ethics sections.

  2. **No structured output format:** Unlike every other path in every other skill, the original_research Stage 3 produced no specified `study_protocol.md` markdown template. Agents generated free-form protocols, making them incompatible with downstream stage expectations (Stage 4 reads `study_protocol.md` to design data collection instruments; Stage 5 reads it for eligibility filtering).

  3. **No reviewer checklist in `reviewer-protocol.md`:** The Stage 3 reviewer checklist was PICO-only — entirely inapplicable to a study protocol. An Independent Reviewer dispatched for an original_research Stage 3 output would have checked "PICO component from research_question.md maps to an inclusion criterion" against a study protocol that has no inclusion criteria in the review-type sense. This is a structural mismatch.

  4. **Not consistent with established sub-file pattern:** All other complex Stage 3 paths (qualitative_synthesis inline, DTA via section-dta.md) follow a clear pattern. The original_research path was an outlier holdover from the initial implementation.

  **Changes:**

  **`skills/inclusion-exclusion/section-original-research.md` — new file (~230 lines):**
  - **Step 1 — Methodologist prompt (Part 1–6):** 6-part structured prompt covering:
    - Part 1: Study design selection with named design options and when to choose each; requires design name + justification + key limitation
    - Part 2: Participant eligibility tables (inclusion + exclusion) with required fields: condition with ICD-10 range, age, setting, time period, baseline characteristics; confounding pathway required for each exclusion criterion
    - Part 3: Variables and outcomes — primary outcome (measurement tool + time point), secondary outcomes table, exposure definition with categories, covariate table requiring explicit confounder justification, mediator distinction
    - Part 4: Sample size — one of: formal power calculation (α, power, effect size source, N per group), feasibility estimate, or precision estimate; absence is a FAIL
    - Part 5: Data collection — specific data source names, collection procedure, standardization, quality checks
    - Part 6: Ethics — IRB review type with regulatory basis, HIPAA/privacy approach, consent approach, ClinicalTrials.gov registration
  - **SELF-CHECK (7 criteria):** study design justification specificity, primary outcome operationalization ✗✗, confounder justification completeness ✗✗, mediator-confounder distinction, sample size presence ✗✗, ethics four-item coverage ✗✗, eligibility operationalizability
  - **Step 2 — PI review prompt:** 5 structured review dimensions (clinical feasibility, primary outcome relevance, confounder completeness, design fit, eligibility edge cases)
  - **Step 3 — Output template:** Full `study_protocol.md` markdown template with 6 numbered sections matching the Methodologist prompt structure; tables with required columns for each section
  - **Step 4 — Quick Review loop:** 8 specific original-research review criteria; full REVISE/REJECT loop with REVISION REQUIRED table; `stages.inclusion_exclusion: completed` project.yaml update; completion print statement

  **`skills/inclusion-exclusion/SKILL.md`:**
  - Original_research section (70 lines) replaced with 4-line routing block: "Read `skills/inclusion-exclusion/section-original-research.md` and follow it exactly."
  - Line count: 460 → 393 lines (within 500-line Rule 6 limit; 107 lines of headroom added for future improvements)

  **`references/reviewer-protocol.md` — Stage 3 checklist:**
  - Added `original_research` conditional block (11 items) with a clear disclaimer that standard PICO Alignment, Operationalizability, and Design and Scope checks do NOT apply for original_research:
    - Study design named with aim-specific justification ✗✗
    - Primary outcome operationalized with tool + time point ✗✗
    - Every covariate has explicit confounder justification ✗✗
    - Mediators distinguished from confounders ✗✗
    - Sample size justification present (REJECT if absent) ✗✗
    - Ethics covers all four items ✗✗
    - Eligibility criteria operational for standard data access
    - Protocol aligns with Stage 2 research question

  **Why this matters:** Original research (chart reviews, cohorts, cross-sectional surveys, retrospective studies, RCTs) is one of the 8 supported project types. A PI using the pipeline for original research would previously receive: (a) a study protocol with unnamed or undefined primary outcomes, (b) a covariate list with no confounder justification (potentially introducing adjustment for mediators — a methodological error), (c) no sample size estimate, and (d) a reviewer that checked PICO alignment against a protocol that has no PICO criteria. The Methodologist and Statistician in Stage 4 and Stage 6 read `study_protocol.md` to design the data collection instrument and analysis plan — an incomplete protocol propagates errors through every downstream stage.

  **Files changed:**
  - `skills/inclusion-exclusion/section-original-research.md` — new file (~230 lines)
  - `skills/inclusion-exclusion/SKILL.md` — original_research section replaced with routing (460 → 393 lines)
  - `references/reviewer-protocol.md` — original_research Stage 3 conditional block added (~20 lines)
  - `.claude-plugin/plugin.json` — version bumped to 1.0.45

---

## [1.0.44] — 2026-03-14

### Fixed

- **Area 1 (Autonomy & Self-Correction) + Area 5 (Quality & Proofreading) — `original_research` path in `database-search-build/SKILL.md`: stub Step 3 replaced with real Quick Review; Methodologist prompt expanded with SELF-CHECK**

  **Problem 1 — Broken Quick Review in Step 3:**
  `skills/database-search-build/SKILL.md` Step 3 for `original_research` was titled "Quick Review + Update project.yaml + Manual Handoff" but contained only a `Print:` statement — no reviewer was dispatched. This means:
  - Data collection instruments for original research projects received no automated quality review
  - The `stages.database_search: completed` flag was never set in `project.yaml`, causing the pipeline-orchestrator resume logic to re-run Stage 4 on subsequent invocations
  - Issues with instrument completeness (missing variables, absent codebooks, no quality checks) would silently pass through to Stage 5 data collection — leading to uncollectable or uncleaned variables downstream

  **Problem 2 — Sparse Methodologist prompt with no SELF-CHECK:**
  The Methodologist agent prompt for instrument design asked only for three items ("Data collection form/template", "Variable codebook", "Data quality checks") without:
  - Specifying what fields the collection form must contain (agents often guessed generic fields rather than reading the study protocol variables)
  - Requiring a participant_id field or specifying it must be anonymized
  - Requiring coverage of every variable named in `study_protocol.md`
  - Any SELF-CHECK block (the only agent prompt in the pipeline without one)
  - Mandating that the form be operationalizable by a research coordinator

  **Problem 3 — Missing Stage 4 original_research checklist in reviewer-protocol.md:**
  The Stage 4 checklist in `references/reviewer-protocol.md` only covered search strategy files (`search_strategy.md`). The checklist items (concept blocks, Boolean logic, MeSH terms, database-specific syntax) are completely inapplicable to data collection instruments. An Independent Reviewer dispatched for an original_research Stage 4 output would have had no valid checklist to follow.

  **Changes:**

  **`skills/database-search-build/SKILL.md` — original_research section:**

  1. **Expanded Methodologist prompt (~40 lines added):**
     - Explicitly structured into three deliverables with field-level specifications:
       - Data Collection Form: must include every variable from study_protocol.md, named in snake_case with data_type, valid_values, required flag, section classification, plus participant_id, data_entry_date, and data_entry_by fields
       - Variable Codebook: categorical variables require exhaustive code-label tables; continuous variables require numerical range + units
       - Data Quality Checks: range checks, logic checks (conditional fields), required-field rules, participant_id uniqueness
     - **SELF-CHECK block (6 criteria):** protocol variable coverage (✗✗ trigger), codebook completeness, numerical range specification, logic check requirement, coordinator operationalizability, mandatory administrative fields

  2. **Step 3 replaced — proper Quick Review loop:**
     - Initializes `review_iteration = 0`
     - Dispatches Quick Review per `references/reviewer-protocol.md` with 6 stage-specific review criteria
     - REVISE loop: builds REVISION REQUIRED table → re-dispatches Methodologist → updates instruments on disk → increments counter → re-dispatches reviewer with revision history
     - REJECT/iteration ≥ 2 escalation path (consistent with all other skills)

  3. **Step 4 (new):** Sets `stages.database_search: completed` in project.yaml; then prints the manual handoff message

  **`references/reviewer-protocol.md` — Stage 4 checklist:**
  - Added `original_research` conditional block (6 items) with a clear disclaimer that the standard Stage 4 checklist items (Coverage, Technical Correctness, Reproducibility) do NOT apply for original_research:
    - Protocol variable coverage ✗✗
    - Codebook completeness ✗✗
    - Numerical range specification
    - Quality checks completeness ✗✗
    - Anonymized participant_id ✗✗
    - Coordinator operationalizability

  **Why this matters:** Original research (chart reviews, cohort studies, cross-sectional surveys) is one of the 8 supported project types. Without a proper Stage 4, any PI using the pipeline for original research would get data collection instruments that: (a) might be missing study protocol variables (undetected by any review), (b) had no quality checks to prevent bad data entry, and (c) left the pipeline stuck re-running Stage 4 on resume because `stages.database_search` was never set to `completed`. The Statistician and manuscript writing stages at the end of the pipeline rely on a clean, complete dataset — one that can only exist if the collection instruments are correct from the start.

  **Files changed:**
  - `skills/database-search-build/SKILL.md` — original_research section expanded: Methodologist prompt ~50 → ~110 lines (with SELF-CHECK); Step 3 stub replaced with Quick Review loop + retry logic; Step 4 added for project.yaml update + manual handoff. Total: 196 lines → 316 lines.
  - `references/reviewer-protocol.md` — original_research Stage 4 conditional block added (~18 lines). Note: reviewer-protocol.md is a reference file; the 500-line skill cap does not apply.
  - `.claude-plugin/plugin.json` — version bumped to 1.0.44

---

## [1.0.42] — 2026-03-14

### Refactored

- **Rule 6 compliance — `skills/abstract-screening/SKILL.md` split into sub-files (754 lines → 320 lines)**

  **Problem:** `skills/abstract-screening/SKILL.md` had grown to 754 lines — 50% over the 500-line limit specified in Rule 6 — due to sequential additions of project-type-specific screening processes across multiple prior runs (rapid_review, original_research, diagnostic_test_accuracy, qualitative_synthesis). A single 754-line file is harder to maintain, harder for agents to parse efficiently, and violates the plugin's own structural rule.

  **Solution:** Applied the same sub-file split pattern used in `skills/manuscript-writing/` (which uses `section-a-qualitative.md`, `section-b-quantitative.md`, etc.).

  **Files created:**
  - `skills/abstract-screening/section-rapid-review.md` (125 lines) — rapid_review single-screener process with 10% verification audit, Quick Review, deviation logging.
  - `skills/abstract-screening/section-original-research.md` (207 lines) — original_research data cleaning process: raw data profiling, eligibility filtering, missing data audit, duplicate detection, range checks, variable standardization, Quick Review.
  - `skills/abstract-screening/section-dta.md` (112 lines) — diagnostic_test_accuracy screening with PICOS decision rules, `threshold_reported` capture field, case-control/partial-verification exclusion enforcement.
  - `skills/abstract-screening/section-qualitative-synthesis.md` (110 lines) — qualitative_synthesis PICo decision framework, `qualitative_method` capture field, mixed-methods UNCERTAIN handling.

  **Files modified:**
  - `skills/abstract-screening/SKILL.md`: 754 lines → 320 lines. Core content retained (parsing, dedup, edge cases, batching). Added Project-Type Routing table (same pattern as manuscript-writing SKILL.md) that directs agents to the correct sub-file for each project type. Process — Review Types (systematic_review, scoping_review, meta_analysis) and Process — case_report remain inline (both are short enough to stay in the main file).
  - `.claude-plugin/plugin.json` — bumped version to 1.0.42

  **No functionality was removed.** All process instructions are identical to what was in v1.0.41 — this is a structural refactor only.

---

## [1.0.41] — 2026-03-14

### Added

- **Area 4 (Research Type Coverage) — `original_research` data cleaning path in `skills/abstract-screening/SKILL.md`: stub replaced with full implementation**

  **Problem:** The `original_research` process in `skills/abstract-screening/SKILL.md` was a 2-line stub: "Data Extractors clean and code raw data files per the collection instruments. Single pass, no triple redundancy." This contained:
  - No instructions for locating or reading raw data files
  - No agent dispatch prompt
  - No eligibility filtering logic
  - No data cleaning steps (missing data, deduplication, range checks, variable standardization)
  - No output file specifications
  - No review step
  - No summary or handoff message

  For original research, Stage 5 is the data cleaning stage — it is not abstract screening. Raw collected data (REDCap .csv exports, survey spreadsheets, chart review files, EHR extracts) must be cleaned, eligibility-filtered, and validated before the Statistician can analyze them in Stage 6. Without this stage, Stage 6's Statistician agent would receive unvalidated raw data, producing unreliable analyses with no audit trail.

  **Changes to `skills/abstract-screening/SKILL.md`:**

  1. **Prerequisites check:** Instructions to look for raw data files in `06_data_extraction/raw_data/` (primary) or `04_database_search/abstracts/` (fallback). Clear error message if no data file is found.

  2. **Step 1 — Raw data profiling:** Agent reads the raw file and `study_protocol.md`, then writes `05_screening/data_profile.md` with: row/column counts, expected vs. present variables (flagging missing or extra columns), per-variable null counts and type classification.

  3. **Step 2 — Data Extractor dispatch:** Full structured agent prompt covering six cleaning tasks:
     - Eligibility filtering: applies participant-level inclusion/exclusion criteria from `study_protocol.md`; writes `excluded_participants.csv` with criterion and reason per exclusion.
     - Missing data audit: per-variable null counts, >20% missingness flagged as HIGH_MISSINGNESS; no imputation (Statistician decision in Stage 6).
     - Duplicate detection: exact duplicates removed and logged; 70–84% similarity flagged as POSSIBLE_DUPLICATE (kept for human review).
     - Range and consistency checks: out-of-range values and logical inconsistencies flagged as OUT_OF_RANGE (not removed).
     - Variable standardization: recoding to protocol coding scheme, whitespace trimming, column renames.
     - Output: `05_screening/cleaned_data.csv` + `05_screening/cleaning_log.md`.
     - SELF-CHECK (6 criteria): row count arithmetic, exclusion reason specificity, flag specificity, no-imputation check, protocol alignment, cleaning log completeness.

  4. **Step 3 — Quick Review:** Reviewer checks row-count arithmetic, criterion citation quality, flag presence, absence of imputation (REJECT trigger), and cleaning log completeness.

  5. **Step 4 — Summary + handoff message:** CONSORT/STROBE-compatible participant flow counts (enrolled → eligible → analyzed), data quality flag summary, output file listing, POSSIBLE_DUPLICATE advisory, and instruction to run `/data-extraction`.

  **Why this matters:** Original research (chart reviews, cohorts, cross-sectional surveys, retrospective studies) is one of the 8 supported project types. Without a working data cleaning stage, any PI running an original research project through the pipeline would get to Stage 6 with unvalidated raw data — potentially including ineligible participants, duplicates, and out-of-range values — leading to incorrect statistical analysis and a non-reproducible audit trail. The fix gives original research a complete, structured data preparation stage that produces a cleaned dataset and exclusion log compatible with CONSORT and STROBE participant flow diagram reporting.

  **Files changed:**
  - `skills/abstract-screening/SKILL.md` — `## Process — original_research` section expanded from 2 lines to ~130 lines
  - `plugin.json` — bumped version to 1.0.41

---

## [1.0.40] — 2026-03-14

### Fixed

- **Area 2 (Statistical Capabilities) + Area 1 (Autonomy & Self-Correction) — `original_research` Step 3 in `data-extraction/SKILL.md`: expanded stub into a full Statistician agent dispatch**

  **Problem:** The `original_research` path Step 3 (Statistical Analysis) in `skills/data-extraction/SKILL.md` was a brief bullet-list stub with no actual agent prompt. Unlike every other data-extraction dispatch (review types, qualitative synthesis, DTA, rapid review), the `original_research` path had no structured Statistician prompt, no SELF-CHECK block, and no output specification. The instructions just said "Dispatch Statistician to analyze per the pre-specified study protocol" followed by a list of what to produce — but without an actual prompt, the Statistician agent would have no guidance on:
  - How to produce Table 1 (CONSORT vs. STROBE differences — e.g., no p-values in RCT baseline table)
  - How to handle missing data (complete-case vs. multiple imputation threshold)
  - What statistical reporting format to follow (exact p-values, both unadjusted and adjusted estimates)
  - What the executable analysis script must contain (imports, file paths, output directories, stdout summary)
  - How to self-verify output completeness before writing to disk
  The result was that the agent would produce inconsistent, incomplete analysis reports for original research projects — missing adjusted estimates, presenting "p < 0.05" instead of exact values, producing scripts that don't run cleanly, or skipping pre-specified subgroup analyses silently.

  **Changes:**

  `skills/data-extraction/SKILL.md` — Step 3 of `Process — original_research`:
  - Replaced the 9-line bullet-list stub with a full structured Statistician agent prompt (~70 lines)
  - Prompt covers: Table 1 (with CONSORT vs. observational distinction), missing data summary with imputation trigger at >5% threshold, primary and secondary analyses, subgroup and sensitivity analyses, executable Python/R script requirements (pandas/statsmodels/lifelines for Python; tidyverse/survival/ggplot2 for R), and a SELF-CHECK block with 8 explicit pass/fail criteria
  - SELF-CHECK verifies: N in Table 1 matches cleaned_data.csv, unadjusted AND adjusted estimates both present for observational designs, exact p-values (no "p < 0.05" shortcuts), no baseline p-values in RCT Table 1, script runs without manual editing, all pre-specified subgroup/sensitivity analyses addressed
  - Output instruction clarified: synthesis report to `synthesis_report.md` AND script to `analysis_scripts/primary_analysis.py` (or `.R`)
  - File: `skills/data-extraction/SKILL.md` — line count 603 → 676 (exceeds 500-line guideline; future refactor should move `qualitative_synthesis` process to a sub-file, as noted in v1.0.26)

## [1.0.39] — 2026-03-14

### Added

- **Area 4 (Research Type Coverage) — `original_research` manuscript-writing path separated from review-types template:**

  **Problem:** `skills/manuscript-writing/SKILL.md` routed `original_research` projects to `section-b-quantitative.md` — the template designed for systematic reviews, scoping reviews, meta-analyses, and rapid reviews. This caused agents writing an original research manuscript to:
  1. Include a PRISMA flow diagram section (wrong — primary research uses CONSORT or STROBE participant flow, not a literature screening flow)
  2. Tell Writer B to "Present search/screening flow (PRISMA numbers)" and "Include all tables and figures referenced in synthesis report" — review-centric instructions that produce structurally incorrect Results sections
  3. Tell Writer A to describe "search strategy, databases, date range" and "screening process and criteria" in the Methods section — these do not exist in original primary research
  4. Have the Statistician prepare "GRADE certainty assessment" and "forest plots" — inappropriate for primary data analysis
  5. Have the self-check validate PRISMA compliance while the original_research path should validate CONSORT/STROBE compliance
  The result was that any agent writing an original research manuscript would produce a document with systematic-review structure and language ("included studies", "search strategy", "PRISMA flow") embedded in a primary research context — an unfixable structural error requiring a complete rewrite.

  **Root cause:** When the routing table was first written, `original_research` was grouped with review types as "all others". This was correct as a temporary placeholder but was never replaced with a dedicated path — even though original research has a completely different document structure (patient flow, Table 1, primary analysis results, STROBE/CONSORT compliance).

  **Changes:**

  1. **Created `skills/manuscript-writing/section-e-original-research.md`** (~240 lines) with a full original-research-specific manuscript process:
     - **E1 (Fetch guidelines + determine reporting standard):** Infers CONSORT vs. STROBE from `project.yaml`; falls back to scanning `study_protocol.md` for randomization language.
     - **E2 (Parallel dispatch — Writer A, Writer B, Statistician):** Three original-research-specific agent prompts:
       - Writer A (Introduction + Methods): References `study_protocol.md` (not `criteria.md` or search strategy); includes study design, participant eligibility, randomization (RCT) or matching (observational), variable definitions, bias section, sample size calculation, pre-specified statistical methods, and ethics/IRB citation. Explicitly instructs agents not to include a database search strategy section.
       - Writer B (Results + Discussion): References `synthesis_report.md` and `cleaned_data.csv` (not `screened_results.csv` or `extracted_data.csv`); instructs agents to include CONSORT/STROBE participant flow, Table 1 (without p-values for RCT baseline comparisons), unadjusted + adjusted primary outcome results, subgroup interaction p-values, and missing data description. Explicitly instructs agents not to reference PRISMA or "included studies".
       - Statistician: Prepares Table 1, primary results table, CONSORT/STROBE flow text, figure code (Kaplan-Meier, subgroup forest plot), and statistical text snippets — not GRADE/SoF tables or meta-analytic forest plots.
     - **E3 (Self-check):** Includes a "No PRISMA flow, no systematic search strategy section — this is primary research ✗✗ if present" check as a structural blocker.
     - **E4–E6:** Write, Full Review, update project.yaml — standard pattern.

  2. **Updated routing table in `skills/manuscript-writing/SKILL.md`:** Added `original_research → section-e-original-research.md` as an explicit row; removed `original_research` from the "all others" catch-all row.

  3. **Added `original_research` conditional block to Stage 7 reviewer checklist in `references/reviewer-protocol.md`** (12 check items):
     - PRISMA-in-original-research detection (REJECT trigger) ✗✗
     - CONSORT or STROBE compliance ✗✗
     - Study registration and IRB citation ✗✗
     - Table 1 structure (no p-values for RCT baseline comparisons) ✗✗
     - Unadjusted + adjusted estimates both present ✗✗
     - Interaction p-values for subgroup analyses ✗✗
     - Associative language only in observational discussion ✗✗
     - No review-type language in Results ✗✗

  **Why this matters:** Original research (chart reviews, prospective cohorts, cross-sectional surveys, retrospective cohorts) is one of the 8 supported project types. Any PI using the pipeline for an original study would receive a manuscript with a PRISMA flow diagram and search strategy — structurally incompatible with any clinical journal submission. The fix gives original research a proper, differentiated manuscript path that matches its distinct structure and reporting guidelines.

  **Files changed:**
  - `skills/manuscript-writing/section-e-original-research.md` — new file (~240 lines)
  - `skills/manuscript-writing/SKILL.md` — routing table updated (1 row added, 1 row modified)
  - `references/reviewer-protocol.md` — original_research conditional block added to Stage 7 checklist (~15 lines)
  - `plugin.json` — bumped version to 1.0.39

---

## [1.0.38] — 2026-03-14

### Fixed

- **Area 1 (Autonomy & Self-Correction) — `original_research` path in `inclusion-exclusion` skill: empty Step 4 filled in; Review Types step numbering gap fixed:**

  **Problem 1 — Empty Step 4 in original_research path:** `skills/inclusion-exclusion/SKILL.md` had an `## Process — original_research` section with Steps 1–3 followed by an entirely empty `### Step 4: Quick Review + Update project.yaml` placeholder (header only — zero instruction content). An agent following this path would reach Step 4 and have no guidance: no reviewer dispatch instruction, no review criteria, no REVISE/REJECT retry loop, and no instruction to set `stages.inclusion_exclusion: completed` in `project.yaml`. In practice this means:
  - A study protocol with an infeasible sample size, an undefined primary outcome, or missing IRB considerations would pass to Stage 4 unreviewed
  - The pipeline-orchestrator would not mark the stage as completed (no `stages.inclusion_exclusion: completed` instruction), causing the resume logic to re-run Stage 3 on subsequent invocations
  - The same category of stub problem fixed for rapid_review in 1.0.34 and 1.0.35 — this was the last remaining empty stub across the pipeline

  **Fix:** Replaced the empty placeholder with a complete Step 4:
  - Quick Review dispatch per `references/reviewer-protocol.md`
  - Eight `original_research`-specific review criteria: study design justification, operational eligibility criteria, primary outcome definition (instrument + timing), secondary outcomes and confounders listed, sample size consideration (formal or feasibility-based), IRB/ethics noted, feasibility check (not over/under-inclusive), protocol–research question alignment
  - Full REVISE retry loop: REVISION REQUIRED table → re-dispatch Methodologist for targeted fixes → increment counter → re-dispatch reviewer with revision history
  - REJECT/`review_iteration ≥ 2` escalation path
  - `stages.inclusion_exclusion: completed` instruction

  **Problem 2 — Step numbering gap in Review Types path:** The Review Types process had steps 1–5 followed by `### Step 7: Quick Review + Update project.yaml` — skipping Step 6 entirely. This gap had no functional impact (agents would execute Step 7 regardless of its label) but was confusing: an agent seeing "Step 7" after "Step 5" might reason "I must have missed Step 6" and attempt to invent or repeat a step. The gap appears to have occurred when a prior edit renumbered PROSPERO generation to Step 5 without renumbering the subsequent Quick Review step.

  **Fix:** Renamed `### Step 7` to `### Step 6` in the Review Types path. The step numbering now runs consecutively: 1 (Methodologist) → 2 (Librarian) → 3 (PI) → 4 (Write Output) → 5 (PROSPERO, conditional) → 6 (Quick Review + Update). The qualitative_synthesis path already correctly labels its review step as Step 5 (it has no PROSPERO step) — no change needed there.

  **Files changed:**
  - `skills/inclusion-exclusion/SKILL.md` — empty Step 4 body filled in (~30 lines added); `### Step 7` renamed to `### Step 6` in Review Types path
  - `plugin.json` — bumped version to 1.0.38

---

## [1.0.37] — 2026-03-14

### Fixed

- **Area 5 (Quality & Proofreading) — Stage 2 reviewer checklist: framework mapping bug fixed + conditional blocks added for `qualitative_synthesis` and `diagnostic_test_accuracy`:**

  **Problem 1 — Framework mapping bug:** The Stage 2 PICO/PEO/PCC completeness checklist stated `(PICO for intervention/meta-analysis, PEO for scoping, PCC for qualitative)`. Both mappings for non-interventional types were wrong:
  - "PEO for scoping" is incorrect — scoping reviews use **PCC** (Population, Concept, Context), not PEO. PEO is for exposure-based/etiological studies.
  - "PCC for qualitative" is incorrect — qualitative synthesis uses **PICo** (Population, phenomenon of Interest, Context), not PCC.
  A reviewer following this checklist would incorrectly APPROVE a scoping review using PEO (wrong framework) and incorrectly REJECT one correctly using PCC. Similarly, it would misclassify the framework for qualitative synthesis. This affected every scoping review and qualitative synthesis run since the checklist was written.

  **Fix:** Updated the framework type check to correctly enumerate all project-type-specific frameworks:
  - `PICO` for `systematic_review`, `meta_analysis`, `rapid_review`
  - `PCC` for `scoping_review`
  - `PICo` for `qualitative_synthesis`
  - `PICOS` for `diagnostic_test_accuracy`

  **Problem 2 — Missing conditional blocks:** The Stage 2 checklist had no conditional checks for `qualitative_synthesis` or `diagnostic_test_accuracy` despite both being fully supported in the research-question skill since v1.0.30. The Stage 5 and Stage 6 checklists both have project-type-specific conditional blocks — Stage 2 was inconsistently missing them. A reviewer evaluating a qualitative_synthesis Stage 2 output had no way to verify PICo completeness, phenomenon specificity, or synthesis method selection. A reviewer evaluating a DTA Stage 2 output had no way to check PICOS component completeness, index test specificity, reference standard justification, or spectrum bias documentation.

  **Changes to `references/reviewer-protocol.md` Stage 2 checklist:**
  1. **`Qualitative Synthesis` conditional block (4 items):**
     - PICo table has all three components (P, I, Co) ✗✗
     - No PICO-style fields (no Intervention, Comparator, or quantitative Outcome) ✗✗
     - phenomenon of Interest is operationally specific ✗✗
     - Synthesis approach (thematic/meta-ethnography/framework/meta-aggregation) recommended with rationale

  2. **`Diagnostic Test Accuracy` conditional block (6 items):**
     - PICOS table has all five components with required content ✗✗
     - Index test named specifically with protocol/threshold ✗✗
     - Reference standard named and validity justified ✗✗
     - Spectrum bias documented in Population component ✗✗
     - No treatment-effect language in any PICOS component ✗✗
     - Partial verification risk addressed

  **Files changed:**
  - `references/reviewer-protocol.md` — framework type check updated (1 line); two conditional blocks added (~20 lines total after Stage 2 Downstream Compatibility section)
  - `plugin.json` — bumped version to 1.0.37

---

## [1.0.36] — 2026-03-14

### Added / Fixed

- **Area 4 (Research Type Coverage) — `database-search-build` now handles `qualitative_synthesis` and `diagnostic_test_accuracy` correctly:**

  **Problem:** `skills/database-search-build/SKILL.md` had a single generic "Process — Review Types" path that all project types fell through. For `qualitative_synthesis`, this produced searches that (a) omitted CINAHL and PsycINFO — the primary qualitative databases, (b) used a PICO concept structure instead of PICo (no Phenomenon of Interest block), and (c) potentially applied RCT or intervention-study filters that exclude qualitative studies entirely. For `diagnostic_test_accuracy`, searches (a) omitted the DTA methodological filter block (sensitivity/specificity MeSH terms, ROC curve MeSH) that is required to retrieve accuracy studies, and (b) potentially applied RCT filters that exclude cross-sectional diagnostic accuracy studies. Both cases would produce search strategies that systematically miss the most relevant literature for those project types.

  **Changes to `skills/database-search-build/SKILL.md`:**
  1. Added a **PROJECT-TYPE DATABASE AND FILTER ADAPTATIONS** block inside the Librarian agent prompt, covering:
     - **`qualitative_synthesis`:** CINAHL (mandatory) + PsycINFO + AMED added to database list; PICo concept structure (Population + Phenomenon of Interest blocks only, no Comparator/Outcome); per-database qualitative study design filter (`"qualitative research"[MeSH]`, focus groups, thematic analysis, phenomenology, ethnography, grounded theory — with correct syntax for PubMed, Ovid, CINAHL, PsycINFO); explicit prohibition on RCT/controlled trial filters.
     - **`diagnostic_test_accuracy`:** PICOS concept structure (Population + Index Test blocks, no Comparator block); per-database DTA methodological filter (`"sensitivity and specificity"[MeSH]`, ROC Curve, likelihood ratios, AUROC — with correct syntax for PubMed, Ovid, Embase); explicit prohibition on RCT/systematic review/intervention-study filters.
  2. Extended the **SELF-CHECK** with items 8 and 9:
     - Item 8: Verifies CINAHL presence + qualitative filter + no RCT filter for `qualitative_synthesis`
     - Item 9: Verifies DTA methodological filter + no RCT/intervention filter for `diagnostic_test_accuracy`

## [1.0.35] — 2026-03-14

### Added / Fixed

- **Area 4 (Research Type Coverage) — `rapid_review` data extraction path expanded from stub to full implementation:**

  **Problem:** `skills/data-extraction/SKILL.md` had a 3-line stub for `## Process — rapid_review`: "Same as review types but abbreviated: single extractor (no partitioning), descriptive tables only in synthesis, no meta-analysis." This gave agents no concrete instructions on how to run the single-extractor workflow, what abbreviations to apply to the risk-of-bias appraisal, which review tier to use, or how to document the methodological shortcuts for PRISMA reporting. An agent following this stub would either fall back to the full 3-extractor workflow (incorrect) or halt with no guidance. This is the same category of problem fixed in 1.0.34 for the rapid_review abstract-screening path.

  **Changes:**
  1. Created `skills/data-extraction/section-rapid-review.md` (~130 lines) with the full rapid_review data extraction process:
     - **Step 1 (Abbreviated Template):** Statistician designs a minimal extraction template without meta-analytic input columns; abbreviated risk-of-bias appraisal (first 2 RoB 2 domains for RCTs; NOS selection + comparability only for observational) — both flagged as rapid-review abbreviations. Clinical coding columns included if `clinical_coding: true`.
     - **Step 2 (Single Extractor):** Single Data Extractor dispatched with a rapid-review note prepended explaining single-extractor mode. Batching at 10 papers per invocation, same as standard path.
     - **Step 3 (Merge):** Trivial merge for single-extractor output.
     - **Step 4 (Quick Review):** Quick Review (not Full Review) per project-types.md rapid review adaptation — single reviewer, max 2 iterations, 20% random sample audit. Retry logic for REVISE verdicts with REVISION REQUIRED block passed back to the Data Extractor.
     - **Step 5 (Descriptive Synthesis):** Statistician produces study characteristics table, narrative synthesis, and abbreviated risk-of-bias summary — explicitly prohibited from pooling or generating GRADE. SELF-CHECK enforces no pooled estimates and confirms all studies accounted for.
     - **Step 6 (Deviation Log):** Appends 3 documented shortcuts to `07_manuscript/rapid_review_deviations.md` (single extractor, abbreviated RoB, no pooling/GRADE) per PRISMA 2020 item 27; prints a completion summary with shortcut warnings.
  2. Replaced the 3-line stub in `skills/data-extraction/SKILL.md` with a routing instruction to the new sub-file (keeping SKILL.md from growing further — it was already at 603 lines).

  **Why this matters:** Rapid reviews are a common project type for time-constrained clinical researchers (4–8 week turnaround). Without a concrete workflow, agents would either silently apply the full 3-extractor process (defeating the purpose of a rapid review) or fail to proceed. The new path gives agents exact instructions for every step while faithfully implementing the methodological shortcuts described in `references/project-types.md`.

  **Files changed:**
  - `skills/data-extraction/section-rapid-review.md` — new file (~130 lines)
  - `skills/data-extraction/SKILL.md` — 3-line stub replaced with 1-line routing instruction
  - `plugin.json` — bumped version to 1.0.35

---

## [1.0.34] — 2026-03-14

### Added / Fixed

- **Area 4 (Research Type Coverage) — `rapid_review` abstract screening path expanded from stub to full implementation:**

  **Problem:** `skills/abstract-screening/SKILL.md` had a 3-line stub for `## Process — rapid_review`: "Same as above but with only 1 Data Extractor (no redundancy, no reconciliation step). Skip directly to review." This gave agents no concrete instructions on how to run the single-screener workflow, what to substitute for the 3-way reconciliation step, which review tier to apply, or how to document the methodological shortcut for PRISMA reporting. Any agent trying to follow this stub would fall back to the full 3-extractor workflow or halt with no guidance.

  **Changes made to `skills/abstract-screening/SKILL.md`:**
  1. **Step 1 (Single Extractor):** Clear instruction to dispatch exactly one Data Extractor with a note prepended to the prompt explaining single-screener mode; same criteria/PICO prompt structure as the full review path.
  2. **Step 2 (10% Verification Audit):** New step — Methodologist audits a random 10% sample (min 20 records). If ≥10% flagged, triggers a re-screen. Implements the "single-screener with verification sample" approach specified in `references/project-types.md`. Provides full agent prompt for the audit.
  3. **Step 3 (Write Output):** Instructs the agent to annotate `screened_results.csv` with a comment row documenting the rapid review shortcut and audit result.
  4. **Step 4 (Quick Review):** Explicitly specifies Quick Review (not Full Review) per `project-types.md` rapid review adaptation — with the single reviewer, max 2 iterations, and rapid-review-specific review criteria.
  5. **Step 5 (Update + Handoff):** Instructs the agent to append a deviation record to `07_manuscript/rapid_review_deviations.md` (a new file for tracking all rapid review shortcuts across stages). The deviation record includes standard practice, the shortcut taken, potential impact, and audit result — ready to drop into the Limitations section. Prints a completion summary with PRISMA flow counts and a shortcut warning referencing PRISMA 2020 item 27.

## [1.0.33] — 2026-03-14

### Added / Fixed

- **Area 4 (Research Type Coverage) — `diagnostic_test_accuracy` path in `inclusion-exclusion` skill:**

  **Problem:** `skills/inclusion-exclusion/SKILL.md` had no `## Process — diagnostic_test_accuracy` section. A DTA project would fall through to the generic `systematic_review` path, producing a PICO-structured criteria table that: (a) used "Intervention/Exposure" and "Comparator" rows instead of "Index Test" and "Reference Standard"; (b) contained no spectrum-bias or verification-bias exclusion criteria; (c) did not include DTA methodological search filter guidance for the Librarian. This would produce methodologically incorrect eligibility criteria for STARD/QUADAS-2–compliant DTA reviews.

  **Changes:**
  1. Added `## Process — diagnostic_test_accuracy` routing stub to `skills/inclusion-exclusion/SKILL.md` that delegates to a new sub-file.
  2. Created `skills/inclusion-exclusion/section-dta.md` with a full DTA-specific process: Methodologist produces PICOS-structured criteria (Population, Index Test, Reference Standard, Outcomes, Study Design); mandatory exclusion criteria for case-control designs (spectrum bias) and partial verification studies (verification bias); Librarian validates DTA methodological filter blocks per database; PI reviews clinical applicability; output template uses PICOS table structure; Quick Review checks all 5 PICOS components and both bias-specific exclusions.

- **Area 4 (Research Type Coverage) — scoping review PCC framework in `inclusion-exclusion` skill:**

  **Problem:** The `## Process — Review Types` path in `skills/inclusion-exclusion/SKILL.md` bundled `scoping_review` with `systematic_review` and used a PICO output template (Population, Intervention/Exposure, Comparator, Outcomes). Scoping reviews use the **PCC framework** (Population, Concept, Context) — there is no Intervention, Comparator, or quantitative Outcome criterion in a scoping review. This would produce structurally incorrect eligibility criteria for PRISMA-ScR–compliant scoping reviews.

  **Changes made to `skills/inclusion-exclusion/SKILL.md`:**
  1. Added an explicit conditional instruction at the top of the Methodologist prompt to use PCC framing for `scoping_review` — replacing items 2–4 (Intervention/Exposure, Comparator, Outcomes) with CONCEPT and CONTEXT, and explicitly stating that Outcomes criteria are omitted for scoping reviews.
  2. Added a separate PCC output template for scoping reviews (rows: Population, Concept, Context, Study design, Publication date, Language) alongside the existing PICO template, with clear routing instructions by project type.

---

## [1.0.32] — 2026-03-14

### Fixed

- **Area 6 (Robustness) — `.nbib` format parsing in abstract-screening:** Added explicit parsing rules for `.nbib` files (NLM/PubMed flat-file format — the default PubMed "Citation manager" export) and updated `.txt` file handling to detect PubMed-format text exports.

  **Problem:** `skills/abstract-screening/SKILL.md` listed only three supported abstract file formats: `.csv`, `.ris`, and `.txt` (treated as single-abstract files). However, `references/getting-started.md` and the pipeline-orchestrator Manual Handoff 1 both explicitly instruct users to export PubMed results as `.nbib` files (`Send to → Citation manager → Format: PubMed → Create file`). A user following these instructions would place `.nbib` files in `04_database_search/abstracts/` — but the screening agent had no instructions for parsing this format and would either skip the files or fail silently. This would cause incorrect PRISMA counts (zero records from PubMed) and empty screening results despite the user having executed the search correctly.

  **Changes made to `skills/abstract-screening/SKILL.md`:**
  1. Added a dedicated `.nbib` parsing block with field-by-field extraction rules for the NLM flat-file format: `PMID-` → id, `TI  -` → title (with multi-line continuation support), `AU  -` → authors (multiple lines joined with "; "), `AB  -` → abstract_text (continuation lines appended), `JT  -` / `TA  -` → journal (with fallback), `DP  -` → year (extract first 4-digit number from date string), `AID -` / `LID -` with `[doi]` suffix → doi (strip suffix, prefer AID over LID).
  2. Updated `.txt` file handling: agent now scans the file for a `PMID- ` line first; if found, parses it as NLM flat-file (same rules as `.nbib`); otherwise falls back to the legacy "single abstract per file" behaviour. This handles the PubMed "Save → Format: PubMed → Create file" export that produces a `.txt` file with identical NLM formatting.

  **Why this matters:** PubMed is the most-used database for clinical research. Virtually every user will place PubMed `.nbib` exports in the abstracts folder. Without this fix, the screening stage would silently fail to parse the most common input file type.

## [1.0.31] — 2026-03-14

### Added

- **Area 5 (Quality & Proofreading) — DTA-specific Stage 5 reviewer checklist block:** Added a conditional `Diagnostic Test Accuracy` block to the Stage 5 reviewer checklist in `references/reviewer-protocol.md`.

  **Problem:** The Stage 5 reviewer checklist had an unconditional Deduplication block and a conditional Qualitative Synthesis block (both added in 1.0.29), but no conditional block for `diagnostic_test_accuracy`. Yet DTA abstract screening (SKILL.md) has several requirements that generic Stage 5 checks do not cover:
  1. The `threshold_reported` field is required for every INCLUDE and UNCERTAIN row — a reviewer had no checklist item to verify this was populated.
  2. Case-control designs must be EXCLUDED due to spectrum bias — the generic "EXCLUDE decisions cite a specific criterion" check would catch mislabelled exclusion reasons, but a reviewer evaluating a DTA output had no explicit instruction to verify that no case-control study appeared as INCLUDE.
  3. Partial verification studies (reference standard applied only to test-positive cases) must be EXCLUDED if listed in criteria.md — reviewers had no specific check for this verification bias criterion.
  4. DTA UNCERTAIN decisions have three specific valid triggers (reference standard application ambiguity, spectrum ambiguity, threshold absence) — reviewers had no way to verify UNCERTAIN was not being misused for borderline population/outcome fit.
  5. Reconciliation notes for design CONFLICTs should explicitly address spectrum bias — the generic conflict resolution check did not specify DTA-specific content.
  6. Studies where the abstract clearly shows no reference standard was applied should not be INCLUDE — this was not captured by generic criteria.

  **Solution:** Added to the Stage 5 checklist in `reviewer-protocol.md`, after the Qualitative Synthesis block:

  **Diagnostic Test Accuracy conditional block (6 items):**
  - Every INCLUDE and UNCERTAIN row has a non-blank `threshold_reported` field ✗✗ (value or "NR" — never blank)
  - No case-control design appears as INCLUDE without documented exception ✗✗
  - Partial verification studies are EXCLUDE if stated as an exclusion criterion ✗✗
  - UNCERTAIN decisions cite one of the three valid DTA UNCERTAIN triggers (reference standard application ambiguity, spectrum ambiguity, or threshold absence) — not borderline population/outcome fit
  - Reconciliation notes for design CONFLICTs address spectrum bias and partial verification explicitly
  - No INCLUDE decision for a study where the abstract shows no reference standard was applied

  **Files changed:**
  - `references/reviewer-protocol.md` — DTA conditional block added after Qualitative Synthesis block; Stage 5 checklist grows by ~10 lines
  - `plugin.json` — bumped version to 1.0.31

## [1.0.30] — 2026-03-14

### Added

- **Area 4 (Research Type Coverage) — `qualitative_synthesis` and `diagnostic_test_accuracy` processes in research-question skill:** Added two new project-type–specific process sections to `skills/research-question/SKILL.md`, and updated the routing table to direct these project types explicitly instead of silently falling through to the Review Types catch-all.

  **Problem:** `research-question/SKILL.md` had process sections for `case_report`, `rapid_review`, and `original_research`, but `qualitative_synthesis` and `diagnostic_test_accuracy` had no dedicated routes. Both fell through to the "All others → Review Types" path, which dispatches the PI agent with PICO framing. This was wrong for both types:
  - `qualitative_synthesis` requires PICo (Population, phenomenon of Interest, Context) — there is no Intervention, Comparator, or quantitative Outcome. A PICO-prompted PI would generate a question with incorrect structure, and the Methodologist would not evaluate for synthesis approach fit.
  - `diagnostic_test_accuracy` requires PICOS (Population, Index test, Comparator/reference standard, Outcomes as sensitivity/specificity/AUC, Study design). A PICO-prompted PI would omit the spectrum bias characterisation, threshold analysis question, and partial verification flag — all methodologically critical for DTA reviews.

  This was the last stage in the pipeline where these two project types lacked dedicated handling. Both were already fully supported in Stages 3–7 (inclusion-exclusion: qualitative_synthesis PICo section added in 1.0.26, DTA path added earlier; abstract-screening, data-extraction, manuscript-writing all have dedicated DTA and qualitative paths).

  **Solution:** Added to `skills/research-question/SKILL.md`:

  **Routing table update:** Added explicit routes for `qualitative_synthesis` and `diagnostic_test_accuracy` before the catch-all "All others" entry.

  **`## Process — qualitative_synthesis` section (~45 lines):** Follows the same step structure as Review Types but specifies PICo modifications:
  - PI agent generates 2-3 PICo candidate questions (P / phenomenon of Interest / Co); explicitly prohibits PICO components; SELF-CHECK enforces no PICO language, phenomenon specificity, context operationalizability.
  - Methodologist evaluates synthesis approach fit (thematic vs. meta-ethnography vs. framework vs. meta-aggregation) and recommends the appropriate method.
  - User confirmation presents PICo breakdown + synthesis approach before writing to disk.
  - Output format uses PICo table (not PICO).
  - Review criteria verify PICo completeness, absence of PICO language, synthesis appropriateness, method fit.
  - project.yaml update maps pico fields to PICo components (`intervention_exposure` → phenomenon of Interest; `comparator` → Context; `outcomes` → "N/A — qualitative synthesis").

  **`## Process — Diagnostic Test Accuracy` section (~50 lines):** Follows the same step structure but specifies PICOS modifications:
  - PI agent frames a PICOS question with all 5 components; SELF-CHECK enforces specific index test name, proportion-based outcome reporting, and explicit spectrum bias + partial verification flagging.
  - Methodologist evaluates reference standard validity, spectrum bias control in population definition, synthesis method fit (bivariate/Reitsma vs. HSROC), and likely QUADAS-2 domain ratings.
  - User confirmation presents PICOS breakdown + planned synthesis model + spectrum bias note.
  - Output format uses PICOS table; Framework section labels index test, reference standard, accuracy outcomes, study design.
  - Review criteria verify all 5 PICOS components, reference standard justification, spectrum bias handling, synthesis model rationale, and absence of treatment-effect language.
  - project.yaml update maps pico fields: I → index test, C → reference standard, O → accuracy outcomes.

  **Files changed:**
  - `skills/research-question/SKILL.md` — routing table updated (2 new routes); two new process sections appended; file now 483 lines (within 500-line limit, up from 421)
  - `plugin.json` — bumped version to 1.0.30

## [1.0.29] — 2026-03-14

### Added

- **Area 5 (Quality & Proofreading) — Stage 5 reviewer checklist: deduplication and qualitative synthesis blocks:** Added two new conditional blocks to the Stage 5 reviewer checklist in `references/reviewer-protocol.md`.

  **Problem:** Two recent improvements added significant Stage 5 outputs that reviewers should verify, but the Stage 5 checklist was never updated to cover them:
  - 1.0.28 added programmatic deduplication to abstract-screening, writing `05_screening/deduplication_report.md` with PRISMA-required pre/post-deduplication counts. The Stage 5 reviewer had no checklist item to verify the report existed, that counts were internally consistent, or that possible-duplicate flags were annotated.
  - 1.0.26 added a full qualitative synthesis screening path with a new `qualitative_method` field and PICo-based criteria. The Stage 5 reviewer had no checklist items to verify qualitative-specific screening quality: that `qualitative_method` was populated, that mixed-methods studies were included (not excluded), or that exclusion reasons cited the correct qualitative-design criterion.

  **Solution:** Added to the Stage 5 checklist in `reviewer-protocol.md`:

  **Deduplication block (unconditional — applies to all project types):**
  - `05_screening/deduplication_report.md` exists ✗✗
  - Report states pre- and post-deduplication counts ✗✗
  - Counts are internally consistent (no negatives or implausible figures)
  - Row count in report matches `screened_results.csv` actual row count
  - `POSSIBLE_DUPLICATE` records have reasoning annotations (if any)

  **Qualitative Synthesis conditional block (`project_type: qualitative_synthesis` only):**
  - Every record has a non-blank `qualitative_method` field ✗✗
  - All INCLUDE decisions are qualitative or mixed-methods only ✗✗
  - Mixed-methods studies coded INCLUDE when ≥1 qualitative component present ✗✗
  - EXCLUDE reasons for quantitative designs cite the specific design criterion (not generic phrases)
  - PICo criterion alignment documented in reconciliation notes for INCLUDE decisions

  **Files changed:**
  - `references/reviewer-protocol.md` — added Deduplication block (5 items) and Qualitative Synthesis conditional block (5 items) to Stage 5 checklist; file grows by ~20 lines
  - `plugin.json` — bumped version to 1.0.29

## [1.0.28] — 2026-03-14

### Added

- **Area 6 (Robustness) — Programmatic deduplication in abstract screening:** Added an automatic two-pass deduplication step to `skills/abstract-screening/SKILL.md`, inserted between the Abstract Parsing section and the Edge Cases section. The pipeline previously promised "programmatic deduplication" in the Manual Handoff 1 message and in `getting-started.md` but had no actual deduplication logic in the skill itself. This meant duplicate records from multiple database exports (PubMed `.nbib`, Ovid `.ris`, Embase `.ris`) would inflate the screened count and could produce duplicate INCLUDE decisions.

  **Problem:** Users import search results from 2-3 databases into `04_database_search/abstracts/`. The same paper indexed in both PubMed and Embase would appear twice in the abstract pool, be screened twice by 3 extractors each, and potentially receive two INCLUDE decisions — inflating the `n_included` count and requiring manual deduplication before Stage 6. More critically, the PRISMA flow diagram (which PRISMA 2020 requires to report pre/post-deduplication counts separately) had no data source for the "duplicates removed" field.

  **Solution:** Added `## Deduplication` section (~50 lines) with two-pass logic:
  - **Pass 1 — DOI-exact:** Normalize DOIs (lowercase, trim whitespace); group records sharing a DOI; keep the record with the most non-empty fields; mark others as DUPLICATE.
  - **Pass 2 — Fuzzy title+year:** For DOI-less records, compute character-overlap ratio between normalized titles (lowercase, stripped punctuation). Records with same year and ≥85% title similarity → DUPLICATE (keep most complete). Records with 70–84% similarity → `POSSIBLE_DUPLICATE` (keep in pool, add flag to `reasoning` field at screening time).
  - Writes `05_screening/deduplication_report.md` with all PRISMA-required counts: records identified per database, duplicates removed by method, possible duplicates flagged, records proceeding to screening.
  - Prints a brief pre-screening summary with pre/post counts.

  Updated both completion messages (review types and qualitative synthesis) to report deduplication counts alongside screening counts, labelled as "for PRISMA flow diagram."

  **Files changed:**
  - `skills/abstract-screening/SKILL.md` — added `## Deduplication` section; updated both Step 5 completion messages to include deduplication PRISMA counts; file now 440 lines (within 500-line limit, up from 381)
  - `plugin.json` — bumped version to 1.0.28

## [1.0.27] — 2026-03-14

### Added

- **Area 5 (Quality & Proofreading) — NMA reviewer checklists and manuscript NMA support:** Added network meta-analysis (NMA) conditional checklist blocks to the Stage 6 and Stage 7 reviewer checklists in `references/reviewer-protocol.md`, and updated `skills/manuscript-writing/section-b-quantitative.md` to properly handle NMA outputs.

  **Problem:** The reviewer-protocol's Stage 6 checklist had conditional blocks for standard meta-analysis, diagnostic test accuracy (DTA), and qualitative synthesis — but no block for NMA. Similarly, Stage 7 had DTA and qualitative synthesis conditional blocks but nothing for NMA. Yet the data-extraction skill (Step 6b) produces a distinct set of NMA outputs: `nma_results.md`, `nma_analysis.R`, and 4 required figures (`network_geometry.png`, `nma_forest.png`, `sucra_ranking.png`, `nma_league_table.csv`). Without reviewer checklists, a reviewer evaluating NMA outputs had no stage-specific criteria — they could not verify network geometry documentation, consistency testing, league table completeness, SUCRA rankings, or GRADE-NMA (which has 2 NMA-specific domains beyond standard GRADE: Indirectness and Incoherence).

  Additionally, the Statistician agent prompt in `section-b-quantitative.md` only read `meta_analysis_results.md` — it never read `nma_results.md` when NMA was enabled, and the manuscript SELF-CHECK had no NMA items.

  **Files changed:**

  - `references/reviewer-protocol.md`:
    - **Stage 6 checklist — new NMA conditional block (10 items):**
      - `nma_results.md` existence ✗✗
      - Network geometry: nodes, edges, studies per edge, sparse edge flagging ✗✗
      - Consistency: global design-by-treatment test (χ², p) + node-splitting for ≥3 comparisons ✗✗
      - League table: all pairwise NMA estimates with CI/CrI, direct vs. indirect annotated ✗✗
      - Treatment ranking: SUCRA/P-score for every treatment, mean rank + rank probability histogram ✗✗
      - Heterogeneity: tau² and I² for network model
      - GRADE-NMA 7-domain table (standard 5 + Indirectness + Incoherence) ✗✗
      - Sensitivity analysis (high-RoB exclusion), SUCRA rankings stability ✗✗
      - All 4 output files exist: `network_geometry.png`, `nma_forest.png`, `sucra_ranking.png`, `nma_league_table.csv` ✗✗
      - Analysis script exists with correct imports ✗✗
    - **Stage 7 checklist — new NMA manuscript conditional block (8 items):**
      - Network geometry figure numbered and referenced in Results ✗✗
      - League table as Table or supplement ✗✗
      - SUCRA/P-score ranking plot referenced ✗✗
      - Statistical Methods describes NMA model and convergence criteria (if Bayesian)
      - Consistency test results in Methods/Results ✗✗
      - GRADE-NMA 7-domain table with downgrading rationale ✗✗
      - Sensitivity analysis results and treatment hierarchy stability in Discussion
      - Discussion frames rankings with indirect evidence uncertainty

  - `skills/manuscript-writing/section-b-quantitative.md`:
    - **Statistician agent prompt:** Added NMA results source: reads `nma_results.md` when `network_meta_analysis` is true; expanded Prepare list to include league table, SUCRA ranking table, network geometry figure legend, and GRADE-NMA 7-domain certainty table
    - **Manuscript SELF-CHECK — Statistical Reporting block:** Added NMA conditional item: league table present; SUCRA rankings reported; consistency test results in text; GRADE-NMA 7-domain table; network geometry and NMA forest plot referenced by number

- `plugin.json` — bumped version to 1.0.27

## [1.0.26] — 2026-03-14

### Added

- **Area 4 (Research Type Coverage) — qualitative_synthesis screening and eligibility criteria:** Added dedicated `qualitative_synthesis` process sections to `abstract-screening/SKILL.md` and `inclusion-exclusion/SKILL.md`. Previously, qualitative synthesis projects had no guidance for Stages 3 and 5 — they would fall through to the generic review-types path, which uses PICO-based criteria and INCLUDE/EXCLUDE logic that is inappropriate for qualitative research.

  **Problem:** The qualitative_synthesis project type was well-supported at Stage 6 (data-extraction has a full qualitative path including thematic synthesis, CERQual, and JBI appraisal) and Stage 7 (manuscript-writing section A covers ENTREQ and qualitative reporting). But Stages 3 and 5 used the wrong framework:
  - Stage 3 (`inclusion-exclusion`): Methodologist was prompted to draft PICO-based criteria with Population, Intervention, Comparator, and Outcomes — none of which map to qualitative synthesis. The correct framework is **PICo** (Population, phenomenon of Interest, Context), with no Comparator or quantitative Outcome.
  - Stage 5 (`abstract-screening`): No qualitative_synthesis section existed. Screening used `INCLUDE | EXCLUDE | UNCERTAIN` with PICO-based criteria, incorrectly excluding or mis-classifying qualitative studies. Mixed-methods studies were not handled.

  **Solution:**

  **`skills/inclusion-exclusion/SKILL.md` — new `## Process — qualitative_synthesis` section:**
  - Methodologist prompt replaced with PICo-specific template: Population (P), Phenomenon of Interest (I — specific experience/behaviour/process), Context (Co — setting, country, healthcare system)
  - No Intervention, Comparator, or Outcome fields — explicitly excluded from template
  - Study design criteria enumerate acceptable qualitative methods (semi-structured interviews, focus groups, ethnography, grounded theory, phenomenology, thematic analysis, PAR, narrative inquiry) and state mixed-methods handling rule explicitly
  - SELF-CHECK updated: verifies all three PICo components are operationalized, phenomenon is specific, no PICO-style fields sneak in
  - Librarian step: verifies searchability of PICo components and recommends qualitative study design filters per database (PubMed, Ovid)
  - Output template uses PICo table structure (not PICO)
  - Quick Review criteria updated for PICo quality: phenomenon specificity, meta_ethnography vs. meta_aggregation scope guidance (meta_ethnography needs conceptually diverse studies; meta_aggregation needs unambiguous findings)
  - Section inserted before `## Process — Original Research` to follow the logical project-type ordering

  **`skills/abstract-screening/SKILL.md` — new `## Process — qualitative_synthesis` section (~90 lines):**
  - Replaces the INCLUDE/EXCLUDE/UNCERTAIN decision framework with a two-step filter: (1) study design filter (exclude quantitative-only; UNCERTAIN for mixed-methods; INCLUDE for clearly qualitative); (2) PICo relevance filter applied after design confirmed
  - Default bias toward INCLUDE preserved (prefer INCLUDE when abstract lacks detail — full text resolves)
  - New output field: `qualitative_method` — captures the specific qualitative research method at screening (semi-structured interviews, focus groups, ethnography, grounded theory, phenomenology, NR). Used by Stage 6 Methodologist to select the appropriate synthesis approach (thematic vs. meta-ethnography vs. meta_aggregation vs. framework)
  - Methodologist reconciliation step adds mixed-methods handling rule: do not exclude if ≥1 qualitative component is reported
  - Review criteria updated: checks qualitative design classification accuracy, mixed-methods handling, `qualitative_method` field completion, PICo criterion consistency
  - Completion message includes tabulation of qualitative methods in included studies (useful for Stage 6 synthesis planning)
  - Section inserted between DTA and case_report sections, consistent with ordering convention

**Files changed:**
- `skills/abstract-screening/SKILL.md` — added `## Process — qualitative_synthesis` section (~90 lines); file now ~370 lines (within 500-line limit)
- `skills/inclusion-exclusion/SKILL.md` — added `## Process — qualitative_synthesis` section (~85 lines); file now ~360 lines (within 500-line limit)
- `plugin.json` — bumped version to 1.0.26

## [1.0.25] — 2026-03-14

### Changed

- **Maintenance — Split `manuscript-writing/SKILL.md` into section sub-files:** The manuscript-writing skill had grown to 965 lines (nearly 2× the 500-line limit), making it difficult to navigate and maintain. Split the four manuscript type sections into separate sub-files under `skills/manuscript-writing/`, reducing the main SKILL.md to 68 lines.

  **Problem:** A single 965-line file containing four largely independent manuscript workflows (qualitative synthesis, quantitative/mixed methods, case report, diagnostic test accuracy) was hard to navigate, exceeded the 500-line guideline by 93%, and made targeted edits error-prone because any change required scrolling through the entire file.

  **Solution:** Extract each section into a dedicated sub-file and replace the section bodies in `SKILL.md` with a compact routing table that instructs the agent to read the appropriate sub-file.

  **Files created:**

  - `skills/manuscript-writing/section-a-qualitative.md` (117 lines) — Section A: Qualitative Synthesis Manuscript. ENTREQ reporting standard. CERQual, PRISMA-ScR, reflexivity placeholder, thematic/meta-ethnography/framework/meta-aggregation synthesis approaches. Includes the parallel Writer A + Writer B dispatch prompts and the qualitative-specific SELF-CHECK items.

  - `skills/manuscript-writing/section-b-quantitative.md` (302 lines) — Section B: Quantitative / Mixed Methods Manuscript. All project types except qualitative_synthesis, case_report, and diagnostic_test_accuracy. PRISMA 2020 / PRISMA-ScR / STROBE / CONSORT reporting. Parallel Writer A + Writer B + Statistician dispatch prompts; full merge SELF-CHECK (grammar, structure, citations, statistical reporting, tables/figures, reporting guideline compliance); full review criteria checklist; review iteration loop with REVISION REQUIRED table; completion message.

  - `skills/manuscript-writing/section-c-case-report.md` (241 lines) — Section C: Case Report Manuscript. CARE 2013 (13-item) reporting standard. CARE-specific Writer A (Introduction + Case Narrative) and Writer B (Timeline + Diagnostics + Treatment + Outcomes + Discussion) prompts; CARE SELF-CHECK (items 1–12, patient privacy, writing quality); CARE-specific review criteria (guideline compliance, clinical content, teaching value, patient privacy).

  - `skills/manuscript-writing/section-d-dta.md` (230 lines) — Section D: Diagnostic Test Accuracy Manuscript. STARD 2015 (30-item) reporting standard. Writer A (Introduction + Methods with QUADAS-2 and bivariate model description) + Writer B (Results + Discussion with clinical utility framing) + Statistician (5 figures: QUADAS-2 traffic light, sensitivity/specificity forest plots, SROC curve, Fagan nomogram); DTA-specific SELF-CHECK (STARD compliance, results accuracy, clinical utility framing, no treatment-effect language).

  **Files changed:**

  - `skills/manuscript-writing/SKILL.md` — Reduced from 965 lines to 68 lines. Retains full Prerequisites, Edge Cases (pre-flight checks, study count guards, journal guidelines check), and Process routing. The routing block is now a concise table mapping each `project_type` to its sub-file with instructions to read and follow the sub-file exactly.

  - `plugin.json` — bumped version to 1.0.25

## [1.0.24] — 2026-03-14

### Added

- **Area 1 (Autonomy & Self-Correction) — Quick Review loops for all research-question process paths:** Added Quick Review steps with retry logic to the three alternative paths in `skills/research-question/SKILL.md` that were missing them.

  **Problem:** The main review types path (systematic_review, scoping_review, meta_analysis) has a full 6-step workflow including a Quick Review with retry loop and escalation (Steps 5–6). The three alternative process paths had no review step at all — they wrote directly to `project.yaml` as `completed` without any independent quality check:
  - **case_report**: Step 1 (PI dispatch) → Step 2 (write) → Step 3 (update yaml) — no review.
  - **rapid_review**: Step 1 (PI) → Step 2 (present to user) → Step 3 (write + update yaml) — no review.
  - **original_research**: Step 1 (PI) → Step 2 (Methodologist) → Step 3 (present to user) → Step 4 (write + update yaml) — no review.

  Without a reviewer, a poor-quality case report teaching objectives document, an overly broad rapid review question, or an infeasible original research hypothesis would proceed unchecked to Stage 3 (Inclusion/Exclusion), where the downstream protocol would be built on flawed foundations.

**Files changed:**

- `skills/research-question/SKILL.md`:
  - **Case report path**: Added new Step 3 (Quick Review) before existing Step 3 (now Step 4 — update yaml). Review criteria check: ≥3 distinct teaching points, compelling clinical significance grounded in case features, each CARE checklist concern is specific and actionable (not just named), and related literature references are real and relevant. Full retry loop: REVISE → apply corrections to `research_question.md` → increment counter → re-dispatch reviewer with revision history in context; REJECT or `review_iteration ≥ 2` → escalation with full pause.
  - **Rapid review path**: Separated Step 3 into Step 3 (write output) + new Step 4 (Quick Review) + Step 5 (update yaml, renumbered from 3). Review criteria specific to rapid review quality: PICO is narrow enough for the rapid review scope (single intervention + primary outcome only), methodological shortcuts are explicitly named (not vague), feasibility rationale is grounded in a specific evidence base size from the landscape survey, question is answerable without full systematic review apparatus. Same retry loop pattern.
  - **Original research path**: Separated Step 4 into Step 4 (write output) + new Step 5 (Quick Review) + Step 6 (update yaml, renumbered from 4). Review criteria specific to original research: hypothesis is testable (null/alternative pair or specific primary aim with defined DV), study design logically matches question with appropriate caveats for design limitations, both IVs and DVs are operationally defined, feasibility assessment is realistic given stated data source, IRB/consent considerations are noted where required. Same retry loop pattern.

- `plugin.json` — bumped version to 1.0.24

## [1.0.23] — 2026-03-14

### Added

- **Area 5 (Quality & Proofreading) — DTA and qualitative synthesis reviewer checklists:** Extended `references/reviewer-protocol.md` with stage-specific conditional checklist blocks for the two project types added in v1.0.20–1.0.22 but never covered in the reviewer checklists.

  Without these additions, an Independent Reviewer dispatched to assess a `diagnostic_test_accuracy` Stage 6 output had no stage-specific criteria for QUADAS-2 completeness, bivariate model data capture, SROC outputs, or GRADE-DTA. Similarly, a qualitative synthesis Stage 6 output had no checklist for CERQual, theme documentation, or the Summary of Qualitative Findings table. And the Stage 7 checklist omitted STARD 2015 entirely from its reporting guideline compliance line.

**Files changed:**

- `references/reviewer-protocol.md`:
  - **Stage 6 checklist — new `diagnostic_test_accuracy` conditional block (8 items):**
    - QUADAS-2 completeness (all 4 bias + 3 applicability domains) ✗✗
    - Bivariate model input data present (sensitivity/specificity/variance or TP/FP/FN/TN) ✗✗
    - SROC curve output file ✗✗
    - Sensitivity/specificity forest plots ✗✗
    - LR+, LR−, DOR with 95% CI in synthesis report
    - Threshold effect documented (Spearman correlation)
    - Deeks' funnel plot (≥10 studies) or documented infeasibility
    - GRADE-DTA certainty for sensitivity and specificity ✗✗
  - **Stage 6 checklist — new `qualitative_synthesis` conditional block (7 items):**
    - Per-study extraction completeness (design, setting, themes) ✗✗
    - JBI Critical Appraisal scores for every study ✗✗
    - Synthesis method documented step by step
    - Descriptive vs. analytical themes clearly distinguished ✗✗
    - CERQual confidence ratings (4 domains) per synthesised finding ✗✗
    - Summary of Qualitative Findings (SoQF) table ✗✗
    - Reflexivity statement present
  - **Stage 7 checklist — reporting guideline compliance line:**
    - Added `STARD 2015 (diagnostic_test_accuracy)` to the enumerated list (was omitted; all other project types had their guideline named)
  - **Stage 7 checklist — new `diagnostic_test_accuracy` conditional block (9 items):**
    - STARD 2015 30-item supplement ✗✗
    - QUADAS-2 traffic light plot ✗✗
    - SROC curve in Results ✗✗
    - Sensitivity + specificity forest plots ✗✗
    - Bivariate summary point (sensitivity + specificity + 95% confidence region) ✗✗
    - Threshold effect analysis result in text
    - LR+, LR−, DOR + clinical utility framing in Discussion
    - Fagan nomogram reference (if applicable)
    - GRADE-DTA certainty in Results/SoF table ✗✗
  - **Stage 7 checklist — new `qualitative_synthesis` conditional block (7 items):**
    - ENTREQ 21-item supplement ✗✗
    - PICo framework stated ✗✗
    - Synthesis method steps described ✗✗
    - Findings present synthesised themes with supporting material (not study summaries) ✗✗
    - CERQual/SoQF table ✗✗
    - Reflexivity statement ✗✗
    - Transferability statement in Discussion

## [1.0.22] — 2026-03-14

### Added

- **Area 4 (Research Type Coverage) — DTA manuscript completion:** Added `Section D: Diagnostic Test Accuracy Manuscript` to `skills/manuscript-writing/SKILL.md`, completing the end-to-end DTA pipeline. Without this section, a `diagnostic_test_accuracy` project would fall through to Section B (generic quantitative manuscript), producing a manuscript with incorrect reporting standard (STARD rather than PRISMA), missing QUADAS-2 traffic light plot references, no SROC curve, no Fagan nomogram, and no LR+/LR− clinical utility framing. This completes the DTA project type introduced in v1.0.20 (project-types.md + orchestrator) and v1.0.21 (abstract-screening + data-extraction skills).

**Files changed:**

- `skills/manuscript-writing/SKILL.md`:
  - **Project-type routing block** updated: added `diagnostic_test_accuracy → Section D` branch (previously, DTA fell through to Section B)
  - **Edge Cases required-file check** updated: `diagnostic_test_accuracy` added to the "all other project types" list (DTA does produce screened_results.csv, extracted_data.csv, and synthesis_report.md)
  - **Section D: Diagnostic Test Accuracy Manuscript** added (new section, ~200 lines):
    - **D1 (Fetch Journal Guidelines):** Same WebFetch pattern; notes DTA-specific figure requirements (≥4 figures: QUADAS-2 traffic light, sensitivity/specificity forest plots, SROC curve) and whether STARD checklist supplement is required
    - **D2 (Parallel dispatch — Writer A + Writer B + Statistician):**
      - Writer A: STARD 2015–compliant Introduction + Methods (PICOS objective, QUADAS-2 domain-by-domain description, bivariate model + threshold effect test + HSROC + Deeks' funnel + GRADE-DTA methods)
      - Writer B: Results + Discussion with full STARD flow data, QUADAS-2 narrative, bivariate pooled sensitivity/specificity with 95% CI, LR+/LR−/DOR, I² per metric, SROC reference, Fagan nomogram for clinical utility, GRADE-DTA certainty table
      - Statistician: 5 figure legends (QUADAS-2 traffic light, sensitivity forest, specificity forest, SROC + 95% confidence ellipse, Fagan nomogram), 2 table shells (study characteristics, GRADE-DTA SoF), STARD flow text, statistical text snippets for embedding; references dta_analysis.R outputs from Stage 6 or regenerates if missing
    - **D3 (Merge + SELF-CHECK):** DTA-specific self-check items: STARD flow completeness, all 5 figures referenced by number, sensitivity/specificity as proportions, LR+/LR−/DOR present, I² reported separately per metric, no treatment-effect language, GRADE-DTA for both pooled metrics
    - **D4 (Full Review):** DTA-specific review criteria: STARD flow, QUADAS-2 domain coverage, bivariate model description, I² per metric, GRADE-DTA, clinical utility (LR + Fagan nomogram), accuracy of statistical language
    - **D5 (Update project.yaml + Completion):** Same pattern as other sections; next steps include completing the STARD 2015 30-item supplementary checklist required by most journals
  - Note: manuscript-writing SKILL.md is now ~930 lines — well above the 500-line guideline. A future refactor should consider splitting Sections A–D into referenced sub-files (e.g., `section-a-qualitative.md`, `section-b-quantitative.md`, etc.) once the pipeline-orchestrator is updated to inject sub-file content in stage dispatch.

- `plugin.json` — bumped version to 1.0.22

## [1.0.21] — 2026-03-14

### Added

- **Area 4 (Research Type Coverage) — DTA pipeline completion:** Added `diagnostic_test_accuracy`-specific process paths to `abstract-screening/SKILL.md` and `data-extraction/SKILL.md`. This completes the DTA project type that was introduced in v1.0.20 (which only added support in `project-types.md` and `pipeline-orchestrator`). Without these paths, a DTA project would have fallen through to the generic systematic review extraction logic, producing incorrect output (wrong quality tool, missing 2×2 table fields, wrong synthesis model).

**Files changed:**

- `skills/abstract-screening/SKILL.md` — New `## Process — diagnostic_test_accuracy` section:
  - DTA-specific decision rules: INCLUDE requires both index test AND reference standard applied to same participants; cross-sectional/cohort designs only
  - Explicit EXCLUDE triggers for case-control designs (spectrum bias) and partial verification bias
  - New `threshold_reported` column in screener output CSV — captures thresholds at screening for later heterogeneity analysis
  - DTA-specific review criteria (Step 4): case-control exclusion verification, partial verification check, threshold field population

- `skills/data-extraction/SKILL.md` — New `## Process — diagnostic_test_accuracy` section:
  - DTA-specific extraction template fields: 2×2 contingency table (TP/FP/TN/FN) with reconstruction formula when not explicitly reported; sensitivity/specificity as proportions; threshold; blinding (index and reference); partial verification; QUADAS-2 (all 7 domains — 4 bias + 3 applicability)
  - Data Extractor self-check additions: 2×2 consistency, all 7 QUADAS-2 domains scored, values as proportions, partial verification documented
  - DTA synthesis: threshold effect test (Spearman r on logit-transformed sensitivity/specificity); bivariate model (Reitsma et al., R `mada::reitsma()`) as default; HSROC (Rutter & Gatsonis, `mada::hsroc()`) if threshold effect detected; I² for sensitivity and specificity separately; Deeks' funnel plot + asymmetry test (n ≥ 10); sensitivity analysis excluding high-RoB studies; GRADE-DTA for pooled sensitivity and specificity
  - Executable R script: `dta_analysis.R` with outputs `sensitivity_forest.png`, `specificity_forest.png`, `sroc_curve.png`, `quadas2_traffic_light.png`, `deeks_funnel.png`
  - Note: data-extraction SKILL.md now slightly exceeds the 500-line guideline (~630 lines) due to DTA section addition; future refactor should move qualitative_synthesis process to a sub-file

## [1.0.20] — 2026-03-14

### Added

- **Area 4 (Research Type Coverage):** Added `diagnostic_test_accuracy` as a new (8th) project type — systematic reviews of how accurately an index test identifies a condition vs. a reference standard. Relevant for reviews of dermoscopy, biomarkers, clinical scoring systems, imaging, and any test where the primary outcome is sensitivity/specificity rather than treatment effect.

**Files changed:**

- `references/project-types.md` — New `## 8. diagnostic_test_accuracy` section with full stage-by-stage adaptations:
  - PICOS framing (P=suspected patients, I=index test, C=reference standard, O=diagnostic accuracy, S=cross-sectional/cohort design)
  - Stage 3 eligibility: includes spectrum bias and partial verification bias as exclusion triggers; flags case-control study design as a spectrum bias concern
  - Stage 4 search: diagnostic accuracy filter block (`sensitivity[tiab]`, `specificity[tiab]`, `ROC curve[MeSH]`, `likelihood ratio[tiab]`) added on top of standard concept blocks
  - Stage 6 data extraction: full 2×2 table (TP/FP/TN/FN), QUADAS-2 (4 bias domains + 3 applicability domains), index test and reference standard blinding documentation
  - Stage 6 synthesis: bivariate model (Reitsma et al.) for pooled sensitivity/specificity with 95% confidence ellipse on SROC plane; HSROC curve (Rutter & Gatsonis) when threshold effect detected (Spearman r > 0.6); I² separately for sensitivity and specificity; Deeks' funnel plot (≥10 studies); GRADE-DTA assessment; R code using `mada` package (`reitsma()`, `ROCellipse()`) saving `sensitivity_forest.png`, `specificity_forest.png`, `sroc_curve.png`, `quadas2_traffic_light.png`, script to `dta_analysis.R`
  - Stage 7 manuscript: STARD 2015 (30-item) reporting, STARD flow diagram, QUADAS-2 traffic light plot (Figure 1), sensitivity/specificity forest plots (Figures 2–3), SROC plot (Figure 4), GRADE-DTA SoF table, Fagan nomogram for clinical utility

- `skills/pipeline-orchestrator/SKILL.md`:
  - Added `diagnostic_test_accuracy` to project type list (Required parameter #2) with brief description
  - Updated parameter #3 label to include PICOS alongside PICO/PEO/PCC/PICo
  - Added PICOS decomposition guidance for DTA projects (spectrum, threshold, partial verification considerations)
  - Added `reporting_standard` auto-select: `diagnostic_test_accuracy→STARD`
  - Added `meta_analysis` always-false note for `diagnostic_test_accuracy`
  - Added `diagnostic_model` field to `project.yaml` template (`bivariate` | `hsroc`; DTA projects only)
  - Updated onboarding blurb to mention STARD DTA reviews

- `references/getting-started.md`:
  - Added STARD 2015–compliant DTA reviews to methodological capabilities list
  - Added GRADE-DTA and QUADAS-2 to certainty assessment and risk-of-bias lists
  - Added `mada` R package to analysis scripts list
  - Added DTA row to Review Type Decision Guide: "Is your question 'how accurately does this test identify the condition?'"

- `references/agent-roles.md` (minimal targeted additions):
  - Methodologist bias tools: Added QUADAS-2 with 4-domain and 3-applicability description; spectrum bias and partial verification guidance; conditional application rule (`project_type: diagnostic_test_accuracy`)
  - Statistician expertise: Added DTA meta-analysis methods (bivariate model, HSROC, R `mada` package) to the meta-analytic methods list

- `plugin.json` — bumped version to 1.0.20; updated description to mention diagnostic test accuracy reviews

## [1.0.19] — 2026-03-14

### Changed

- `skills/manuscript-writing/SKILL.md` — Area 4 (Research Type Coverage) + Area 6 (Robustness & Edge Cases): Fixed a critical bug where `case_report` and `original_research` projects would always fail with "Abstract screening output not found" at Stage 7, because the Edge Cases pre-flight check unconditionally required `screened_results.csv` — a file that is never produced for those project types (both skip Stages 3–5 per the pipeline-orchestrator):
  - **Edge Cases section rewritten to be project-type aware:** Three separate pre-flight check branches now apply based on `project_type` read from project.yaml at the start of the check:
    - **`case_report`**: Checks only for `06_data_extraction/case_presentation.md` (the correct Stage 6 output for case reports); explicitly skips the screened_results/extracted_data/synthesis_report checks. Error message points to the correct remediation step.
    - **`original_research`**: Checks for `06_data_extraction/synthesis_report.md` and `06_data_extraction/cleaned_data.csv`; explicitly skips the screened_results check. Error messages are specific to the original_research Stage 6 workflow.
    - **All other types** (systematic_review, scoping_review, rapid_review, meta_analysis, qualitative_synthesis): Unchanged — original checks apply, with the qualitative_synthesis extras retained.
  - **Included study count guard** now prefaced with a note that it is skipped for `case_report` and `original_research` (both lack a screening stage).
  - **Process routing block updated**: `case_report` now routes to new **Section C: Case Report Manuscript** (same pattern as `qualitative_synthesis` routing to Section A).
  - **Section C: Case Report Manuscript added** — full CARE 2013-compliant manuscript workflow:
    - **C1 (Fetch Journal Guidelines):** Same WebFetch as Section B Step 1; handles missing URL gracefully (defaults to CARE structure with user notification).
    - **C2 (Writer A — Introduction + Case Narrative):** Writer A drafts: Title (with "case report" per CARE item 1), Introduction (CARE item 5 — uniqueness, educational value), Patient Information (CARE item 6 — de-identified demographics, medical/family history), Clinical Findings (CARE item 7 — physical exam with measurements). Anonymization guidance (relative time references, no DOB/name) included in prompt.
    - **C3 (Writer B — Timeline through Discussion):** Writer B drafts: Timeline (CARE item 8 — chronological table with relative time references), Diagnostic Assessment (CARE item 9 — methods, differentials, final diagnosis), Therapeutic Intervention (CARE item 10 — doses, routes, treatment changes with reasons), Follow-up and Outcomes (CARE item 11 — clinician and patient-reported, adherence, AEs), Discussion (CARE item 12 — 2-3 specific teaching points, comparison to literature, limitations), Patient Perspective (CARE item 13 — optional).
    - **C4 (Merge + Self-Check):** Standard merge into complete 17-section manuscript (including Informed Consent placeholder — CARE item 4). SELF-CHECK block covers: CARE items 1–12 completeness, patient privacy (no identifying information), writing quality (tense, abbreviations, citations, word count).
    - **C5 (Full Review):** Review criteria specific to case reports: CARE guideline compliance, clinical content quality (diagnostic reasoning, treatment rationale, outcome specificity), teaching value (explicit teaching points linked to case facts), patient privacy, and writing. Same review iteration loop as Section B (REVISE → rebuild → re-review; REJECT or ≥2 iterations → escalation).
    - **C6 (Update project.yaml + Completion message):** Same pattern as Section B Step 6; completion message includes case-report-specific next steps (patient consent documentation, IRB exemption).

- `plugin.json` — bumped version to 1.0.19

## [1.0.18] — 2026-03-14

### Changed

- `skills/manuscript-writing/SKILL.md` — Area 6 (Robustness & Edge Cases): Added **Pre-Flight Edge Case Check** section between Prerequisites and Process:
  - **Required file check**: Verifies that `screened_results.csv`, `extracted_data.csv`, and `synthesis_report.md` all exist before dispatching any agents. For `qualitative_synthesis` projects, also checks `extracted_quotes.csv` and `casp_appraisal.md`. Any missing file triggers an immediate STOP with a specific, actionable error message pointing the user to the correct remediation step.
  - **Included study count guard**: Counts `INCLUDE` decisions from screened_results.csv and applies graduated rules: (1) zero studies → STOP with guidance to review criteria or expand search; (2) 1 study with `meta_analysis: true` → auto-downgrades to narrative synthesis, adds required Limitations text; (3) 2 studies with `meta_analysis: true` → same auto-downgrade with appropriate Limitations text; (4) fewer than 5 studies with `network_meta_analysis: true` → auto-downgrades NMA to pairwise synthesis. All downgrades print a clear notification to the user and inject the appropriate Limitations language into the Writers' instructions.
  - **Journal guidelines check**: Notes when `journal_guidelines_url` is blank/TBD so the agent prompts the user for manual paste at Step 1 rather than attempting a failing WebFetch silently.
  - Manuscript-writing was the only stage-7 skill without edge case handling — all other skills (abstract-screening, data-extraction, database-search-build) already had analogous pre-flight checks added in v1.0.5.

- `plugin.json` — bumped version to 1.0.18

## [1.0.17] — 2026-03-14

### Changed

- `skills/manuscript-writing/SKILL.md` — Area 4 (Research Type Coverage): Added **Section A: Qualitative Synthesis Manuscript** to complete the `qualitative_synthesis` pipeline end-to-end (project type was introduced in v1.0.13, data-extraction process in v1.0.16, but manuscript-writing had never been updated):
  - Added **project-type routing block** at the top of the Process section — routes `qualitative_synthesis` to Section A and all other types to Section B (existing process, now labelled)
  - **Section A: Qualitative Synthesis Manuscript** — full ENTREQ-compliant manuscript workflow:
    - **A1 (Fetch Journal Guidelines):** Same as quantitative Step 1 but additionally notes COREQ/ENTREQ-specific requirements
    - **A2 (Parallel Writers):** Two writer agents tailored to qualitative synthesis:
      - Writer A drafts Introduction + Methods: ENTREQ items 1–17 coverage, synthesis approach justification with method-specific detail (Thomas & Harden for thematic, Noblit & Hare for meta-ethnography, JBI for meta-aggregation, framework mapping), CERQual appraisal workflow description, CASP quality appraisal coverage
      - Writer B drafts Results + Discussion: study characteristics table (Table 1) with CASP scores, analytical themes with ≥2 verbatim supporting quotes per theme, CERQual Summary of Qualitative Findings table (Table 2) with confidence ratings and justifications, PRISMA-ScR flow data, reflexivity placeholder (ENTREQ item 21) flagged for PI
    - **A3 (Merge + Self-Check):** Standard merge with additional qualitative-specific self-check items: ENTREQ items 1–21 coverage, verbatim quote requirement, CERQual table completeness, reflexivity placeholder flag, no-statistical-pooling guard, PRISMA-ScR (not standard PRISMA) flow, no causal language guard
    - **A4 (Full Review + project.yaml update):** Uses same review iteration loop (initialize `review_iteration = 0`, REVISE → rebuild table → re-dispatch merge, REJECT or ≥2 iterations → escalation) as Section B Step 5

- `plugin.json` — bumped version to 1.0.17

## [1.0.16] — 2026-03-14

### Changed

- `skills/data-extraction/SKILL.md` — Area 4 (Research Type Coverage): Added three missing process sections:
  - **Step 6b (NMA)**: New step that fires when `review_config.network_meta_analysis: true`. Statistician produces: network geometry (node/edge enumeration + diagram code), consistency assessment (global design-by-treatment interaction test + local node-splitting for 3-5 key comparisons with p < 0.1 threshold), pooled NMA estimates as a full league table with direct vs. indirect evidence annotation, SUCRA/P-score ranking with rank probability histogram, tau²/I² for the network, GRADE-NMA (standard 5 domains + Indirectness + Incoherence), and sensitivity NMA excluding high-RoB studies. Code requirements: R `netmeta` (frequentist) or `gemtc`/`BUGSnet` (Bayesian); saves `network_geometry.png`, `nma_forest.png`, `sucra_ranking.png`, `nma_league_table.csv` to `analysis_scripts/outputs/`; script to `nma_analysis.R`. Output written to `nma_results.md`.
  - **`original_research` process**: 4-step workflow for primary research projects: (1) Statistician designs REDCap-compatible data collection form with participant ID, demographics, primary/secondary outcomes, confounders, adverse events, and data quality flags — written to `data_collection_form.csv`; (2) data validation and cleaning (range checks, missing data pattern analysis, cleaning log) once raw data is placed in `raw_data/` — includes a pipeline pause message if data collection not yet complete; (3) Statistician runs descriptive stats, primary analysis per study protocol, pre-specified subgroup/sensitivity analyses, generates executable Python/R script; (4) updates project.yaml.
  - **`qualitative_synthesis` process**: 6-step workflow to complete the qualitative evidence synthesis pipeline (first added to `project-types.md` in v1.0.13 but never implemented in this SKILL). Steps: (1) Methodologist designs qualitative extraction template (study design, participant characteristics, data collection method, verbatim quotes per theme, author interpretations, reflexivity statements, CASP scores); (2) 3 parallel Data Extractor agents extract verbatim participant quotes and complete CASP Qualitative Checklist — with SELF-CHECK block requiring verbatim quotes (no paraphrasing), ≥1 quote per theme with page number, all 10 CASP items scored; (3) CASP quality appraisal compiled per JBI guidance (studies rarely excluded, quality informs CERQual); (4) Methodologist performs thematic synthesis — with 4 method-specific instructions for `thematic` (Thomas & Harden line-by-line coding → descriptive → analytical themes), `framework` (map to pre-specified framework), `meta_ethnography` (Noblit & Hare reciprocal/refutational translation → line-of-argument), `meta_aggregation` (JBI findings → categories → synthesized findings) — with Full Review + retry loop; (5) CERQual assessment across 4 domains (methodological limitations, coherence, adequacy, relevance) with confidence ratings per synthesized finding; (6) updates project.yaml.

- `references/agent-roles.md` — Area 4 (Research Type Coverage): Added behavioral rule #5 (NMA) to the Statistician role:
  - Network geometry enumeration with diagram specifications (node size ∝ participants, edge width ∝ study count)
  - Consistency assessment protocol with explicit p < 0.1 threshold for global test and node-splitting
  - Model selection guidance: frequentist (`netmeta`) vs. Bayesian (`gemtc`/`BUGSnet`) with convergence criteria (R-hat < 1.01, ESS > 1,000)
  - League table format specification (annotated with direct vs. indirect evidence, study counts per cell)
  - GRADE-NMA instructions: standard 5 domains + Indirectness domain + Incoherence domain
  - Figure requirements and output path specifications

- `plugin.json` — bumped version to 1.0.16

## [1.0.15] — 2026-03-13

### Changed

- `references/reviewer-protocol.md` — Area 5 (Quality & Proofreading): Added **Stage-Specific Review Checklists** section with concrete, binary pass/fail criteria for each of the 7 pipeline stages:
  - **Stage 1 (Literature Search):** 13 items covering search comprehensiveness (3+ sources, 5+ reviews, synonym use), gap quality (specific, falsifiable, non-duplicative), strategic analysis (gap-to-citation traceability, opportunity ranking calibrated to project_type, internal consistency between saturation and opportunity), and citation integrity. ✗✗ (REJECT-threshold) flags on: unsubstantiated gap claims, gaps already covered by cited reviews, internal contradiction between saturation/opportunity ranking.
  - **Stage 2 (Research Question):** 11 items covering PICO completeness (all components defined, framework matches project_type), novelty (named-review gap citation, 3-year deduplication search), feasibility/clinical significance (evidence-base grounded, concrete unmet need), and downstream compatibility (project.yaml pico block updated). ✗✗ flags on: vague/undefined PICO components, framework mismatch, unsubstantiated gap citation.
  - **Stage 3 (Inclusion/Exclusion):** 9 items covering PICO alignment (each component maps to a criterion, no internal contradictions), operationalizability (title/abstract assessable, no subjective proxies), design/scope (study design matches project_type, date range with rationale, breadth risk check), and conditional PROSPERO check. ✗✗ flags on: unmapped PICO components, internal criterion contradictions, missing PROSPERO protocol when required.
  - **Stage 4 (Database Search):** 9 items covering coverage (every database in project.yaml has a strategy, 3+ concept blocks, 3+ synonyms per block), technical correctness (Boolean logic, verified MeSH/Emtree, database-specific syntax, filters as final AND step), and reproducibility. ✗✗ flags on: missing database strategy, unverified MeSH terms, Boolean syntax errors, non-reproducible shorthand.
  - **Stage 5 (Abstract Screening):** 8 items covering completeness (every abstract has a row, 3 screener columns, agreement rate reported), decision quality (EXCLUDE cites specific criterion, UNCERTAIN appropriateness, INCLUDE bias for borderline, >60% EXCLUDE rate flagged), and conflict resolution (all CONFLICT rows have reconciliation notes). ✗✗ flags on: missing abstract rows, EXCLUDE without criterion citation, unresolved CONFLICT rows.
  - **Stage 6 (Data Extraction):** 12 items covering extraction completeness (one row per study, no shortcut NR, verbatim effect sizes with source references, clinical coding columns), synthesis quality (characteristics table covers all studies, narrative synthesis covers all outcomes, RoB grounded in scores, discrepant findings documented, no fabricated stats), and conditional meta-analysis checks (forest/funnel plots, I²/Q/tau², GRADE, executable analysis script). ✗✗ flags on: missing study rows, generic RoB statements, meta-analysis outputs missing when required.
  - **Stage 7 (Manuscript):** 17 items covering journal compliance (word count, abstract format, mandatory sections), reporting guideline compliance (correct guideline per project_type, PRISMA flow diagram consistency, registration number), writing/internal consistency (tense, abbreviations, APA numbers, no placeholder flags), statistical reporting (effect estimates + 95% CI + exact p, heterogeneity metrics, GRADE, no undisclosed post-hoc), and tables/figures (sequential numbering, self-contained legends, no orphaned references). ✗✗ flags on: wrong/missing reporting guideline, PRISMA flow inconsistency, placeholder flags, missing CIs/p-values.
  - Updated Review Dispatch Template Task step 2 to reference the Stage-Specific Checklist by section name instead of generic criteria list.

- `plugin.json` — bumped version to 1.0.15

## [1.0.14] — 2026-03-13

### Changed

- `skills/inclusion-exclusion/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **SELF-CHECK** blocks to two producing agents and a full explicit retry loop to Step 7:
  - **Methodologist self-check**: Before outputting criteria, verifies: (1) every PICO/PEO component maps to at least one inclusion criterion, (2) each criterion is operationalizable from title/abstract without subjective judgement, (3) no internally contradictory criterion pairs, (4) exclusion criteria add information beyond negating inclusion criteria, (5) study design criteria match the project type (`meta_analysis` → controlled designs; `scoping_review` → broad), (6) date range is present with clinical rationale, (7) language restriction is stated with brief rationale. Failures corrected inline before passing to Librarian.
  - **Librarian self-check**: Before outputting annotations, verifies: (1) every criterion has a searchability verdict (none skipped), (2) key condition/intervention/population terms confirmed via WebSearch to have real MeSH headings — no guessing, (3) each "full-text only" criterion includes a specific reason and an abstract-level proxy for interim screening, (4) if ≥3 criteria are flagged full-text-only, adds a summary note recommending PI simplify criteria to reduce full-text retrieval burden.
  - **Step 7 retry loop**: Previously said "apply revisions directly without re-running the full agent loop" with no iteration handling or escalation path. Now: initialize `review_iteration = 0`; on REVISE, build a REVISION REQUIRED table (finding | criterion affected | issue | required action), re-dispatch Methodologist to address each finding, update `criteria.md`, increment counter, re-dispatch reviewer with revision history; on REJECT or `review_iteration ≥ 2` without APPROVE, trigger full escalation per reviewer-protocol.md with all findings across iterations and pause.

- `plugin.json` — bumped version to 1.0.14

## [1.0.13] — 2026-03-13

### Changed

- `references/project-types.md` — Area 4 (Research Type Coverage): Added **`qualitative_synthesis`** as a 7th supported project type with full stage-by-stage adaptations:
  - **Framework:** PICo (Population, phenomenon of Interest, Context) replaces PICO
  - **Stage 1:** Landscape search augmented with qualitative terminology (phenomenology, grounded theory, ethnography, thematic analysis); CINAHL added as required database
  - **Stage 2:** PI frames question around experience/perspectives/meanings using PICo; research questions use "What are...", "How do...", "What is the experience of..." framing
  - **Stage 3:** Broad inclusion of all qualitative primary study designs (phenomenological, grounded theory, ethnographic, narrative, thematic, action research, mixed-methods with extractable qualitative component); outcome replaced by "reports data relevant to phenomenon of interest"
  - **Stage 4:** Search strategy adds qualitative design filter block (qualitative research MeSH, interviews, focus groups, grounded theory, phenomenolog*, ethnograph*); omits RCT/clinical trial filters; CINAHL added
  - **Stage 6:** Replaces quantitative extraction with qualitative thematic synthesis workflow — Data Extractors extract verbatim quotes and author interpretations; Methodologist performs CASP Qualitative Checklist appraisal; second Methodologist performs synthesis per chosen method; four synthesis approaches supported: `thematic` (Thomas & Harden, default), `framework`, `meta_ethnography` (Noblit & Hare), `meta_aggregation` (JBI); CERQual confidence assessment (the GRADE equivalent for qualitative evidence); outputs `extracted_quotes.csv`, `casp_appraisal.md`, `synthesis_report.md`; no statistical analysis, no Python/R scripts
  - **Stage 7:** ENTREQ (21-item) reporting guideline; manuscript includes analytical themes with illustrative quotes, CERQual Summary of Qualitative Findings table; reflexivity note flagged for PI to add

- `skills/pipeline-orchestrator/SKILL.md` — Area 4 (Research Type Coverage):
  - Added `qualitative_synthesis` to the project type list with description (PICo, ENTREQ, thematic synthesis methods)
  - Updated parameter #3 label to include PICo alongside PICO/PEO/PCC
  - Added `synthesis_approach` as an optional advanced parameter (`thematic` | `framework` | `meta_ethnography` | `meta_aggregation`; defaults to `thematic`)
  - Added PICo decomposition guidance when user provides a vague qualitative research area
  - Updated `reporting_standard` auto-select: `qualitative_synthesis→ENTREQ`
  - Added `synthesis_approach` field to `project.yaml` template (qualitative_synthesis only)
  - Set `meta_analysis` and `network_meta_analysis` always false for qualitative_synthesis
  - Clarified PICo component mapping in `pico` block comments

- `references/getting-started.md` — Area 4 (Research Type Coverage) + Area 8 (Academic Onboarding):
  - Added ENTREQ-compliant qualitative evidence synthesis to methodological capabilities list; added CASP Qualitative to risk-of-bias appraisal tools; added CINAHL to search databases
  - Added `qualitative_synthesis` row to Review Type Decision Guide
  - Added detailed "When to choose qualitative synthesis" guidance with PICo framing and CERQual explanation
  - Added "Qualitative synthesis methods" subsection comparing thematic synthesis, meta-ethnography, framework synthesis, meta-aggregation
  - Added two qualitative synthesis example trigger phrases to Starting the Pipeline section

- `plugin.json` — bumped version to 1.0.13

## [1.0.12] — 2026-03-13

### Changed

- `skills/research-question/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **SELF-CHECK** block to PI Step 3 and **explicit retry logic** to Step 5:
  - **PI Step 3 self-check**: Before presenting the final question to the user, PI verifies: (1) every PICO/PEO component is explicitly defined with no vague terms, (2) gap alignment cites a named gap by section heading from the landscape report (not a generic claim), (3) a web search is performed to check for near-identical published reviews in the last 3 years — revises scope/framing if found, (4) feasibility justification is calibrated to the evidence base size described in the landscape report, (5) PICO/PEO/PCC framework matches the `project_type` in project.yaml
  - **Step 5 retry loop**: Previously just said "apply revisions automatically, then mark approved" with no iteration tracking. Now: initialize `review_iteration = 0`; on REVISE, build a REVISION REQUIRED table (finding | PICO component | issue), apply revisions to `research_question.md` without re-running the full agent loop, increment counter, re-dispatch reviewer with revision history; on REJECT or `review_iteration ≥ 2` without APPROVE, trigger full escalation with pause

- `skills/manuscript-writing/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **explicit retry loop** to Step 5 (Full Review):
  - Previously the step listed review criteria but had no iteration handling — reviewers had no pathway to revise and re-review
  - Now: initialize `review_iteration = 0`; on REVISE (any reviewer), build a consolidated REVISION REQUIRED table (reviewer | category | finding | location | action), re-dispatch merge agent to address all findings and annotate revisions, update manuscript.md, increment counter, re-dispatch Full Review with revision history; on REJECT or `review_iteration ≥ 2` without APPROVE, trigger full escalation with all reviewer findings across iterations and pause

- `plugin.json` — bumped version to 1.0.12

## [1.0.11] — 2026-03-13

### Changed

- `skills/data-extraction/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **SELF-CHECK** blocks to two producing agents and explicit retry logic to Step 4:
  - **Data Extractor self-check**: Before outputting CSV rows, each extractor verifies: (1) every assigned paper has exactly one row, (2) "NR" used only for genuinely absent data, (3) effect sizes/CIs/p-values transcribed character-for-character with page/table references, (4) `source_notes` populated for all key values, (5) if `clinical_coding: true` — all coding columns filled with verified codes or `[UNSPECIFIED]` with notes.
  - **Statistician synthesis self-check**: Before writing `synthesis_report.md`, verifies: (1) study characteristics table covers all studies in `extracted_data.csv`, (2) narrative synthesis addresses every outcome in the research question, (3) risk-of-bias summary grounded in actual quality scores (not generic statements), (4) discrepant findings flagged explicitly in a "Discrepant Findings" subsection, (5) no fabricated statistics — all values traceable to extraction columns, (6) heterogeneity noted even without meta-analysis.
  - **Step 4 retry loop**: Added explicit iteration logic — initialize `review_iteration = 0`; on REVISE, build a REVISION REQUIRED table, re-dispatch Data Extractors on flagged rows only, update CSV, increment counter, re-run reviewer; on REJECT or `review_iteration ≥ 2` without APPROVE, trigger full escalation per reviewer-protocol.md.

- `skills/database-search-build/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **SELF-CHECK** block to Librarian agent and explicit retry logic to Step 3:
  - **Librarian self-check**: Before writing output, verifies: (1) every target database has a complete numbered strategy, (2) Boolean logic is structurally correct (no orphaned operators, correct nesting), (3) MeSH/Emtree terms verified via WebSearch — no guessing, (4) ≥3 free-text synonyms per concept block including truncated forms, (5) database-specific syntax correct for each platform (PubMed `[MeSH Terms]`/`[All Fields]`, Ovid `/`/`.mp.`, Embase `/exp`), (6) filters applied as final AND step, (7) strategy is copy-paste reproducible.
  - **Step 3 retry loop**: Previously listed review criteria with no iteration handling. Now: initialize `review_iteration = 0`; on REVISE, build a REVISION REQUIRED table (finding | database | line numbers), re-dispatch Librarian to address each finding, overwrite search_strategy.md, increment counter, re-run reviewer; on REJECT or `review_iteration ≥ 2` without APPROVE, trigger full escalation.

- `plugin.json` — bumped version to 1.0.11

## [1.0.10] — 2026-03-13

### Changed

- `skills/abstract-screening/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **SELF-CHECK** blocks to both producing agents and explicit retry logic to Step 4:
  - **Data Extractor self-check**: Before submitting CSV output, each extractor verifies: (1) every abstract has exactly one decision row, (2) every EXCLUDE decision cites a specific criterion from criteria.md (not vague "not relevant"), (3) UNCERTAIN used only for genuine information absence — not borderline hedging, (4) confidence calibration (HIGH = clear match, LOW = close call), (5) INCLUDE bias — borderline abstracts must be INCLUDE, not EXCLUDE.
  - **Methodologist self-check**: Before outputting reconciled CSV, Methodologist verifies: (1) every abstract ID appears exactly once, (2) spot-checks 5 random rows for correct majority vote, (3) all 3-way disagreements flagged as CONFLICT, (4) arithmetic consistency of reported agreement rate, (5) every CONFLICT row has a substantive reconciliation_notes entry. Corrections applied inline.
  - **Step 4 retry loop**: Previously a bare "dispatch Full Review per reviewer-protocol.md" with no iteration handling. Now: initialize `review_iteration = 0`; on REVISE, re-screen flagged abstracts with reviewer findings table prepended, update results CSV, increment counter, re-dispatch reviewer with revision history; on REJECT or `review_iteration >= 2` without APPROVE, trigger full escalation per reviewer-protocol.md.

- `plugin.json` — bumped version to 1.0.10

## [1.0.9] — 2026-03-13

### Changed

- `skills/literature-search/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **SELF-CHECK** blocks to both producing agents:
  - **Librarian self-check**: Verifies coverage (3+ sources, 5+ reviews identified, synonym/MeSH-equivalent terms used), gap quality (specific and falsifiable, not already addressed by cited reviews, actionable), and citation integrity (every bibliography item cited in body, no fabricated citations — unverified references marked `[VERIFY]`)
  - **PI self-check**: Verifies gap analysis quality (each gap substantiated by a cited review's stated limitation, distinguishes unstudied vs. low-quality vs. conflicting areas), opportunity ranking soundness (distinct opportunities, feasibility calibrated to project type, impact grounded in real unmet need), and logical consistency (no contradiction between saturation assessment and opportunity ranking)
  - Both self-checks instruct agents to correct issues inline before producing output — failures do not propagate to the reviewer

- `skills/literature-search/SKILL.md` — Area 1 (Autonomy & Self-Correction): Added **explicit retry logic** to Step 4 (Quick Review):
  - Previously the step just said "per reviewer-protocol.md" without implementing the iteration loop — agents had no concrete instructions for handling REVISE/REJECT
  - Now includes: initialize `review_iteration = 0`; on REVISE/REJECT re-dispatch producing agents with the reviewer's findings table as a "REVISION REQUIRED" block (agents address each finding explicitly); re-dispatch reviewer with `review_iteration += 1` and revision history in context; if `review_iteration >= 2` and still not APPROVE, trigger full escalation per reviewer-protocol.md (banner, all reviewer findings, revision notes, pipeline pause)

- `plugin.json` — bumped version to 1.0.9

## [1.0.8] — 2026-03-13

### Changed

- `skills/pipeline-orchestrator/SKILL.md` — Area 7 (Agent Team Coordination): Added structured **PRIOR STAGE OUTPUTS** block to the Stage Dispatch Template:
  - Orchestrator now injects all completed prior stage outputs into each stage's agent context at dispatch time (not just `project.yaml`)
  - Injection is stage-specific: Stage 2 gets landscape report; Stage 3 also gets research question; Stage 4 also gets criteria; Stage 5 also gets search strategy concept block; Stage 6 also gets included studies list; Stage 7 also gets full synthesis report and extracted data sample
  - Sections are skipped gracefully if source files don't exist yet (no error)
  - This eliminates "cold start" problem where agents had to read prior outputs themselves without knowing what prior decisions were most important

- `skills/abstract-screening/SKILL.md` — Area 7 (Agent Team Coordination): Added **RESEARCH QUESTION & PICO CONTEXT** to the Data Extractor agent prompt:
  - Data Extractors now receive `02_research_question/research_question.md` before screening, giving them PICO context to resolve borderline UNCERTAIN cases
  - Added screening guidance: prefer INCLUDE over EXCLUDE for ambiguous abstracts (full-text review is the safety net); use UNCERTAIN only when information genuinely missing from abstract

- `skills/database-search-build/SKILL.md` — Area 7 (Agent Team Coordination): Added **LANDSCAPE SURVEY CONTEXT** to the Librarian agent prompt:
  - Librarian now receives the Preliminary Search Terms and Recommended Databases sections from `01_literature_search/landscape_report.md`
  - Rationale: terms used in existing key papers reflect actual database indexing — seeding MeSH blocks with this vocabulary maximizes recall compared to building terms from scratch

- `plugin.json` — bumped version to 1.0.8

## [1.0.7] — 2026-03-13

### Changed

- `references/agent-roles.md` — Area 3 (Clinical Coding Capabilities): Added behavioral rule #3 to the Data Extractor role:
  - When `review_config.clinical_coding: true`, Data Extractor assigns ICD-10-CM, ICD-11, CPT, and SNOMED CT codes to all diagnoses, conditions, procedures, and population descriptors
  - Includes mapping guidance for non-standard terminology, `[UNSPECIFIED]` flag for parent-code fallbacks, and explicit rule against fabricating codes (must use WebSearch to verify uncertain codes)
  - Outputs go into dedicated columns in the extraction template

- `skills/data-extraction/SKILL.md` — Area 3 (Clinical Coding Capabilities):
  - Step 1 (Statistician template design prompt) now conditionally adds 5 clinical coding columns when `clinical_coding: true`: `diagnosis_icd10`, `diagnosis_icd11`, `procedure_cpt`, `snomed_population`, `clinical_coding_notes`
  - Columns are pipe-separated for multi-code fields and positioned after population columns for natural data flow

- `skills/pipeline-orchestrator/SKILL.md` — Area 3 (Clinical Coding Capabilities):
  - Added `clinical_coding` to optional advanced parameters with description (recommended for dermatology, oncology, procedure-based research)
  - Added `clinical_coding` field to `review_config` block in the `project.yaml` template

- `plugin.json` — bumped version to 1.0.7

## [1.0.6] — 2026-03-13

### Changed

- `skills/research-question/SKILL.md` — Area 4 (Research Type Coverage) + Area 7 (Agent Team Coordination):
  - Added **project-type routing block** at the top: routes to dedicated process sections for `case_report`, `rapid_review`, and `original_research`; all review types use existing process
  - **Step 3.5 (User Confirmation):** Added explicit user-facing confirmation step between PI final selection and writing output — presents proposed PICO question, requests user confirmation or adjustment before proceeding; prevents incorrect questions from propagating to downstream stages
  - **Step 4 output format:** Added explicit markdown template for `research_question.md` (Primary Question, Framework table, Clinical Significance, Anticipated Challenges, Expected Contribution)
  - **Step 5 Quick Review:** Expanded from 4 bullets to 5 specific criteria including: novelty check via web search (changed from 2-year to 3-year lookback), PICO operationalization check, and automatic revision without re-running agent loop
  - **Step 6:** Added explicit instruction to update the `pico` block in `project.yaml` with finalized components
  - **Case report process:** New section — PI frames teaching objectives, clinical significance, and CARE checklist relevance; writes to `research_question.md` for pipeline consistency
  - **Rapid review process:** Abbreviated single-PI-pass process with explicit scope constraint ("narrow enough for rapid review timeframe"); retains user confirmation
  - **Original research process:** New section — PI generates 2-3 candidate primary aims with hypotheses, study design, variables, and feasibility; Methodologist evaluates and recommends; user confirmation retained

- `skills/inclusion-exclusion/SKILL.md` — Area 4 (Research Type Coverage) + Area 1 (Autonomy):
  - **Step 4 output format:** Added explicit markdown template for `criteria.md` with inclusion table (Criterion/Definition/Rationale), exclusion table, and Librarian searchability notes section
  - **Step 5 (PROSPERO Protocol Generation — conditional):** New step that triggers when `review_config.prospero_registration: true` — dispatches Methodologist to generate a PROSPERO-ready protocol document covering all 10 PROSPERO form sections; saves to `03_inclusion_exclusion/prospero_protocol.md`; prints PROSPERO URL and timing tip (register before Stage 4 searching)
  - **Step 7 Quick Review:** Previously empty placeholder — now a full 9-criterion review checklist covering: PICO alignment, operationalizability, internal consistency, result-set size risk (too broad/narrow), meta-analysis homogeneity requirements, scoping review breadth check, NMA criteria check, and language restriction justification; includes automatic revision instruction

- `plugin.json` — bumped version to 1.0.6

## [1.0.5] — 2026-03-13

### Changed
- `skills/database-search-build/SKILL.md` — Area 6 (Robustness & Edge Cases): Added **Edge Cases** section with:
  - **Zero results**: Re-dispatches Librarian with targeted diagnosis prompt; generates `search_strategy_revised.md` with 2 fallback options (moderate to broad)
  - **Too many results (>500)**: Re-dispatches Librarian with narrowing prompt targeting 100-300 results; writes `search_strategy_narrowed.md`; notes user can skip narrowing and use batching instead
  - **Database access failures**: Records skipped databases in `project.yaml`, adds PRISMA flow note and study limitation annotation
- `skills/abstract-screening/SKILL.md` — Area 6 (Robustness & Edge Cases): Added **Edge Cases** section (checked before processing):
  - **Empty abstracts folder**: Prints clear error with expected formats and stops — does not attempt to run with no input
  - **Parse failures**: Records malformed records in `parse_failures.csv`, skips gracefully, reports count; warns if >10% parse failure rate
  - **Very large result sets (>500)**: Saves interim summaries every 5 batches; flags unexpectedly high inclusion rates (>50% after batch 1); recommends search narrowing
- `skills/data-extraction/SKILL.md` — Area 6 (Robustness & Edge Cases): Added **Edge Cases** section (checked before processing):
  - **Missing full texts**: Pre-flight comparison of expected vs. available PDFs; generates `missing_full_texts.md`; writes placeholder rows in extracted_data.csv; does not abort
  - **Unreadable/corrupted PDFs**: Records failures in `extraction_progress.yaml`; sets row values to explanation string; continues extraction
  - **Conflicting data between studies**: Statistician must add "Discrepant Findings" subsection to synthesis_report.md; documents magnitude and methodological explanations; includes sensitivity analysis excluding outlier in meta-analyses
- `plugin.json` — bumped version to 1.0.5

## [1.0.4] — 2026-03-13

### Changed
- `skills/manuscript-writing/SKILL.md` — Area 5 (Quality & Proofreading):
  - **Self-check step added to Step 3 (Merge agent prompt):** Before writing the merged manuscript to disk, the merge agent must perform a structured self-proofreading pass covering: grammar/writing quality (tense, abbreviations, number conventions, SI units), structural completeness (all required sections, within word limits), citation/reference integrity (no orphaned citations, no unresolved `[CITATION NEEDED]` flags), statistical reporting (effect sizes + 95% CI + p-values, heterogeneity metrics, GRADE), table/figure sequential numbering and self-contained figure legends, and reporting guideline compliance (PRISMA/PRISMA-ScR/STROBE/CONSORT/CARE checklist items, study registration details). Any failing item must be corrected before output.
  - **Step 5 Full Review criteria expanded** from 6 generic bullets into a structured 5-category checklist for Independent Reviewers: (1) Journal Compliance, (2) Reporting Guideline Compliance, (3) Writing & Internal Consistency, (4) Statistical Reporting, (5) Tables & Figures. Each category contains specific, actionable review questions for consistent, evidence-based review verdicts.
- `plugin.json` — bumped version to 1.0.4


## [1.0.3] — 2026-03-13

### Changed
- `references/getting-started.md` — complete rewrite for academic audience (med students, MSc/PhD students, junior clinician-scientists, PIs). Prior version was layperson-oriented. New version:
  - Opens with methodological capabilities summary: PRISMA 2020, PRISMA-ScR, STROBE, CARE, random-effects meta-analysis, GRADE, RoB 2/ROBINS-I/NOS/JBI, NMA
  - Adds "Agent Team" table with all roles and responsibilities
  - Adds "AI vs. Human Decisions" section — explicit delineation of what the pipeline handles autonomously vs. what requires researcher judgment
  - Replaces basic pipeline overview with methodological stage table (agents, outputs, manual flag)
  - Manual Handoff 1 now includes: export format details (.nbib/.ris), PubMed/Ovid/Embase-specific syntax notes, deduplication workflow (Rayyan, Zotero, Covidence), PROSPERO tip
  - Manual Handoff 2 now includes: prioritized retrieval workflow (PMC → Unpaywall → institution → ILL → author contact), naming conventions, ILL timing note
  - Review type decision guide uses academic framing: question type (PICO vs. PCC), NMA note, SR vs. scoping decision rule
  - Added "Tips for Better Pipeline Outputs" section: pre-specifying subgroup/sensitivity analyses, GRADE, NMA at setup
  - Trigger phrases updated to academic register ("set up a PRISMA review", "meta-analysis on", "systematic review protocol")

- `skills/pipeline-orchestrator/SKILL.md` — multiple updates:
  - Trigger description: added academic phrases ("run the pipeline on", "set up a PRISMA review", "I need to do a meta-analysis on", "help me structure my systematic review protocol", "PROSPERO registration")
  - Onboarding message: rewritten to methodological framing (lists supported outputs: PRISMA/PRISMA-ScR/STROBE/CARE, mentions agent team, GRADE, automated review cycles)
  - Project setup: now collects PICO components explicitly, includes optional advanced parameters (subgroup analyses, sensitivity analyses, NMA, PROSPERO registration); assumes user knows their PICO
  - `project.yaml` template: added `pico` block and advanced `review_config` fields (`network_meta_analysis`, `prospero_registration`, `subgroup_analyses`, `sensitivity_analyses`)
  - Manual Handoff 1: technical format (export format specs, deduplication options, PROSPERO note, PRISMA record count reminder)
  - Manual Handoff 2: technical format (prioritized PDF retrieval workflow with ILL timing, naming convention, unavailable-article protocol)

- `plugin.json` — bumped version to 1.0.3


## [1.0.2] — 2026-03-13

### Changed
- `references/agent-roles.md` — Expanded Statistician role with explicit code-generation instructions (behavioral rule #5):
  - Python stack specified: pandas, scipy.stats, statsmodels, pymare/DerSimonian-Laird, matplotlib, seaborn
  - R stack specified: meta, metafor, ggplot2, forestplot, dmetar
  - Scripts must be complete, runnable, and saved to `06_data_extraction/analysis_scripts/`
  - Code must include inline comments and print a human-readable summary to stdout
- `skills/data-extraction/SKILL.md` — Step 6 (meta-analytic pooling) now instructs Statistician to:
  - Generate executable Python/R code alongside the markdown report
  - Save forest plot PNG, funnel plot PNG, and results CSV to `06_data_extraction/analysis_scripts/outputs/`
  - Save the analysis script itself to `06_data_extraction/analysis_scripts/meta_analysis_primary.py` (or .R)
  - Added GRADE certainty assessment as an explicit required output
- `plugin.json` — bumped version to 1.0.2



## [1.0.1] — 2026-03-13

### Added
- `references/getting-started.md` — comprehensive layperson onboarding guide covering:
  - Plain-English explanation of the plugin and what it does
  - Prerequisites checklist before starting
  - Natural language examples of how to invoke the pipeline
  - Overview table of all 7 stages with time estimates and manual step indicators
  - Step-by-step instructions for Manual Handoff 1 (running PubMed/MEDLINE/Embase searches)
  - Step-by-step instructions for Manual Handoff 2 (obtaining full-text PDFs via PMC, institutional access, Unpaywall)
  - Decision helper Q&A for choosing between review types (systematic, scoping, rapid, original research, case report, meta-analysis)
  - FAQ section addressing common beginner concerns

### Changed
- `skills/pipeline-orchestrator/SKILL.md` — improved layperson onboarding:
  - Expanded trigger description to include natural language phrases non-experts use ("help me write a paper", "I want to research X", "how do I start", "what should I study")
  - Added friendly intro message that briefly explains the pipeline before asking questions
  - Added guidance to reference `getting-started.md` decision helper when user is unsure about review type
  - Improved project setup to handle vague topics (prompts for narrowing), missing journal info (optional), and users without Ovid/Embase access
  - Manual Handoff 1 message: replaced terse instructions with numbered step-by-step guide for PubMed export (including exact UI navigation: Save → Format → Create file)
  - Manual Handoff 2 message: replaced terse instructions with numbered step-by-step guide for PDF retrieval including PubMed Central, institutional access, DOI lookup, and Unpaywall; added graceful fallback for inaccessible articles
- `plugin.json` — bumped version to 1.0.1

## [1.0.43] — 2026-03-14

### Refactored

- **Rule 6 compliance — `skills/data-extraction/SKILL.md` split into sub-files (676 lines → 305 lines)**

  **Problem:** `skills/data-extraction/SKILL.md` had grown to 676 lines — 35% over the 500-line limit — due to the addition of complete project-type-specific workflows (original_research, qualitative_synthesis, diagnostic_test_accuracy) across prior runs.

  **Solution:** Applied the same sub-file pattern used in `abstract-screening/` and `manuscript-writing/`.

  **Files created:**
  - `skills/data-extraction/section-original-research.md` (153 lines) — REDCap data collection form design, data validation/cleaning, full statistical analysis (Table 1, primary/secondary analyses, subgroup/sensitivity analyses, missing data imputation, executable Python/R script), CONSORT/STROBE compliance self-check.
  - `skills/data-extraction/section-qualitative-synthesis.md` (119 lines) — Qualitative extraction template, 3-parallel-extractor dispatch, CASP appraisal, thematic/framework/meta-ethnography/meta-aggregation synthesis, CERQual certainty assessment.
  - `skills/data-extraction/section-diagnostic-test-accuracy.md` (114 lines) — DTA-specific extraction template (2×2 table, QUADAS-2, threshold fields), bivariate/HSROC meta-analysis, GRADE-DTA, executable R script (mada, ggplot2).

  **Files modified:**
  - `skills/data-extraction/SKILL.md` — replaced the three inlined process sections with one-line read-and-follow references; all shared infrastructure (prerequisites, edge cases, batching, review-types path, case_report) retained inline.
