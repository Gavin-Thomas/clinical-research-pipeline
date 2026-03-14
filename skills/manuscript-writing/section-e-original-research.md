# Section E: Original Research Manuscript

_For `original_research` projects only. Uses STROBE (observational cohort/cross-sectional/case-control) or CONSORT (RCT) reporting standard. No PRISMA flow, no search strategy, no systematic screening step. Source data is the primary collected dataset, not extracted literature._

### E1: Load or Fetch Journal Guidelines and Determine Reporting Standard

First, check if `00_journal_guidelines/guidelines_summary.md` exists (pre-fetched at project setup). If yes, read it and use it — do NOT re-fetch. If no, use WebFetch to retrieve guidelines from `journal_guidelines_url` in `project.yaml` and write the summary to `00_journal_guidelines/guidelines_summary.md`. Extract:
- Word limits (abstract and main text)
- Required sections and order
- Reference style
- Structured abstract format
- Table/figure limits and format requirements
- Supplementary materials policy (e.g., is the full statistical analysis plan accepted as a supplement?)
- CONSORT or STROBE checklist submission requirement

If `journal_guidelines_url` is blank and no pre-fetched guidelines exist: print "Journal guidelines URL not set — using IMRAD defaults. Please paste journal-specific requirements if available." and proceed.

### E1a: Journal Compliance Pre-Validation

Run the same pre-validation as described in `section-b-quantitative.md` Step 1a. Write results to `07_manuscript/journal_compliance.md`. Pass Writer Instructions to all agents in Step E2.

**Determine reporting standard from `project.yaml`:**
- `review_config.reporting_standard: CONSORT` → RCT; use CONSORT 2010 (25-item checklist)
- `review_config.reporting_standard: STROBE` → Observational (cohort, case-control, or cross-sectional); use STROBE 2007 (22-item checklist)
- Not set → infer from `study_protocol.md`: look for randomization language (→ CONSORT) or absence of randomization (→ STROBE). Print inference to the user.

### E2: Dispatch Writer A, Writer B, and Statistician in Parallel

**Writer A agent prompt (Introduction + Methods):**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft the INTRODUCTION and METHODS sections for an original research manuscript.

REPORTING STANDARD: [CONSORT 2010 / STROBE 2007 — from project.yaml]
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
STUDY TYPE: Original primary research (NOT a systematic review — no search strategy, no PRISMA flow)

SOURCE MATERIALS:
- Landscape report (background only): [Read 01_literature_search/landscape_report.md]
- Research question / objectives: [Read 02_research_question/research_question.md]
- Study protocol (design, population, procedures): [Read 03_inclusion_exclusion/study_protocol.md]
- Data collection form (variable definitions): [Read 06_data_extraction/data_collection_form.csv]
- Analysis summary: [Read 06_data_extraction/synthesis_report.md — methods section only]

INTRODUCTION should:
- Establish clinical context and disease burden (from landscape report and external citations)
- Summarize current evidence and identify the specific knowledge gap this study addresses
- State the primary study objective and, if applicable, primary hypothesis
- State secondary objectives if pre-specified
- Keep to 3–5 paragraphs; do NOT describe study results here

METHODS must follow [CONSORT / STROBE] and include:
1. Study design (e.g., "prospective cohort study", "randomised controlled trial") with study period and setting
2. Participants: eligibility criteria (inclusion and exclusion), recruitment source and method, enrollment dates
   - For RCT (CONSORT): allocation ratio, randomization method (sequence generation, allocation concealment, blinding)
   - For observational (STROBE): matching or selection criteria for comparison groups if applicable
3. Variables: primary exposure/intervention and control, primary outcome with measurement instrument and timing, secondary outcomes, confounders/covariates, and their operational definitions (reference data_collection_form.csv for field definitions)
4. Data sources and measurement: data collection instruments, data entry process, missing data handling approach
5. Bias: anticipated sources of bias and how they were addressed (e.g., selection bias, information bias, confounding)
6. Sample size: calculation (power, α, expected effect size, SD or event rate) or justification for feasibility sample
7. Statistical methods: as pre-specified in study_protocol.md — describe primary analysis, secondary analyses, subgroup analyses, sensitivity analyses; state software and version; describe how missing data were handled (complete-case, multiple imputation, etc.)
   - For RCT: state intention-to-treat as primary analysis; per-protocol as sensitivity analysis if applicable
   - For observational: state whether confounder adjustment was performed and method (regression, propensity score, etc.)
8. Ethics: IRB/REC approval number, informed consent process, trial registration number (ClinicalTrials.gov/other) if applicable

IMPORTANT: Do NOT include a database search strategy section. Do NOT describe abstract screening. Do NOT include a PRISMA flow. This is original primary research, not a systematic review.

OUTPUT: Introduction and Methods sections in markdown.
```

**Writer B agent prompt (Results + Discussion):**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft the RESULTS and DISCUSSION sections for an original research manuscript.

REPORTING STANDARD: [CONSORT 2010 / STROBE 2007 — from project.yaml]
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
STUDY TYPE: Original primary research

SOURCE MATERIALS:
- Research question / objectives: [Read 02_research_question/research_question.md]
- Landscape report (for Discussion — comparison with existing literature): [Read 01_literature_search/landscape_report.md]
- Analysis report (contains all results): [Read 06_data_extraction/synthesis_report.md]
- Cleaned dataset summary: [Read 06_data_extraction/cleaned_data.csv — first 20 rows for variable structure]
- Analysis scripts (for result values): [Check 06_data_extraction/analysis_scripts/ for any output CSVs]

RESULTS must include:
1. Participant flow:
   - For RCT (CONSORT item 13): numbers assessed for eligibility, excluded (with reasons), randomized, allocated, lost to follow-up, analysed per group — formatted as CONSORT flow diagram text
   - For observational (STROBE item 13): numbers considered, eligible, and included; for cohort: follow-up duration and numbers completing follow-up; for case-control: matching ratios
2. Baseline characteristics (Table 1): one row per variable, columns by study group (if applicable); report continuous variables as mean ± SD or median (IQR); categorical as n (%). Include all pre-specified confounders. Do NOT p-value test baseline characteristics (per CONSORT/STROBE guidance — baseline differences are pre-randomization for RCTs, not hypothesis tests)
3. Primary outcome results:
   - State number of events or outcome values by group
   - Report unadjusted and adjusted estimates with 95% CI and exact p-value
   - For RCT: absolute risk reduction/NNT if binary outcome; MD or SMD if continuous
   - For observational: OR, RR, or HR with 95% CI from the primary regression model
4. Secondary outcome results: one brief paragraph or table per secondary outcome; same reporting format
5. Subgroup analyses (if pre-specified): forest plot of subgroup estimates by pre-specified subgroup variable; report interaction p-value, not individual subgroup p-values
6. Sensitivity analyses: state what was varied and how results changed; flag any substantial differences from primary analysis
7. Missing data: state number and pattern of missing values for the primary outcome and key predictors; if multiple imputation used, report pooled estimate and comparison with complete-case

DISCUSSION should:
- Summarize the key finding in 1–2 sentences referencing the primary outcome result
- Compare findings with the existing literature cited in landscape_report.md (support or contradict? — cite specific studies)
- Discuss biological or mechanistic plausibility (if applicable)
- Strengths: study design, sample characteristics, pre-registration, comprehensive follow-up, etc.
- Limitations: potential sources of bias (selection, information, confounding); generalizability constraints; sample size if underpowered; observational nature if applicable (cannot establish causation)
- Clinical implications: who should act on this finding, and what action is appropriate?
- Future research directions: what is the next unanswered question?
- Conclusion: one short paragraph — restate primary finding and its clinical significance

IMPORTANT: Do NOT reference a PRISMA flow, a search strategy, or systematic review screening. Do NOT use the phrase "included studies" — this is primary data collection, not a literature review.

OUTPUT: Results and Discussion sections in markdown.
```

**Statistician agent prompt (Tables + Figures + Statistical text):**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Prepare all tables, figures, and statistical reporting text for an original research manuscript.

REPORTING STANDARD: [CONSORT 2010 / STROBE 2007]
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines — table/figure requirements]
STUDY TYPE: Original primary research

SOURCE MATERIALS:
- Study protocol: [Read 03_inclusion_exclusion/study_protocol.md]
- Cleaned dataset: [Read 06_data_extraction/cleaned_data.csv]
- Analysis report: [Read 06_data_extraction/synthesis_report.md]
- Existing analysis scripts (if any): [List files in 06_data_extraction/analysis_scripts/]

Prepare:
1. **Table 1: Baseline / Participant Characteristics** — rows = variables (demographics, clinical characteristics, confounders), columns = study group(s) + total; continuous: mean ± SD or median (IQR); categorical: n (%); NO p-values for baseline comparisons in RCTs
2. **Primary analysis result table** — effect estimate, 95% CI, p-value for unadjusted and adjusted models; include covariates in adjusted model with their coefficients/estimates if regression table format appropriate
3. **Secondary outcome summary table** (if ≥3 secondary outcomes)
4. **Participant flow diagram** (as structured text):
   - For RCT: CONSORT flow (assessed → randomized → allocated → follow-up → analysed)
   - For observational: STROBE flow (screened/eligible → included → follow-up if cohort)
5. **Figure(s)** as needed: Kaplan-Meier curve (if time-to-event outcome), forest plot of subgroup estimates (if pre-specified subgroup analyses), scatter plot or box plots for exploratory visualizations — produce as executable Python or R code saved to `06_data_extraction/analysis_scripts/`
6. **Statistical reporting text snippets** for Writer B to embed: pre-formatted sentences with exact values for primary outcome, each secondary outcome, and each sensitivity analysis

Generate (or verify existing) executable analysis script:
- If `06_data_extraction/analysis_scripts/primary_analysis.py` (or `.R`) does not exist, write it now
- Script must: read from `06_data_extraction/cleaned_data.csv`, run the primary analysis as pre-specified in `study_protocol.md`, produce all required output files, print a results summary to stdout
- Save any new or updated scripts to `06_data_extraction/analysis_scripts/`

OUTPUT: All tables, flow diagram text, figure code, and statistical text in markdown.
```

### E3: Merge Step

Dispatch Writer A as merge agent with the same prompt structure as section-b-quantitative.md Step 3, but with original-research-specific self-check:

**SELF-CHECK (required before writing output to disk):**

Grammar & Writing Quality:
- [ ] Past tense throughout Methods and Results; present tense for Discussion of established facts
- [ ] No first-person voice unless journal guidelines permit it
- [ ] All abbreviations defined at first use
- [ ] Numbers: spelled out below 10, numerals ≥10, always numerals with units

Structural Completeness:
- [ ] All required [CONSORT / STROBE] sections present — no mandatory items are missing
- [ ] Abstract matches journal format (structured/unstructured) and is within word limit
- [ ] Main text is within word limit
- [ ] No PRISMA flow, no systematic search strategy section — this is primary research ✗✗ if present

Citation & Reference Integrity:
- [ ] Every in-text citation has a corresponding reference; no orphaned references
- [ ] No `[CITATION NEEDED]` flags remain unresolved

Statistical Reporting:
- [ ] Primary outcome reported with effect estimate, 95% CI, and exact p-value ✗✗
- [ ] Adjusted and unadjusted estimates both reported for observational studies
- [ ] For RCT: absolute risk difference and NNT/NNH reported in addition to relative measures
- [ ] Baseline characteristics table (Table 1) present and does not include p-values for RCT comparisons
- [ ] Subgroup analyses reported with interaction p-value, not individual subgroup p-values
- [ ] Missing data pattern described; imputation method stated if used

Reporting Guideline Compliance:
- [ ] [CONSORT / STROBE] checklist items are all addressed — cross-check each item
- [ ] Study registration number cited in Methods (if applicable)
- [ ] IRB/ethics approval cited in Methods

Tables & Figures:
- [ ] Table 1 (participant characteristics) present and numbered as Table 1 ✗✗
- [ ] Every table and figure referenced in the text by number
- [ ] All figure legends are self-contained

If any self-check item fails, correct it before producing final output.

OUTPUT: Complete manuscript in markdown + references in BibTeX format.

### E4: Write Output

Write manuscript to `07_manuscript/manuscript.md`.
Write references to `07_manuscript/references.bib`.

### E4b: Proofreader Pass

Dispatch a Proofreader agent for linguistic, structural, and formatting quality assurance.

**Proofreader agent prompt:**

```
[Insert Proofreader system prompt from agent-roles.md]

TASK: Proofread the following original research manuscript before peer review.

MANUSCRIPT: [Read 07_manuscript/manuscript.md]
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
REPORTING STANDARD: [CONSORT 2010 / STROBE 2007 — from project.yaml]

Perform a full proofread. In addition to standard checks, verify:
- No PRISMA flow, no search strategy section, no "included studies" language — this is primary research
- [CONSORT / STROBE] checklist items are all addressed
- Statistical results: effect estimate + 95% CI + exact p-value for every outcome
- Table 1 does NOT contain p-values for baseline comparisons (RCT requirement)
- Observational: causal language is appropriately hedged ("associated with", not "caused by")
- IRB approval and study registration cited in Methods
- Participant flow described (CONSORT flow or STROBE enrollment)

OUTPUT: Proofread report in the standard format.
```

Write to `07_manuscript/proofread_report.md`.

If Errors found: re-dispatch merge agent to apply corrections, then proceed.

### E5: Full Review

Dispatch Full Review per `references/reviewer-protocol.md`. Initialize `review_iteration = 0`.

**Review criteria for this stage:**

Reporting Guideline Compliance:
- Are all applicable [CONSORT 2010 / STROBE 2007] items addressed?
- Is study registration cited in Methods (if applicable)?
- Is IRB/ethics approval cited?
- Is the participant flow diagram present (CONSORT or STROBE format — NOT PRISMA)?

Results Quality:
- Does Table 1 contain baseline characteristics without p-values (RCT) or with appropriate group comparison (observational)?
- Are primary outcome results reported with both unadjusted and adjusted estimates (95% CI and exact p-value)?
- Are subgroup analyses reported with interaction p-values rather than individual group p-values?
- Is missing data handling documented?

Structural Checks:
- Does the manuscript contain any review-type language (PRISMA, "included studies", "search strategy", "abstract screening")? If so, REJECT ✗✗ — this is primary research, not a review.
- Does the Discussion correctly distinguish association from causation for observational designs?

On **REVISE** verdict: build REVISION REQUIRED table → re-dispatch merge agent for targeted fixes → increment `review_iteration` → re-dispatch reviewer.

On **REJECT** or `review_iteration ≥ 2` without APPROVE: escalate per `reviewer-protocol.md`.

### E6: Update project.yaml

Set `stages.manuscript_writing: completed`.

Print:

```
✅ Manuscript complete!

📄 Files:
- Manuscript: 07_manuscript/manuscript.md
- References: 07_manuscript/references.bib
- Analysis scripts: 06_data_extraction/analysis_scripts/

📋 NEXT STEPS:
1. Review Table 1 and primary results for accuracy against your raw data
2. Verify that IRB approval number and trial registration (if applicable) are correct
3. Prepare [CONSORT / STROBE] checklist as a supplementary file for submission
4. Upload to [journal submission portal]
```
