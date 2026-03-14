# Section A: Qualitative Synthesis Manuscript

_For `qualitative_synthesis` projects only. Uses ENTREQ reporting standard. No statistical pooling or forest plots._

### A1: Fetch Journal Guidelines

Same as Step 1 below — fetch guidelines from `journal_guidelines_url`. Note word limits, abstract format, and any qualitative-specific requirements (reflexivity statement, COREQ/ENTREQ compliance).

### A2: Dispatch Qualitative Writers in Parallel

**Writer A agent prompt (Introduction + Methods):**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft INTRODUCTION and METHODS for a qualitative evidence synthesis manuscript.

REPORTING STANDARD: ENTREQ (Enhancing Transparency in Reporting the Synthesis of Qualitative Research — 21 items)
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
SYNTHESIS APPROACH: [from project.yaml review_config.synthesis_approach — thematic | meta_ethnography | framework | meta_aggregation]

SOURCE MATERIALS:
- Landscape report: [Read 01_literature_search/landscape_report.md]
- Research question: [Read 02_research_question/research_question.md] (PICo: P, phenomenon of Interest, Context)
- Inclusion/exclusion criteria: [Read 03_inclusion_exclusion/criteria.md]
- Search strategy: [Read 04_database_search/search_strategy.md]
- CASP appraisal: [Read 06_data_extraction/casp_appraisal.md]

INTRODUCTION should:
- Establish the clinical/social context and why the phenomenon of interest matters
- Summarize what is known about the phenomenon from existing literature
- Identify the gap this synthesis addresses (why a qualitative synthesis, not a quantitative review)
- State the research question using PICo framing

METHODS should include (per ENTREQ items 1–17):
1. Synthesis approach with justification (e.g., "Thematic synthesis (Thomas & Harden) was chosen because...")
2. Inclusion criteria: population, phenomenon of interest, context, study designs (all primary qualitative designs)
3. Search strategy: databases searched (including CINAHL), date range, grey literature
4. Study selection: screening process, number of screeners
5. Data extraction: verbatim quotes and author interpretations extracted independently
6. Appraisal: CASP Qualitative Checklist — note that quality rarely excludes studies but informs CERQual
7. Synthesis method detail per approach:
   - thematic: line-by-line coding → descriptive → analytical themes (Thomas & Harden)
   - meta_ethnography: reciprocal/refutational translation → line-of-argument (Noblit & Hare)
   - framework: mapping to pre-specified framework
   - meta_aggregation: findings → categories → synthesized findings (JBI)
8. CERQual assessment: confidence in each synthesized finding assessed across 4 domains

OUTPUT: Introduction and Methods in markdown.
```

**Writer B agent prompt (Results + Discussion):**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft RESULTS and DISCUSSION for a qualitative evidence synthesis manuscript.

REPORTING STANDARD: ENTREQ
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
SYNTHESIS APPROACH: [from project.yaml review_config.synthesis_approach]

SOURCE MATERIALS:
- Research question: [Read 02_research_question/research_question.md]
- Landscape report (for Discussion — existing literature comparison): [Read 01_literature_search/landscape_report.md]
- Screening results: [Read 05_screening/screened_results.csv — INCLUDE rows only]
- Extracted quotes: [Read 06_data_extraction/extracted_quotes.csv]
- CASP appraisal: [Read 06_data_extraction/casp_appraisal.md]
- Synthesis report: [Read 06_data_extraction/synthesis_report.md]
- CERQual assessment: [Read 06_data_extraction/cerqual_assessment.md]

RESULTS must include:
1. PRISMA-ScR flow diagram data (searches → screened → full-text → included)
2. Study characteristics table (Table 1): study ID, country, design, participants, phenomenon studied, CASP score
3. Analytical themes/synthesized findings, each with:
   - Theme name and brief description
   - Supporting verbatim quotes (at minimum 2–3 per theme, with source study citation)
   - CERQual confidence rating (High / Moderate / Low / Very Low) with brief justification
4. CERQual Summary of Qualitative Findings table (Table 2): one row per synthesized finding — finding statement, studies contributing, CERQual assessment, explanation

DISCUSSION must include:
- Summary of key themes in relation to the research question (PICo)
- Comparison with existing quantitative and qualitative literature (cite specific studies from landscape_report.md)
- Methodological strengths (multi-database search, CASP appraisal, CERQual assessment)
- Limitations: search limitations, CASP quality variation, potential reviewer bias
- Reflexivity note: [flag with placeholder — PI must add first-person reflexivity statement per ENTREQ item 21]
- Implications for practice, policy, and future research
- Conclusion

OUTPUT: Results and Discussion in markdown.
```

### A3: Merge and Self-Check

Dispatch a merge agent (Writer A role) to combine Introduction, Methods, Results, Discussion into a complete manuscript with title page, abstract, keywords, declarations, references, and tables.

Apply the same SELF-CHECK block as Step 3 in Section B, with these additional qualitative-specific checks:

```
Qualitative-Specific Self-Check:
- [ ] ENTREQ items 1–21 are all addressable in the manuscript (check against ENTREQ checklist)
- [ ] Every analytical theme has ≥2 verbatim supporting quotes with source citations
- [ ] CERQual confidence ratings are present for every synthesized finding
- [ ] CERQual Summary of Qualitative Findings table (Table 2) is complete
- [ ] Reflexivity placeholder is flagged clearly for PI to complete (ENTREQ item 21)
- [ ] No statistical pooling or forest plots present (qualitative synthesis does not pool data)
- [ ] PRISMA-ScR flow used (not standard PRISMA) for screening flow reporting
- [ ] No unsupported causal language — qualitative findings describe perceptions/experiences, not causal effects
```

Write output to `07_manuscript/manuscript.md` and `07_manuscript/references.bib`.

### A4: Full Review + Update project.yaml

Dispatch Full Review per `references/reviewer-protocol.md` using the same review iteration loop as Step 5 in Section B. Initialize `review_iteration = 0`.

On APPROVE: set `stages.manuscript_writing: completed` in `project.yaml` and print the standard completion message.
