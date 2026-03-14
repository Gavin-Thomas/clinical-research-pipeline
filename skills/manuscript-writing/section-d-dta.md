# Section D: Diagnostic Test Accuracy Manuscript

_For `diagnostic_test_accuracy` projects only. Uses STARD 2015 reporting standard (30-item checklist). Reports pooled sensitivity/specificity from a bivariate random-effects model or HSROC curve. No relative risk or treatment-effect language._

### D1: Load or Fetch Journal Guidelines

First, check if `00_journal_guidelines/guidelines_summary.md` exists (pre-fetched at project setup). If yes, read it and use it — do NOT re-fetch. If no, use WebFetch to retrieve guidelines from `journal_guidelines_url` in `project.yaml` and write the summary to `00_journal_guidelines/guidelines_summary.md`. Extract:
- Word limits (abstract and main text)
- Required sections and order
- Reference formatting style
- Figure limits and format requirements (DTA manuscripts typically require ≥4 key figures: QUADAS-2 traffic light plot, sensitivity forest plot, specificity forest plot, SROC curve)
- Whether a completed STARD 2015 checklist is required as a supplementary file

If `journal_guidelines_url` is blank and no pre-fetched guidelines exist: print "Journal guidelines URL not set — using STARD 2015 defaults." and proceed.

### D1a: Journal Compliance Pre-Validation

Run the same pre-validation as described in `section-b-quantitative.md` Step 1a. Write results to `07_manuscript/journal_compliance.md`. Pass Writer Instructions to all agents in Step D2.

### D2: Dispatch Writers + Statistician in Parallel

**Writer A agent prompt (Introduction + Methods):**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft INTRODUCTION and METHODS for a diagnostic test accuracy systematic review manuscript.

REPORTING STANDARD: STARD 2015 (Standards for Reporting Diagnostic accuracy Studies — 30 items)
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]

SOURCE MATERIALS:
- Landscape report: [Read 01_literature_search/landscape_report.md]
- Research question / PICOS: [Read 02_research_question/research_question.md]
- Inclusion/exclusion criteria: [Read 03_inclusion_exclusion/criteria.md]
- Search strategy: [Read 04_database_search/search_strategy.md]
- Extracted data (QUADAS-2 columns): [Read 06_data_extraction/extracted_data.csv]

INTRODUCTION should:
- Establish clinical context: prevalence of the target condition, clinical consequences of misdiagnosis (false positives vs. false negatives)
- Describe the index test (technology, current clinical role, known advantages/limitations)
- Summarize existing evidence on the test's diagnostic performance from landscape report
- Identify the gap this review addresses (no prior meta-analysis, conflicting estimates, absence of subgroup data, etc.)
- State the review objective using PICOS framing: P=patients suspected of having [condition], I=[index test] at [threshold if specified], C=[reference standard], O=sensitivity, specificity, and associated accuracy metrics, S=cross-sectional and cohort studies applying both tests to same participants

METHODS should include (per STARD 2015 items 4–15):
- Registration (PROSPERO if applicable — STARD item 5) and protocol reference
- Eligibility criteria: patient spectrum (suspected condition, specify severity range), index test protocol (name, version, threshold used, whether blinded to reference standard result), reference standard (name, timing relative to index test), study design (cross-sectional or cohort applying both index test and reference standard to same patients; state that case-control designs were excluded to avoid spectrum bias)
- Information sources and search dates (STARD item 7)
- Search strategy summary (reference full strategy as online supplement — STARD item 8)
- Study selection: triple-independent screening by Data Extractors; conflicts resolved by consensus
- Data extraction: triple-independent by Data Extractors; Statistician compiled consensus data
- Risk of bias: QUADAS-2 (4 bias domains — patient selection, index test, reference standard, flow and timing; 3 applicability domains — same 3 — each rated Low/High/Unclear)
- Statistical analysis: bivariate random-effects model (Reitsma et al. 2005) as primary synthesis, implemented in R using `mada::reitsma()`; HSROC curve (Rutter & Gatsonis 2001, `mada::hsroc()`) used if significant threshold effect detected (Spearman r between logit-sensitivity and logit-(1-specificity) > 0.6); I² calculated separately for sensitivity and specificity; pooled LR+, LR−, DOR derived from bivariate model; Deeks' funnel plot and asymmetry test if ≥10 studies; sensitivity analysis excluding studies with any QUADAS-2 domain rated High risk; GRADE-DTA certainty assessment for pooled sensitivity and pooled specificity

OUTPUT: Introduction and Methods in markdown.
```

**Writer B agent prompt (Results + Discussion):**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft RESULTS and DISCUSSION for a diagnostic test accuracy systematic review manuscript.

REPORTING STANDARD: STARD 2015
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]

SOURCE MATERIALS:
- Research question: [Read 02_research_question/research_question.md]
- Landscape report (for Discussion — comparison with prior systematic reviews and accuracy studies): [Read 01_literature_search/landscape_report.md]
- Screening results: [Read 05_screening/screened_results.csv — INCLUDE rows only]
- Extracted data: [Read 06_data_extraction/extracted_data.csv]
- Synthesis report: [Read 06_data_extraction/synthesis_report.md]

RESULTS must include:
1. STARD flow diagram data (present as structured text): database yields per source, total retrieved, duplicates removed, screened by title/abstract, excluded with reasons, full-text assessed, excluded with specific reasons (spectrum bias, partial verification, wrong reference standard, insufficient data), final included studies
2. Study characteristics table (Table 1) — columns: Study (author, year), Country, Design, Sample size, Condition prevalence (n/N), Index test + threshold, Reference standard, Blinding (index/reference), QUADAS-2 overall
3. QUADAS-2 results: state these are presented graphically as a traffic light plot (Figure 1); describe the proportion of studies with Low/High/Unclear risk per domain in narrative
4. Diagnostic accuracy results:
   - Individual study sensitivity and specificity (state these are shown in forest plots, Figures 2–3)
   - Pooled sensitivity and specificity with 95% CI from bivariate model — state explicitly as proportions
   - Pooled LR+ and LR− with 95% CI, and DOR with 95% CI
   - I² for sensitivity and specificity separately with interpretation
   - Threshold effect test result (Spearman r value, p-value) and statement of whether HSROC model was applied
   - SROC curve with 95% confidence ellipse (Figure 4)
   - Deeks' funnel plot asymmetry (if ≥10 studies): t-statistic, p-value
   - Sensitivity analysis: direction and magnitude of change when high-RoB studies excluded
5. GRADE-DTA certainty table (Table 2): pooled sensitivity and specificity rows with domain-by-domain ratings and overall certainty (from synthesis_report.md)

DISCUSSION must include:
- Summary of pooled accuracy in clinical language: what does this sensitivity and specificity mean for identifying [condition] with [index test]?
- Clinical utility: frame LR+ and LR− as post-test probability shifts at a stated pre-test prevalence; reference Fagan nomogram (Figure 5) for illustration
- Comparison with prior systematic reviews and individual accuracy studies (cite specific work from landscape_report.md)
- Sources of heterogeneity: threshold variation, patient spectrum differences, reference standard variation, blinding; grounded in subgroup/sensitivity analysis results
- Methodological strengths (triple-independent extraction, QUADAS-2, GRADE-DTA, bivariate model)
- Limitations: number of studies, potential spectrum bias even among included studies, threshold variability preventing a single operating point recommendation, partial verification if present
- Implications for practice: under what pre-test probability and clinical scenario does this test meaningfully change management?
- Conclusion: concise summary of pooled accuracy and clinical relevance

OUTPUT: Results and Discussion in markdown.
```

**Statistician agent prompt (Tables + Figures + Statistical text):**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Prepare tables, figures, figure legends, STARD flow data, and statistical text snippets for a diagnostic test accuracy manuscript.

TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines — figure limits, table formatting]

SOURCE MATERIALS:
- Extracted data: [Read 06_data_extraction/extracted_data.csv]
- Synthesis report: [Read 06_data_extraction/synthesis_report.md]
- DTA analysis outputs: [Check 06_data_extraction/analysis_scripts/outputs/ for: sensitivity_forest.png, specificity_forest.png, sroc_curve.png, quadas2_traffic_light.png, deeks_funnel.png]

If PNG outputs exist, reference them by filename. If dta_analysis.R has not been run, generate or re-run the R script (see agent-roles.md Statistician DTA methods: `mada::reitsma()`, `mada::ROCellipse()`, `mada::hsroc()`).

Prepare:
1. Table 1 shell: Study Characteristics — columns: Study, Country, Design, N, Prevalence (n/N), Index test, Threshold, Reference standard, Index test blinding, Reference standard blinding
2. Table 2 shell: GRADE-DTA Summary of Findings — columns: Outcome (sensitivity / specificity), Pooled estimate [95% CI], No. of studies (N patients), Risk of bias, Inconsistency, Indirectness, Imprecision, Publication bias, Overall certainty
3. STARD flow diagram: structured text representation (databases → retrieved → duplicates removed → title/abstract screened → excluded (n, reason) → full-text assessed → excluded (n, reason per criterion) → included)
4. Figure 1 legend: "QUADAS-2 risk of bias and applicability assessment across included studies. Bars represent the proportion of studies rated low (green), high (red), or unclear (yellow) risk for each domain."
5. Figure 2 legend: "Forest plot of sensitivity for [index test] in detecting [condition] compared with [reference standard]. Pooled sensitivity estimated by bivariate random-effects model."
6. Figure 3 legend: "Forest plot of specificity."
7. Figure 4 legend: "Summary receiver operating characteristic (SROC) curve. Each point represents an individual study's sensitivity and (1–specificity). The ellipse represents the 95% confidence region around the summary operating point from the bivariate model."
8. Figure 5 legend: "Fagan nomogram illustrating the post-test probability of [condition] using the pooled LR+ ([value]) and LR− ([value]) at a stated pre-test probability of [X]%."
9. Statistical reporting text snippets: pooled sensitivity/specificity with 95% CI, I² for each, LR+/LR−/DOR with 95% CI, threshold effect test result, GRADE-DTA certainty ratings — formatted for direct embedding in Results section

OUTPUT: All tables, figure legends, STARD flow data, and statistical text snippets in markdown.
```

### D3: Merge and Self-Check

Dispatch Writer A as merge agent to combine Introduction, Methods, Results, Discussion, and Statistician outputs into a complete manuscript.

Use the standard merge prompt structure from Section B Step 3, but replace the SELF-CHECK block with the following DTA-specific version:

```
**SELF-CHECK (required before writing output to disk):**

STARD 2015 Compliance:
- [ ] STARD flow diagram data present with all required numbers (databases, deduplication, screening, full-text, included — with per-reason exclusion counts at full-text stage)
- [ ] Eligibility criteria specify patient spectrum, index test protocol (including blinding), reference standard, and justify exclusion of case-control designs
- [ ] Statistical methods describe bivariate model (Reitsma), threshold effect test (Spearman r), HSROC if used, I² separately for sensitivity and specificity, and Deeks' funnel if ≥10 studies
- [ ] GRADE-DTA certainty reported for pooled sensitivity AND pooled specificity separately

Results Accuracy:
- [ ] Pooled sensitivity and specificity reported as proportions with 95% CI (not as percentages unless the journal requires it — if percentages, state "e.g., 0.85 [95% CI 0.78–0.91]" format clearly)
- [ ] LR+, LR−, and DOR reported with 95% CI
- [ ] I² stated separately for sensitivity and specificity (not combined)
- [ ] Threshold effect test result (Spearman r and p-value) stated
- [ ] All 5 figures referenced by number in the main text (QUADAS-2 traffic light, sensitivity forest, specificity forest, SROC curve, Fagan nomogram)

Clinical Utility Framing:
- [ ] Discussion uses clinical language: sensitivity and specificity interpreted in context of false-positive and false-negative consequences for this specific condition
- [ ] LR+ and LR− used to frame post-test probability at a stated pre-test prevalence
- [ ] No relative risk, hazard ratio, or treatment-effect language (inappropriate for diagnostic accuracy)

General Quality:
- [ ] Consistent tense: past tense for Methods and Results; present tense for established facts and Discussion
- [ ] All abbreviations defined at first use (QUADAS-2, SROC, HSROC, DOR, LR+, LR−, AUC, CI, GRADE-DTA)
- [ ] All in-text citations have corresponding reference entries
- [ ] No `[CITATION NEEDED]` flags left unresolved
- [ ] Word count within journal limits

If any check fails, correct before producing final output.
```

Write output to `07_manuscript/manuscript.md` and `07_manuscript/references.bib`.

### D3b: Proofreader Pass

Dispatch a Proofreader agent for linguistic, structural, and formatting quality assurance.

**Proofreader agent prompt:**

```
[Insert Proofreader system prompt from agent-roles.md]

TASK: Proofread the following diagnostic test accuracy manuscript before peer review.

MANUSCRIPT: [Read 07_manuscript/manuscript.md]
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
REPORTING STANDARD: STARD 2015 (30 items)

Perform a full proofread. In addition to standard checks, verify:
- STARD 2015 items 1–30 checklist adherence
- Sensitivity/specificity are reported consistently (proportions vs. percentages — pick one and use throughout)
- All DTA abbreviations defined: QUADAS-2, SROC, HSROC, DOR, LR+, LR−, AUC, CI, GRADE-DTA, ESS
- No treatment-effect or relative-risk language (inappropriate for diagnostic accuracy)
- All 5 figures referenced by number: QUADAS-2 traffic light, sensitivity forest, specificity forest, SROC curve, Fagan nomogram
- Clinical utility framing uses LR+/LR− with pre-test probability, not just sensitivity/specificity

OUTPUT: Proofread report in the standard format.
```

Write to `07_manuscript/proofread_report.md`.

If Errors found: re-dispatch merge agent to apply corrections, then proceed.

### D4: Full Review + Update project.yaml

Dispatch Full Review per `references/reviewer-protocol.md`. Initialize `review_iteration = 0`.

**Review criteria (provide to each reviewer):**

STARD Compliance:
- Is the STARD 2015 flow diagram present with per-database yields and per-reason full-text exclusion counts?
- Does the Methods section describe QUADAS-2 (all 4 bias domains and 3 applicability domains)?
- Does the Methods section describe the bivariate model (Reitsma) and threshold effect test?
- Are pooled sensitivity and specificity reported with 95% CI from the bivariate model?
- Are I² values reported separately for sensitivity and specificity?
- Is GRADE-DTA certainty reported for both sensitivity and specificity?

Clinical Utility:
- Are LR+ and LR− reported and used in the Discussion to frame post-test probability?
- Is a Fagan nomogram or equivalent pre/post-test probability example present?
- Is the Discussion framed around clinical decision-making (which patients, at what pre-test probability, does this test change management)?

Accuracy:
- Are sensitivity and specificity reported as proportions or clearly labelled percentages?
- Is there any incorrect statistical language (e.g., OR interpreted as diagnostic accuracy; causal language for an observational diagnostic review)?

**Review iteration loop:**

On **APPROVE**: proceed to Step D5.

On **REVISE** (any reviewer):
- Build REVISION REQUIRED table (reviewer | STARD item | finding | location | required action)
- Re-dispatch merge agent to address all findings
- Update `07_manuscript/manuscript.md`
- Increment `review_iteration += 1`
- Re-dispatch Full Review with revision history

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Pause pipeline for PI resolution

### D5: Update project.yaml

Set `stages.manuscript_writing: completed`.

Print:

```
✅ Diagnostic test accuracy manuscript complete!

📄 Files:
- Manuscript: 07_manuscript/manuscript.md
- References: 07_manuscript/references.bib

📋 NEXT STEPS:
1. Review pooled sensitivity and specificity for clinical plausibility against individual study estimates
2. Confirm index test and reference standard descriptions are accurate
3. Add author names, affiliations, and contact details
4. Complete declarations (funding, conflicts of interest, ethics approval)
5. Complete the STARD 2015 checklist (30 items) as a supplementary file — required by most journals
6. Convert to submission format (Word/LaTeX)
7. Submit to [target_journal]!
```
