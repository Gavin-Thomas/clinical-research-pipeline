# Project Types Reference

This file defines how the clinical research pipeline adapts for each of the 8 supported project types. SKILL.md files read `project_type` from `project.yaml` and consult this reference to determine type-specific behavior at each stage.

---

## 1. systematic_review

The default pipeline. All stages execute as documented in the stage SKILL.md files without modification.

### Stage-by-Stage Adaptations

- **Stage 1 (Literature Search):** Standard PICO/PICOS framework development.
- **Stage 2 (Research Question):** Standard search strategy across all specified databases.
- **Stage 3 (Inclusion/Exclusion):** Standard inclusion/exclusion criteria. Screening decisions use INCLUDE/EXCLUDE/UNCERTAIN.
- **Stage 4 (Database Search Build):** Standard critical appraisal with appropriate risk-of-bias tools.
- **Stage 5 (Abstract Screening):** 3 extractors perform independent extraction. Quick Review (severity 1-4) / Full Review (severity 5-7) thresholds apply.
- **Stage 6 (Data Extraction & Synthesis):** Meta-analysis performed per project setting. Narrative synthesis accompanies quantitative results.
- **Stage 7 (Manuscript Writing):** PRISMA 2020 reporting guideline.

---

## 2. scoping_review

Broader in scope than a systematic review. Aims to map the available evidence rather than answer a focused clinical question.

### Stage-by-Stage Adaptations

- **Stage 1 (Literature Search):** Standard, but research questions are broader (PCC framework: Population, Concept, Context).
- **Stage 2 (Research Question):** Standard, typically with broader search terms reflecting the exploratory nature.
- **Stage 3 (Inclusion/Exclusion):** Broader inclusion criteria. Eligibility is more expansive to capture the breadth of available literature.
- **Stage 4 (Database Search Build):** Standard critical appraisal (optional per scoping review methodology; follows project setting).
- **Stage 5 (Abstract Screening):** Charting process replaces standard extraction. Classification decisions use **MAP / OUT_OF_SCOPE / UNCERTAIN** instead of INCLUDE / EXCLUDE. Data is charted into thematic categories rather than extracted for pooling.
- **Stage 6 (Data Extraction & Synthesis):** **Narrative synthesis only.** No meta-analysis is performed regardless of project setting. Results are presented as thematic maps, frequency counts, and descriptive summaries.
- **Stage 7 (Manuscript Writing):** **PRISMA-ScR** (PRISMA Extension for Scoping Reviews) reporting guideline.

---

## 3. rapid_review

A streamlined systematic review with methodological shortcuts to accelerate delivery. All deviations from full systematic review methodology are explicitly documented.

### Stage-by-Stage Adaptations

- **Stage 1 (Literature Search):** Standard, with tighter scope to facilitate rapid completion.
- **Stage 2 (Research Question):** May use fewer databases or date restrictions. All deviations documented.
- **Stage 3 (Inclusion/Exclusion):** Standard criteria. May use single-screener with verification sample.
- **Stage 4 (Database Search Build):** Standard, though abbreviated tools may be used.
- **Stage 5 (Abstract Screening):** **Single extractor** (no independent dual/triple extraction). Quick Review thresholds apply to **ALL stages** (no Full Review escalation).
- **Stage 6 (Data Extraction & Synthesis):** **Abbreviated synthesis.** Quantitative pooling permitted but narrative summary is condensed.
- **Stage 7 (Manuscript Writing):** **PRISMA 2020 with deviations noted.** A dedicated **Limitations section** is required in the manuscript, explicitly listing all methodological shortcuts taken and their potential impact on findings.

---

## 4. original_research

Primary data collection study (observational or interventional). The pipeline repurposes early stages for protocol development and pivots to data collection at manual handoff.

### Stage-by-Stage Adaptations

- **Stage 1 (Literature Search):** Standard. Develops research question and hypothesis.
- **Stage 2 (Research Question):** Standard. Background literature search to establish gaps and inform study design.
- **Stage 3 (Inclusion/Exclusion → Protocol Design):** **Modified stage.** Methodologist and PI collaboratively design the study protocol. Output: `03_inclusion_exclusion/study_protocol.md`.
- **Stage 4 (Database Search → Instrument Development):** **Modified stage.** Methodologist builds data collection instruments (surveys, case report forms, interview guides). Output: `04_database_search/data_collection_instruments.md`.
- **Manual Handoff:** Shifts to data collection. The research team collects data per the protocol. Raw data files are placed in `06_data_extraction/raw_data/` upon completion.
- **Stage 5 (Abstract Screening → Data Cleaning):** **Single-pass data cleaning.** Statistician performs data validation, coding, and preparation for analysis. No multi-extractor workflow.
- **Stage 6 (Data Extraction → EDA + Analysis):** **Statistician performs exploratory data analysis (Step 2b) then pre-specified statistical analysis.** EDA informs parametric vs. non-parametric choices and flags protocol deviations for PI review.
- **Stage 7 (Manuscript Writing):** Uses **STROBE** (observational studies) or **CONSORT** (randomized trials) reporting guideline as appropriate to the study design.

---

## 5. case_report

Clinical case presentation. The pipeline is significantly abbreviated, focusing on topic framing and manuscript preparation.

### Stage-by-Stage Adaptations

- **Stage 1 (Literature Search):** Standard. Identifies the clinical significance and learning points of the case.
- **Stage 2 (Research Question):** Standard. Literature search to contextualize the case within existing evidence.
- **Stages 3-5:** **Skipped entirely.** No screening, quality assessment, or data extraction stages.
- **Manual Handoff:** Single manual handoff. Clinical materials (patient records, imaging, lab results, clinical photographs) are placed in `full_texts/`.
- **Stage 6 (Case Structuring):** **PI and Manuscript Writer** collaboratively structure the case presentation. Organizes clinical timeline, diagnostic workup, management decisions, and outcomes into a coherent narrative.
- **Stage 7 (Manuscript Writing):** **CARE guidelines** (CAse REport) reporting guideline.

---

## 6. meta_analysis

Full systematic review pipeline with an extended quantitative synthesis phase. All systematic review stages apply, plus additional statistical analyses after Stage 6.

### Stage-by-Stage Adaptations

- **Stages 1-5:** Identical to `systematic_review`. All standard procedures apply (3 extractors, Quick/Full Review thresholds, standard screening criteria).
- **Stage 6 (Data Extraction & Synthesis):** Standard systematic review synthesis, plus the **Statistician performs extended meta-analytic procedures:**
  - **Effect size models:** Fixed-effects and random-effects models.
  - **Forest plots:** Generated for each primary and secondary outcome.
  - **Heterogeneity assessment:** I² statistic, Cochran's Q test, tau² estimate, prediction intervals.
  - **Publication bias:** Funnel plots with visual inspection, Egger's regression test for asymmetry.
  - **Sensitivity analyses:** Leave-one-out analysis, subgroup analyses by pre-specified moderators.
  - **Output:** `meta_analysis_results.md` containing all statistical results, plots, and interpretation.
- **Stage 7 (Manuscript Writing):** **PRISMA 2020 with meta-analysis-specific items** included (per the PRISMA 2020 checklist items designated for reviews with meta-analysis).

---

## 7. qualitative_synthesis

Systematic synthesis of qualitative research findings. Used when the review question concerns patient experience, perspectives, acceptability, or meanings rather than quantifiable outcomes. Produces conceptual or thematic syntheses, not statistical estimates.

**Common synthesis methods:** Thematic synthesis (Thomas & Harden), framework synthesis, meta-ethnography (Noblit & Hare), meta-aggregation (JBI approach).

**When to choose this type:** Use when your question is framed around "how", "why", or "what it is like" — e.g., patient experience of a condition or treatment, barriers and facilitators to care, acceptability of an intervention.

**Framework:** PICo (Population, phenomenon of Interest, Context) rather than PICO.

**Reporting guideline:** ENTREQ (Enhancing Transparency in Reporting the Synthesis of Qualitative Research).

### Stage-by-Stage Adaptations

- **Stage 1 (Literature Search):** Standard landscape search. Librarian additionally searches for qualitative study terminology (phenomenology, grounded theory, ethnography, thematic analysis, interpretive description, action research). Key review databases include CINAHL (strong qualitative nursing/health literature).

- **Stage 2 (Research Question):** Uses **PICo framework** (Population, phenomenon of Interest, Context) instead of PICO. The PI frames the question around experience, perspectives, or meanings. Research questions begin with "What are...", "How do...", or "What is the experience of..." rather than "What is the effect of...". Write PICo components to `research_question.md` in place of PICO.

- **Stage 3 (Inclusion/Exclusion):** Inclusion criteria are broad regarding study design:
  - **Include:** All qualitative primary studies — phenomenological studies, grounded theory studies, ethnographies, narrative inquiries, case studies, thematic analyses, action research, interpretive description, mixed-methods studies (qualitative component extractable).
  - **Exclude:** Quantitative-only studies, systematic reviews of quantitative evidence, opinion pieces without data, editorials.
  - Outcome criteria are replaced by: "Reports data relevant to the phenomenon of interest" — i.e., participant perspectives, experiences, or meanings on the topic.
  - Time and language restrictions apply as in other review types.

- **Stage 4 (Database Search Build):** Search strategy includes a qualitative study design filter block (per relevant database):
  - PubMed/MEDLINE: qualitative research [MeSH], interviews [MeSH], focus groups [MeSH], grounded theory, phenomenolog*, ethnograph*, thematic analysis, narrative research
  - Ovid: as above with `/` and `.mp.` syntax
  - Also include CINAHL as a required database (add to default databases list if not user-specified)
  - Omit randomized controlled trial and clinical trial filters (these would exclude the evidence base)

- **Stage 5 (Abstract Screening):** Standard triple-independent screening. Data Extractors apply the broad qualitative inclusion criteria. Key judgment: does the abstract report qualitative primary data (interviews, focus groups, observations, documents) relevant to the phenomenon of interest?

- **Stage 6 (Data Extraction & Synthesis):** **Qualitative thematic synthesis replaces quantitative data extraction.** The process:
  1. **Data Extractor agents** extract verbatim participant quotes and author interpretive summaries from each included study — using a template with columns: `study_id`, `author_year`, `design`, `population_n`, `setting`, `data_collection`, `analytic_approach`, `casp_score`, `raw_themes_verbatim`, `author_interpretations`, `notes`.
  2. **Methodologist** performs CASP Qualitative Checklist appraisal for each included study (10 questions, score 0–10). Studies are not excluded for low quality but quality informs confidence in findings.
  3. **Statistician role is replaced by a second Methodologist** who performs thematic synthesis per the user's chosen `synthesis_approach`:
     - **thematic** (default): (a) line-by-line coding of verbatim data, (b) development of descriptive themes, (c) generation of analytical themes going beyond study findings.
     - **framework**: Apply a pre-specified or inductively derived conceptual framework; map themes from included studies to framework domains.
     - **meta_ethnography**: Translate study concepts across studies using Noblit & Hare phases: reciprocal translation, refutational synthesis, or lines of argument.
     - **meta_aggregation** (JBI): Aggregate findings into categories and synthesize into synthesised findings; rate confidence in each synthesised finding.
  4. **Output files:**
     - `06_data_extraction/extracted_quotes.csv` — verbatim data and author interpretations per study
     - `06_data_extraction/casp_appraisal.md` — quality scores per study
     - `06_data_extraction/synthesis_report.md` — thematic/framework/meta-ethnographic synthesis with analytical themes, illustrative quotes, and ConQual (or CERQual) confidence assessment
  5. **CERQual assessment** (GRADE equivalent for qualitative evidence): Rate confidence in each synthesised finding on: methodological limitations, coherence, adequacy of data, relevance. Output: Summary of Qualitative Findings table.
  - **No statistical analysis is performed.** No forest plots, no pooled estimates, no heterogeneity statistics. Python/R analysis scripts are not generated.

- **Stage 7 (Manuscript Writing):** **ENTREQ** (Enhancing Transparency in Reporting the Synthesis of Qualitative Research, 21-item checklist) reporting guideline. Key sections differ from quantitative reviews:
  - Methods: Synthesis approach (thematic synthesis, meta-ethnography, etc.), sampling strategy for included studies, data extraction process, analytic process
  - Results: Analytical themes with illustrative participant quotes, CERQual confidence assessment table, Summary of Qualitative Findings
  - Discussion: Transferability of findings, researcher reflexivity note (note: the pipeline does not have subjective positionality — flag this as a transparency item for the PI to add)
  - Appendix: ENTREQ checklist, CASP scores per study, search strategy

---

## 8. diagnostic_test_accuracy

Systematic review of the diagnostic performance of a clinical test (index test) against a reference standard (gold standard). Used to evaluate imaging, biomarkers, clinical scoring systems, or any test where the question is "how accurately does this test identify the condition?"

**When to use:** Choose this type when the primary outcome is diagnostic accuracy — sensitivity, specificity, likelihood ratios, area under the ROC curve — rather than treatment effect. Common examples: dermoscopy for melanoma, serum biomarkers for autoimmune conditions, clinical scales for disease severity.

**PICOS framing:** Population (patients suspected of having the condition), Index test (the test being evaluated), Comparator (reference standard / gold standard, or head-to-head index test), Outcome (sensitivity, specificity, LR+, LR−, diagnostic odds ratio, AUC), Study design (cross-sectional or cohort studies applying both index test and reference standard).

**Reporting guideline:** STARD 2015 (Standards for Reporting Diagnostic accuracy Studies, 30-item checklist).

### Stage-by-Stage Adaptations

- **Stage 1 (Literature Search):** Standard landscape search. Librarian additionally searches for DTA terminology: "diagnostic accuracy", "sensitivity and specificity", "receiver operating characteristic", "ROC curve", "QUADAS", "STARD". Include the Cochrane Handbook chapter on DTA reviews as a methodological reference if needed.

- **Stage 2 (Research Question):** PI frames the question using PICOS (not PICO). The "C" is the reference standard, not a treatment comparator. Example frame: "In [population], how accurate is [index test] compared to [reference standard] for diagnosing [condition]?" Key decisions: acceptable spectrum of disease severity, partial verification allowance (will patients without the condition also receive the reference standard?), thresholds of interest (single cut-point vs. multiple thresholds).

- **Stage 3 (Inclusion/Exclusion):** Methodologist applies DTA-specific eligibility:
  - **Include:** Studies applying both the index test and the reference standard independently to participants; cross-sectional, retrospective cohort, and prospective designs; case-control designs flagged (spectrum bias concern, noted in limitations).
  - **Exclude:** Studies without a reference standard; studies using the index test result to decide who receives the reference standard (partial verification bias); studies with only a sub-sample verified (differential verification bias).
  - Spectrum considerations: Distinguish between "case-control" designs (selected diseased and healthy controls — overestimates accuracy) vs. consecutive/random cohort designs (preferred).
  - Flag PROSPERO registration if a full systematic review is intended.

- **Stage 4 (Database Search Build):** Standard databases plus diagnostic-specific search blocks:
  - Diagnostic accuracy filter block: `sensitivity[tiab] OR specificity[tiab] OR "predictive value"[tiab] OR "ROC curve"[MeSH] OR "diagnostic accuracy"[tiab] OR "likelihood ratio"[tiab]`
  - Methodological filter: `"cross-sectional studies"[MeSH] OR "diagnostic test"[tiab]`
  - Do NOT use RCT or intervention filters.

- **Stage 5 (Abstract Screening):** Standard triple-independent screening. Data Extractors apply DTA inclusion criteria. Key UNCERTAIN triggers: unclear if both index test and reference standard were applied; unclear patient spectrum (case-control vs. consecutive cohort); threshold information absent.

- **Stage 6 (Data Extraction & Synthesis):** DTA-specific extraction template and analysis.
  - **Extraction template** includes: study design (prospective/retrospective, consecutive/non-consecutive), patient spectrum description, index test details (test description, cut-off threshold, who interpreted it, blinding to reference standard), reference standard details (test, who applied it, blinding to index test), 2×2 table (TP/FP/TN/FN or derivable from sensitivity+specificity+prevalence), disease prevalence, number excluded, time interval between index test and reference standard, **QUADAS-2 domains** (patient selection, index test, reference standard, flow and timing — each rated Low/High/Unclear risk of bias, with applicability concern rating).
  - **Quality appraisal:** QUADAS-2 (Quality Assessment of Diagnostic Accuracy Studies) — 4 bias domains + 3 applicability domains. Applied by each Data Extractor; reconciled by Methodologist.
  - **Synthesis:** Statistician performs:
    1. **Descriptive:** Sensitivity/specificity forest plots (one plot per metric) with 95% CIs per study.
    2. **Bivariate model** (Reitsma et al.) for pooling: accounts for the negative correlation between sensitivity and specificity across studies; reports pooled sensitivity, pooled specificity, and the 95% confidence region on the SROC plane.
    3. **Hierarchical SROC (HSROC) curve** (Rutter & Gatsonis): when threshold effect is present (Spearman correlation between logit sensitivity and logit (1-specificity) > 0.6); reports AUC of the SROC curve.
    4. **Heterogeneity:** I² separately for sensitivity and specificity; investigate sources via meta-regression (study design, patient spectrum, threshold, blinding status).
    5. **Publication bias assessment** (if ≥10 studies): Deeks' funnel plot asymmetry test.
    6. **GRADE:** Apply GRADE-DTA framework — start at low certainty (observational design inherent) and upgrade/downgrade per standard domains.
  - **Code requirements (R):** Use `mada` package (functions: `reitsma()` for bivariate model, `ROCellipse()` for SROC plane, `SROCsens()` for HSROC); also `meta` for forest plots; save `sensitivity_forest.png`, `specificity_forest.png`, `sroc_curve.png`, `quadas2_traffic_light.png` to `06_data_extraction/analysis_scripts/outputs/`. Save script to `06_data_extraction/analysis_scripts/dta_analysis.R`.
  - **Output files:** `extracted_data.csv` (with 2×2 table), `quadas2_appraisal.md`, `synthesis_report.md`, `dta_analysis.R`, `meta_analysis_results.md` (pooled estimates + SROC).

- **Stage 7 (Manuscript Writing):** **STARD 2015** (30-item) reporting guideline. Key differences from intervention reviews:
  - Methods must specify: index test execution protocol, reference standard protocol, blinding of test interpreters, time interval between tests, thresholds considered, statistical method for pooling (bivariate / HSROC).
  - Results must include: STARD flow diagram, study characteristics table including QUADAS-2 traffic light plot (Figure 1), sensitivity/specificity forest plots (Figures 2–3), SROC plot (Figure 4), GRADE-DTA summary of evidence table.
  - Discussion: Address spectrum bias if case-control studies included; address partial verification bias; clinical utility section (pre-test vs. post-test probability, Fagan nomogram for selected LR values if clinically useful).
