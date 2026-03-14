---
name: pipeline-orchestrator
description: |
  Use when the user wants to start a new research project, run the full research pipeline,
  manage a systematic review workflow, or coordinate multiple research stages.
  Trigger on phrases like "start a new review", "research pipeline", "new research project",
  "systematic review", "scoping review", "begin the research", "run the pipeline",
  "start a new study", "help me write a paper", "I want to research [topic]",
  "how do I start", "what should I study", "I want to do a literature review",
  "help me with a meta-analysis", "I want to publish a paper on", "can you help me
  write a review on", "I'm not sure what kind of review to do",
  "run the pipeline on", "set up a PRISMA review", "I need to do a meta-analysis on",
  "help me structure my systematic review protocol", "PROSPERO registration",
  "help me with a scoping review", "I want to do a rapid review",
  "help me write a systematic review", "start a PRISMA-compliant review".
---

# Research Pipeline Orchestrator

Coordinate the full clinical research pipeline from literature search to manuscript writing.

## Onboarding New Users

If the user seems unfamiliar with the pipeline (e.g., says "how do I start", "what is this", or asks a general question), briefly orient them before collecting project parameters:

```
This pipeline runs a full evidence synthesis workflow — from landscape search to
submission-ready manuscript — using a coordinated team of AI agents (Librarian, PI,
Methodologist, Data Extractors ×3, Statistician, Programmer, Writers ×2, Proofreader,
Independent Reviewers).

Supported outputs: PRISMA 2020 systematic reviews, PRISMA-ScR scoping reviews,
meta-analyses (random-effects pooling, GRADE, forest/funnel plots), STARD diagnostic
test accuracy reviews (QUADAS-2, bivariate/HSROC synthesis, SROC curves), ENTREQ
qualitative synthesis, STROBE original research, and CARE case reports. All review
cycles include dual-independent review with automated revision.

Two manual steps remain (though significantly reduced by API integrations):
  1. Executing database searches — the pipeline now auto-tests PubMed yield
     via E-utilities API and refines search strategies before you run them
  2. Retrieving full-text PDFs — the pipeline auto-checks Unpaywall for OA
     PDFs (typically finds 30-60%), so you only need to retrieve the rest

The pipeline also uses Semantic Scholar for citation snowballing and OpenAlex
for cross-database volume estimation. See references/api-integrations.md for
full API documentation and references/getting-started.md for the user guide.
```

If the user is unsure what type of review to do, refer them to the "Review Type Decision Guide" in `references/getting-started.md` and guide them interactively using the academic framing (question type, study design, journal expectations) before collecting parameters.

## Starting a New Project

Collect project parameters efficiently. If the user has already provided their PICO/topic in their request, extract those details without re-asking. Use AskUserQuestion to fill gaps only.

Required parameters:
1. **Project name** (will become the directory name)
2. **Project type** (systematic_review, scoping_review, rapid_review, original_research, case_report, case_series, meta_analysis, qualitative_synthesis, diagnostic_test_accuracy)
   - `case_report`: Single patient case with teaching value. Uses CARE guidelines.
   - `case_series`: Multiple patients (typically 3–20) sharing a common exposure, diagnosis, or outcome. Uses CARE guidelines with aggregate summary. Treated as a variant of `case_report` with additional cross-case comparison. If the user says "case series", set `project_type: case_series`.
   - `qualitative_synthesis`: Systematic synthesis of qualitative research — thematic synthesis, meta-ethnography, framework synthesis, meta-aggregation. Use when the question concerns patient experience, perspectives, or acceptability. Uses PICo framework and ENTREQ reporting.
   - `diagnostic_test_accuracy`: Systematic review of how accurately an index test identifies a condition compared to a reference standard. Use when the primary outcome is sensitivity, specificity, or AUC — not treatment effect. Uses PICOS framing, QUADAS-2 appraisal, bivariate/HSROC synthesis, and STARD 2015 reporting.
3. **PICO/PEO/PCC/PICo/PICOS components** (P, I/E, C, O; or for `diagnostic_test_accuracy`: P=suspected patients, I=index test, C=reference standard, O=diagnostic accuracy; or for `qualitative_synthesis`: P, phenomenon of Interest, Context) — or research topic if pre-PICO
4. **Target journal** (optional — can be "undecided"; if provided, fetch guidelines automatically at Stage 7)
5. **Journal guidelines URL** (optional — provide if you have it, otherwise the pipeline will fetch it)
6. **Databases to search** (default: pubmed, medline_ovid, embase — confirm Ovid/Embase institutional access)
7. **Date range** (default: last 10 years)

Optional advanced parameters (ask if relevant):
- **Subgroup analyses** (e.g., "by age group", "by RCT vs. observational") — informs Stage 6 extraction template
- **Sensitivity analyses** (e.g., "exclude high-RoB studies") — planned at Stage 3
- **Meta-analysis** (true/false — if true, Statistician generates pooled estimates and executable analysis code)
- **PROSPERO registration** (true/false — if true, pipeline produces a registerable protocol after Stage 3)
- **Network meta-analysis** (true/false — if true, Statistician adapts to NMA methods)
- **Clinical coding** (true/false — if true, Data Extractor assigns ICD-10/ICD-11, CPT, and SNOMED CT codes to diagnoses, procedures, and population descriptors during Stage 6 extraction; recommended for clinical dermatology, oncology, and procedure-based research topics)
- **Synthesis approach** (`qualitative_synthesis` only): `thematic` (default — Thomas & Harden), `framework`, `meta_ethnography` (Noblit & Hare), or `meta_aggregation` (JBI). Ask if the user has a preference; default to `thematic` if unsure.

If the user provides a vague research area without PICO components, help them decompose it: ask for the Population, Intervention/Exposure, Comparator (if any), and Outcome(s) directly using clinical terminology.

For `qualitative_synthesis`, help them frame a PICo question instead: Population (who), phenomenon of Interest (what experience or perspective), and Context (what setting or circumstance).

For `diagnostic_test_accuracy`, help them frame a PICOS question: Population (patients suspected of having the condition — specify spectrum of severity), Index test (the test being evaluated — name, protocol, threshold), Comparator (reference standard / gold standard test), Outcome (diagnostic accuracy: sensitivity, specificity, AUC, likelihood ratios), Study design (cross-sectional or cohort applying both index and reference standard). Ask whether partial verification is a concern (i.e., whether all patients received the reference standard) and whether multiple thresholds or a single cut-off are of interest.

### Create Project Structure

Create the full directory structure per `references/directory-conventions.md`.

Write `project.yaml`:

```yaml
project_name: "[user input]"
project_type: [user input]
topic: "[user input]"

pico:
  population: "[user input or TBD — refined in Stage 2]"
  intervention_exposure: "[user input or TBD — for qualitative_synthesis, use phenomenon of Interest]"
  comparator: "[user input or N/A — for qualitative_synthesis, use Context]"
  outcomes: "[user input or TBD — for qualitative_synthesis, omit or list perspectives of interest]"

target_journal: "[user input or undecided]"
journal_guidelines_url: "[user input or TBD]"

stages:
  literature_search: pending
  research_question: pending
  inclusion_exclusion: pending
  database_search: pending
  abstract_screening: pending
  data_extraction: pending
  manuscript_writing: pending

databases:
  - [user input list]

review_config:
  reporting_standard: [auto-select: systematic_review→PRISMA, scoping_review→PRISMA-ScR, rapid_review→PRISMA, original_research→STROBE, case_report→CARE, case_series→CARE, meta_analysis→PRISMA, qualitative_synthesis→ENTREQ, diagnostic_test_accuracy→STARD]
  n_cases: [case_series only: number of patients in the series — ask user; omit for other types]
  meta_analysis: [true if meta_analysis type OR if user requested pooling, false otherwise — always false for qualitative_synthesis and diagnostic_test_accuracy]
  network_meta_analysis: [true/false — always false for qualitative_synthesis and diagnostic_test_accuracy]
  diagnostic_model: [diagnostic_test_accuracy only: bivariate | hsroc — default bivariate; set hsroc if significant threshold effect detected; omit for other project types]
  date_range: "[user input]"
  prospero_registration: [true/false]
  subgroup_analyses: [list from user input, or []]
  sensitivity_analyses: [list from user input, or []]
  clinical_coding: [true/false — true enables ICD-10/ICD-11/CPT/SNOMED CT extraction in Stage 6]
  synthesis_approach: [qualitative_synthesis only: thematic | framework | meta_ethnography | meta_aggregation — omit for other project types]
```

## Resuming an Existing Project

If `project.yaml` already exists in the working directory:
1. Read it
2. Find the first stage with status `pending` or `in_progress`
3. Resume from that stage

Print the current project status:

```
📊 Project: [project_name]
📝 Type: [project_type]
🎯 Topic: [topic]
📰 Journal: [target_journal]

Stage Status:
  ✅ Literature Search: completed
  ✅ Research Question: completed
  🔄 Inclusion/Exclusion: in_progress
  ⏳ Database Search: pending
  ⏳ Abstract Screening: pending
  ⏳ Data Extraction: pending
  ⏳ Manuscript Writing: pending

Resuming from: Inclusion/Exclusion Criteria
```

## Pipeline Execution

For each stage, the orchestrator:

1. **Checks prerequisites**: Verifies required input files exist
2. **Updates status**: Sets stage to `in_progress` in `project.yaml`
3. **Dispatches stage agent**: Creates an Agent subagent whose prompt includes:
   - The full content of the relevant stage `SKILL.md` file (read from the plugin's skills directory)
   - The full content of `references/agent-roles.md`
   - The full content of `references/reviewer-protocol.md`
   - The full content of `references/project-types.md`
   - Instruction: "Execute the skill instructions above for the current project."
4. **Verifies output**: Checks that expected output files were created
5. **Updates status**: Sets stage to `completed` in `project.yaml`
6. **Proceeds or pauses**: Moves to next stage, or pauses at manual handoff points

### Stage Dispatch Template

```
You are executing a stage of the clinical research pipeline.

PROJECT DIRECTORY: [absolute path to project directory]

PROJECT CONFIGURATION:
[Insert full project.yaml content]

PRIOR STAGE OUTPUTS (inject completed outputs only — skip sections for stages not yet complete):
[Before dispatching each stage, the orchestrator reads and injects all available prior stage
outputs according to the table below. This ensures each stage agent begins with full pipeline
context and does not need to make assumptions about earlier decisions.]

  Stage 2+ dispatch: Insert 01_literature_search/landscape_report.md
    Label: "LANDSCAPE SURVEY (Stage 1 output):"

  Stage 3+ dispatch: Also insert 02_research_question/research_question.md
    Label: "RESEARCH QUESTION & PICO (Stage 2 output):"

  Stage 4+ dispatch: Also insert 03_inclusion_exclusion/criteria.md
    (or study_protocol.md for original_research)
    Label: "ELIGIBILITY CRITERIA (Stage 3 output):"

  Stage 5+ dispatch: Also insert the first 5 lines of 04_database_search/search_strategy.md
    (just the concept table header — not the full strategy)
    Label: "SEARCH STRATEGY SUMMARY (Stage 4 output — first concept block only):"

  Stage 6+ dispatch: Also insert 05_screening/screened_results.csv INCLUDE rows only
    (filter to final_decision = INCLUDE)
    Label: "INCLUDED STUDIES LIST (Stage 5 output — INCLUDE decisions only):"

  Stage 7 dispatch: Also insert 06_data_extraction/synthesis_report.md (full content)
    and the first 20 lines of 06_data_extraction/extracted_data.csv
    Label: "SYNTHESIS REPORT (Stage 6 output):" and "EXTRACTED DATA SAMPLE (Stage 6 output):"

Omit any section whose source file does not yet exist — do not error, just skip it.

STAGE SKILL INSTRUCTIONS:
[Insert full SKILL.md content for this stage]

AGENT ROLE DEFINITIONS:
[Insert full references/agent-roles.md]

REVIEWER PROTOCOL:
[Insert full references/reviewer-protocol.md]

PROJECT TYPE ADAPTATIONS:
[Insert full references/project-types.md]

DIRECTORY CONVENTIONS:
[Insert full references/directory-conventions.md]

Execute the skill instructions above. Read input files from the project directory. Write output
files to the correct locations per the directory conventions. Follow the reviewer protocol as
specified in the skill. Update project.yaml when complete.
```

## Pipeline Stages

Execute in order, adapting for project type per `references/project-types.md`:

### Stage 1: Literature Search
- Skill: `skills/literature-search/SKILL.md`
- Prerequisite: project.yaml exists
- Output: `01_literature_search/landscape_report.md`

### Stage 2: Research Question
- Skill: `skills/research-question/SKILL.md`
- Prerequisite: `01_literature_search/landscape_report.md`
- Output: `02_research_question/research_question.md`

### Stage 3: Inclusion/Exclusion Criteria
- Skill: `skills/inclusion-exclusion/SKILL.md`
- Prerequisite: `02_research_question/research_question.md`
- Output: `03_inclusion_exclusion/criteria.md` (or `study_protocol.md` for original research)
- **Skip for:** case_report, case_series

### Stage 4: Database Search Build
- Skill: `skills/database-search-build/SKILL.md`
- Prerequisite: `03_inclusion_exclusion/criteria.md`
- Output: `04_database_search/search_strategy.md` (or `data_collection_instruments.md` for original_research)
- **MANUAL HANDOFF after this stage**
- **Skip for:** case_report, case_series
- **Modified for:** original_research (builds data collection instruments, not search strategies)

### Manual Handoff 1

```
⏸️  PIPELINE PAUSED — Database searches ready for execution

Search strategies: 04_database_search/search_strategy.md

─────────────────────────────────────────────
PUBMED/MEDLINE
  Export: Send to → Citation manager → Format: PubMed → Create file (.nbib)
  Alternative: Save → Format: PubMed → Create file (.txt)
  Place in: [project_dir]/04_database_search/abstracts/

OVID MEDLINE / EMBASE (if you have access)
  Export: Export → Format: RIS (.ris) or Direct Export
  Place in: [project_dir]/04_database_search/abstracts/
─────────────────────────────────────────────
DEDUPLICATION (optional — pipeline handles this programmatically)
  If you prefer to deduplicate in a reference manager first:
  • Rayyan: import all files → use Duplicates feature → export as .ris
  • Zotero: import → Duplicates pane → Merge duplicates → export as .ris
  • Covidence: upload .ris files directly (auto-deduplicates)
  Note the pre/post deduplication counts for your PRISMA flow diagram.
─────────────────────────────────────────────
No Ovid/Embase access? PubMed alone is acceptable — note this as a limitation.
PROSPERO tip: If registering, complete the PROSPERO submission now (see
  03_inclusion_exclusion/prospero_protocol.md if generated) before searching.
─────────────────────────────────────────────

EXPECTED VOLUME: Based on the search yield estimate, you should expect
  approximately [estimated_total from Stage 4] records across all databases.
  This is [within the expected range / above typical / below typical] for a
  [project_type]. After deduplication, the pipeline will screen records in
  batches — no manual screening is needed.
─────────────────────────────────────────────

When done: "I've placed the search results in the abstracts folder — please continue"
Include how many records each database returned (for PRISMA flow diagram).
```

### Stage 5: Abstract Screening
- Skill: `skills/abstract-screening/SKILL.md`
- Prerequisite: Files in `04_database_search/abstracts/`
- Output: `05_screening/screened_results.csv`
- **MANUAL HANDOFF after this stage**
- **Skip for:** case_report, case_series
- **Modified for:** original_research (data cleaning, not abstract screening)

### Manual Handoff 2

```
⏸️  PIPELINE PAUSED — Full-text retrieval required

Screening complete: [N] articles included.
Included list with DOIs and PMIDs: 05_screening/screened_results.csv

─────────────────────────────────────────────
🔓 OPEN ACCESS PDFs (auto-discovered via Unpaywall API):
  [n_oa]/[n_total] PDFs are freely available!
  OA PDF links: 05_screening/oa_pdf_links.csv
  → Download these first — no institutional access needed
─────────────────────────────────────────────
REMAINING PAYWALLED PDFs ([n_total - n_oa] articles):
  1. Institutional proxy/VPN — authenticate then search by DOI
  2. PubMed Central — https://www.ncbi.nlm.nih.gov/pmc/ (search by PMID)
  3. Interlibrary loan (ILL) — for inaccessible articles; typically 24–72 hours
  4. Author contact — email corresponding author; most respond quickly
─────────────────────────────────────────────
NAMING: [FirstAuthor]_[Year]_[keyword].pdf (e.g., Smith_2022_dupilumab_AD.pdf)
DESTINATION: [project_dir]/06_data_extraction/full_texts/
─────────────────────────────────────────────
⚠️ PDF FORMAT: Ensure PDFs contain selectable text (not scanned images).
  The pipeline reads PDF text directly — image-only scans cannot be processed.
  If a PDF is a scanned image: run OCR first (Adobe Acrobat → Recognize Text,
  or free tool: ocrmypdf input.pdf output.pdf). The pipeline will pre-check
  all PDFs and flag any that are unreadable before starting extraction.
─────────────────────────────────────────────
UNAVAILABLE ARTICLES:
  Report the PMID/DOI of any articles you cannot retrieve. These will be
  documented as "full text not retrieved" in the PRISMA flow diagram — this
  is standard practice and does not require explanation beyond the reason
  (e.g., "institutional access unavailable").
─────────────────────────────────────────────

When done: "I've added the PDFs to the full_texts folder — please continue"
  OR: "I retrieved [N]/[N] PDFs — unable to access: [PMID list]"
```

### Stage 6: Data Extraction & Synthesis
- Skill: `skills/data-extraction/SKILL.md`
- Prerequisite: PDFs in `06_data_extraction/full_texts/`
- Output: extraction template, extracted data, synthesis report (+ meta-analysis results if applicable)

### Stage 7: Manuscript Writing
- Skill: `skills/manuscript-writing/SKILL.md`
- Prerequisite: All prior stage outputs
- Output: `07_manuscript/manuscript.md` + `07_manuscript/references.bib`

### Pipeline Complete

```
PIPELINE COMPLETE!

Your manuscript is ready at: 07_manuscript/manuscript.md
References: 07_manuscript/references.bib

All stages completed:
  Literature Search
  Research Question
  Inclusion/Exclusion Criteria
  Database Search Build
  Abstract Screening
  Data Extraction & Synthesis
  Manuscript Writing

NEXT STEPS:
1. Review the manuscript carefully for clinical accuracy
2. Add author details and affiliations
3. Complete declarations (funding, conflicts, ethics)
4. Convert to submission format
5. Submit to [target_journal]!
```
