# Directory Conventions Reference

This file defines the per-project directory structure used by the clinical research pipeline. The orchestrator reads this file when creating new project directories, and all skills reference it to determine where to read and write files.

---

## 1. Directory Structure

Every project lives in its own directory under the pipeline workspace. The directory name is the kebab-case form of the project name. All numbered stage directories are created at project initialization; files within them are created by the skill responsible for that stage.

```
<project-name>/
├── project.yaml
├── 01_literature_search/
│   └── landscape_report.md
├── 02_research_question/
│   └── research_question.md
├── 03_inclusion_exclusion/
│   └── criteria.md
├── 04_database_search/
│   ├── search_strategy.md
│   └── abstracts/                  # MANUAL — user places downloaded abstract files here
├── 05_screening/
│   ├── screened_results.csv
│   ├── screening_progress.yaml
│   └── oa_pdf_links.csv            # Unpaywall OA PDF links (auto-generated)
├── 06_data_extraction/
│   ├── extraction_template.csv     # review types only
│   ├── extracted_data.csv          # review types only
│   ├── full_texts/                 # MANUAL — user places full-text PDFs here
│   ├── extraction_progress.yaml
│   ├── synthesis_report.md
│   ├── meta_analysis_results.md    # meta-analysis projects only
│   ├── nma_results.md              # network meta-analysis only
│   ├── case_presentation.md        # case_report / case_series only
│   ├── charted_data.csv            # scoping_review only
│   ├── thematic_map.md             # scoping_review only
│   ├── analysis_scripts/           # meta-analysis / DTA projects
│   │   ├── meta_analysis_primary.py
│   │   ├── code_review.md
│   │   └── outputs/
│   └── raw_data/                   # original_research only — user places data here
└── 07_manuscript/
    ├── manuscript.md
    ├── references.bib
    ├── proofread_report.md
    └── figures/                    # PRISMA flow, forest plots, etc.
```

### Directory and file notes

| Path | Created by | Notes |
|------|-----------|-------|
| `project.yaml` | Orchestrator | Created at project initialization. Updated by each skill as stages progress. |
| `01_literature_search/landscape_report.md` | Stage 1 skill | Narrative landscape survey of existing literature. |
| `02_research_question/research_question.md` | Stage 2 skill | Structured research question (PICO/PEO/PCC framework). |
| `03_inclusion_exclusion/criteria.md` | Stage 3 skill | Inclusion and exclusion criteria table. |
| `04_database_search/search_strategy.md` | Stage 4 skill | Boolean search strings per database. |
| `04_database_search/abstracts/` | User (manual) | User downloads abstract result sets from databases and places files here. Accepted formats listed in Section 3. |
| `05_screening/screened_results.csv` | Stage 5 skill | Title/abstract screening decisions with rationale. |
| `05_screening/screening_progress.yaml` | Stage 5 skill | Batching progress tracker (see Section 4). |
| `06_data_extraction/extraction_template.csv` | Stage 6 skill | Column schema for data extraction, generated from criteria and research question. |
| `06_data_extraction/extracted_data.csv` | Stage 6 skill | Extracted data from included studies. |
| `06_data_extraction/full_texts/` | User (manual) | User places full-text PDF files here for data extraction. |
| `06_data_extraction/extraction_progress.yaml` | Stage 6 skill | Batching progress tracker (see Section 4). |
| `06_data_extraction/synthesis_report.md` | Stage 6 skill | Narrative or thematic synthesis of extracted data. |
| `06_data_extraction/meta_analysis_results.md` | Stage 6 skill | Statistical meta-analysis output. Only created when `review_config.meta_analysis: true`. |
| `06_data_extraction/nma_results.md` | Stage 6 skill | Network meta-analysis output. Only created when `review_config.network_meta_analysis: true`. |
| `06_data_extraction/case_presentation.md` | Stage 6 skill | CARE-structured case presentation. Only for `case_report` and `case_series`. |
| `06_data_extraction/charted_data.csv` | Stage 6 skill | Thematic charting data. Only for `scoping_review`. |
| `06_data_extraction/thematic_map.md` | Stage 6 skill | Evidence gap map and thematic summary. Only for `scoping_review`. |
| `06_data_extraction/analysis_scripts/` | Stage 6 skill | Executable Python/R analysis scripts and code review. Created for meta-analysis and DTA projects. |
| `06_data_extraction/raw_data/` | User (manual) | Raw data files for original research projects. User places CSV/Excel files here. |
| `07_manuscript/manuscript.md` | Stage 7 skill | Full manuscript draft in Markdown. |
| `07_manuscript/references.bib` | Stage 7 skill | BibTeX reference list compiled from cited studies. |
| `07_manuscript/proofread_report.md` | Stage 7 skill | Proofreader quality report (grammar, formatting, AI-artifact detection). |
| `07_manuscript/figures/` | Stage 7 skill | PRISMA flow diagram, forest plots, and other generated figures. |

### Directories marked MANUAL

Directories marked **MANUAL** are not populated by skills. The orchestrator must pause the pipeline and prompt the user to supply the required files before the downstream skill can proceed. The orchestrator should validate that at least one file exists in the manual directory before advancing.

---

## 2. project.yaml Schema

The `project.yaml` file is the single source of truth for project metadata and pipeline state. It is created by the orchestrator at project initialization and updated by skills as stages complete.

```yaml
# ── Project Metadata ──────────────────────────────────────────────
project_name: "string"                  # Human-readable project title
project_type: "enum"                    # One of: systematic_review | scoping_review | rapid_review | original_research | case_report | case_series | meta_analysis | qualitative_synthesis | diagnostic_test_accuracy
topic: "string"                         # Free-text description of the research topic
target_journal: "string"               # Name of the target journal for submission
journal_guidelines_url: "string"       # URL to the journal's author guidelines

# ── Pipeline Stage Status ─────────────────────────────────────────
# Each stage value is one of: pending | in_progress | completed
stages:
  literature_search: "pending"          # Stage 1
  research_question: "pending"          # Stage 2
  inclusion_exclusion: "pending"        # Stage 3
  database_search: "pending"            # Stage 4
  abstract_screening: "pending"         # Stage 5
  data_extraction: "pending"            # Stage 6
  manuscript_writing: "pending"         # Stage 7

# ── Database Configuration ────────────────────────────────────────
# List of databases to search. Used by Stage 4 to generate per-database search strategies.
databases:                              # list of strings
  - "PubMed"
  - "Embase"
  # Add additional databases as needed (e.g., CINAHL, Cochrane, Scopus, Web of Science, PsycINFO)

# ── Review Configuration ──────────────────────────────────────────
review_config:
  reporting_standard: "string"          # Auto-selected from project_type (PRISMA, PRISMA-ScR, STROBE, CARE, ENTREQ, STARD)
  meta_analysis: false                  # boolean — true if the project includes quantitative meta-analysis
  network_meta_analysis: false          # boolean — true if comparing ≥3 interventions indirectly
  date_range: "string"                  # Date filter for database searches, e.g., "2015-01-01 to 2025-12-31"
  subgroup_analyses: []                 # list of strings — pre-specified subgroup analyses (e.g., "by age group")
  sensitivity_analyses: []              # list of strings — pre-specified sensitivity analyses (e.g., "excluding high RoB")
  clinical_coding: false                # boolean — true to add ICD-10/SNOMED/CPT columns to extraction template
  synthesis_approach: "string"          # qualitative_synthesis only: thematic | framework | meta_ethnography | meta_aggregation
  diagnostic_model: "string"            # diagnostic_test_accuracy only: bivariate | hsroc
  n_cases: 0                            # case_series only: number of patients in the series
  prospero_registration: false          # boolean — true if protocol is registered on PROSPERO
  databases_skipped: []                 # list of strings — databases that were inaccessible (populated during Stage 4)
```

### Field details

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `project_name` | string | yes | Human-readable title used in manuscript headers and reports. |
| `project_type` | enum | yes | Determines which skills are activated and which reporting standards apply. Allowed values: `systematic_review`, `scoping_review`, `rapid_review`, `original_research`, `case_report`, `case_series`, `meta_analysis`, `qualitative_synthesis`, `diagnostic_test_accuracy`. |
| `topic` | string | yes | Free-text description of the research topic. Used as seed input for Stage 1. |
| `target_journal` | string | yes | Target journal name. Used by Stage 7 for formatting. |
| `journal_guidelines_url` | string | no | URL to author guidelines. Fetched by Stage 7 to apply formatting rules. |
| `stages.<stage>` | enum | yes | Pipeline progress tracker. Values: `pending` (not started), `in_progress` (skill is executing), `completed` (output written and reviewed). |
| `databases` | list of strings | yes | Databases for literature search. Each entry maps to a search strategy block in Stage 4 output. |
| `review_config.reporting_standard` | string | yes | Reporting guideline the manuscript must follow. Auto-selected: systematic_review/meta_analysis/rapid_review→PRISMA, scoping_review→PRISMA-ScR, original_research→STROBE, case_report/case_series→CARE, qualitative_synthesis→ENTREQ, diagnostic_test_accuracy→STARD. |
| `review_config.meta_analysis` | boolean | yes | When `true`, Stage 6 produces `meta_analysis_results.md` in addition to `synthesis_report.md`. |
| `review_config.network_meta_analysis` | boolean | no | When `true`, Stage 6 also produces NMA results (`nma_results.md`). Requires ≥5 studies with connected network. |
| `review_config.date_range` | string | no | Date boundary for database searches. Format: `YYYY-MM-DD to YYYY-MM-DD`. |
| `review_config.subgroup_analyses` | list | no | Pre-specified subgroup analyses. Informs extraction template design and Stage 6 analysis. |
| `review_config.sensitivity_analyses` | list | no | Pre-specified sensitivity analyses. Informs extraction template and Stage 6 analysis. |
| `review_config.clinical_coding` | boolean | no | When `true`, extraction template includes ICD-10, ICD-11, CPT, SNOMED CT columns. |
| `review_config.synthesis_approach` | string | no | `qualitative_synthesis` only. One of: `thematic`, `framework`, `meta_ethnography`, `meta_aggregation`. |
| `review_config.diagnostic_model` | string | no | `diagnostic_test_accuracy` only. One of: `bivariate`, `hsroc`. |
| `review_config.n_cases` | integer | no | `case_series` only. Number of patients in the series. |
| `review_config.prospero_registration` | boolean | no | Whether the protocol is registered on PROSPERO. |
| `review_config.databases_skipped` | list | no | Databases that were inaccessible during Stage 4. Populated automatically. |

---

## 3. Accepted Abstract Formats

When users download search results from databases and place them in `04_database_search/abstracts/`, the Stage 5 screening skill must be able to parse the files. The following formats are accepted.

| Format | Extension(s) | Expected Structure |
|--------|-------------|-------------------|
| **CSV** (PubMed) | `.csv` | Comma-separated file with headers. Expected columns: `PMID`, `Title`, `Authors`, `Citation`, `First Author`, `Journal/Book`, `Publication Year`, `Create Date`, `PMCID`, `DOI`, `Abstract`. Column order may vary; matching is by header name. |
| **RIS** (general) | `.ris` | Tagged format. Each record starts with `TY  -` and ends with `ER  -`. Key tags: `TI` (title), `AU` (author, one per line), `AB` (abstract), `PY` (publication year), `JO` or `JF` (journal), `DO` (DOI), `AN` (accession number), `KW` (keywords). Multiple records are separated by `ER  -` lines. |
| **TXT** (PubMed plain text) | `.txt` | PubMed plain-text format. Records are separated by blank lines. Each record contains labeled fields on separate lines: `PMID-`, `TI  -`, `AU  -`, `AB  -`, `DP  -` (date of publication), `JT  -` (journal title), `LID -` (DOI). Continuation lines are indented with spaces. |

### Parsing rules

1. The screening skill must auto-detect the format by file extension.
2. If multiple files are present in `abstracts/`, they are all parsed and de-duplicated by DOI (preferred) or by title similarity (fallback) before screening begins.
3. Records missing an abstract are retained but flagged in `screened_results.csv` with a note in the rationale column.

---

## 4. Batching Progress Files

Screening (Stage 5) and data extraction (Stage 6) process records in batches to stay within context limits. Progress is tracked in YAML files so the orchestrator can resume interrupted runs.

### Schema: `screening_progress.yaml` / `extraction_progress.yaml`

```yaml
total_items: 0              # integer — total number of records to process
batch_size: 0               # integer — number of records per batch
batches_completed: 0         # integer — number of batches finished so far
last_batch_end_index: 0      # integer — zero-based index of the last record processed (exclusive)
items_included: 0            # integer — cumulative count of records marked "include"
items_excluded: 0            # integer — cumulative count of records marked "exclude"
status: "not_started"        # enum — one of: not_started | in_progress | completed
```

### Field details

| Field | Type | Description |
|-------|------|-------------|
| `total_items` | integer | Set once at the start of the stage. Equals the number of records to screen (Stage 5) or the number of included studies to extract (Stage 6). |
| `batch_size` | integer | Number of records processed per batch invocation. Determined by the skill based on average record length and context budget. Typical values: 20-50 for screening, 5-10 for extraction. |
| `batches_completed` | integer | Incremented by 1 after each batch finishes and results are written to the output CSV. |
| `last_batch_end_index` | integer | Zero-based exclusive end index. The next batch starts reading from this index. For example, if `last_batch_end_index` is 50, the next batch reads records 50 through 50 + `batch_size` - 1. |
| `items_included` | integer | Running total of records with an "include" decision. Updated after each batch. |
| `items_excluded` | integer | Running total of records with an "exclude" decision. Updated after each batch. |
| `status` | enum | `not_started` before any batch runs. `in_progress` once the first batch begins. `completed` when `last_batch_end_index >= total_items`. |

### Orchestrator usage

1. Before dispatching a batch, read the progress file to determine `last_batch_end_index` and pass it to the skill as the start offset.
2. After the skill writes batch results to the output CSV, verify that the progress file has been updated.
3. If `status` is `completed`, proceed to review. If `in_progress`, dispatch the next batch.
4. If a run is interrupted, the orchestrator resumes from `last_batch_end_index` on the next invocation without reprocessing earlier batches.
