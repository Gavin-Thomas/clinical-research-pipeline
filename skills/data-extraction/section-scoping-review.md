# Data Extraction — Scoping Review (Data Charting)

Scoping reviews use **data charting** (thematic categorization and mapping), not data extraction
for statistical pooling. There is no meta-analysis, no pooled estimates, no forest plots, and no
I-squared. Quality appraisal is optional per JBI scoping review methodology. The output is a
charting table + narrative evidence map, not extracted_data.csv + synthesis_report.md for pooling.

---

## Step 1: Dispatch Methodologist for Charting Framework Design

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Design a data charting framework for this scoping review.

RESEARCH QUESTION (PCC Framework):
[Read and insert 02_research_question/research_question.md]

PROJECT TYPE: scoping_review
REPORTING STANDARD: PRISMA-ScR (PRISMA Extension for Scoping Reviews)

Scoping reviews use data charting — not data extraction for pooling. Design a charting
framework that maps the breadth of evidence relevant to the research question. The framework
must align with the PCC (Population, Concept, Context) components of the scoping question.

Include these column categories:

1. Study identification: author, year, title, journal, DOI, country
2. Study characteristics: study design, setting, sample size, population description,
   recruitment method
3. Population mapping: target population characteristics — aligned with the PCC "Population"
   component. Include age range, sex/gender, condition or status, subgroup descriptors
4. Concept mapping: main topic or concept category — derived from the PCC "Concept" component.
   Assign each study to one or more pre-defined concept categories that capture the breadth of
   the scoping question. List concept categories explicitly and define each one.
5. Context mapping: setting, country, healthcare system, geographic region, income level
   (World Bank classification), policy or regulatory context — aligned with the PCC "Context"
   component
6. Key findings: primary findings relevant to the scoping question (narrative, not effect sizes)
7. Gaps or limitations: gaps or limitations noted by the study authors themselves

Do NOT include columns for effect sizes, confidence intervals, p-values, standard errors,
variance, or any data intended for statistical pooling.

Quality appraisal columns are optional per JBI scoping review methodology. If the project.yaml
specifies `review_config.quality_appraisal: true`, add an appropriate critical appraisal tool
column (e.g., MMAT for mixed study types). Otherwise, omit quality appraisal columns and note
that quality appraisal was not performed per JBI scoping review guidance.

SELF-CHECK before writing to disk:
1. Every PCC component (Population, Concept, Context) has dedicated charting columns
2. Concept categories are explicitly defined with clear inclusion criteria for each
3. No meta-analytic columns are present (no effect sizes, CIs, p-values, SEs)
4. Framework includes both study-level characteristics and scoping-specific mapping fields
5. Column definitions are unambiguous — two independent extractors would code the same study
   the same way

OUTPUT: Charting framework as a structured markdown document with column definitions, plus a
CSV header row showing the exact column names to use.
```

Write charting framework to `06_data_extraction/charting_framework.md`.

---

## Step 2: Partition Papers Across 3 Data Extractors (Charting)

List all PDFs in `06_data_extraction/full_texts/`. Divide into 3 roughly equal groups. Dispatch
3 Data Extractor agents in parallel, each assigned their partition.

**Data Extractor agent prompt (charting):**

```
[Insert Data Extractor system prompt from agent-roles.md]

TASK: Chart data from the following papers using the provided charting framework.

NOTE — SCOPING REVIEW CHARTING MODE:
You are charting studies for a scoping review. This means categorizing and mapping — not
extracting quantitative data for pooling. Focus on accurately classifying each study by its
population, concept, and context. Use the pre-defined concept categories from the framework;
do not invent new categories without flagging them.

CHARTING FRAMEWORK:
[Read and insert 06_data_extraction/charting_framework.md — include column definitions]

PAPERS ASSIGNED TO YOU:
[List of PDF filenames in this partition]

For each paper:
1. Read the PDF using the Read tool (for papers >20 pages, read in chunks: pages="1-20",
   then pages="21-40", etc.)
2. Chart every field in the framework
3. Mark fields as "NR" (not reported) if the information is not available in the paper
4. Assign concept category from the pre-defined list in the charting framework; if a study
   spans multiple concept categories, list all applicable categories (pipe-separated, e.g.,
   "Prevention|Diagnosis")
5. Note the page number where key charting decisions were made (especially concept category
   assignment)

SELF-CHECK (perform before outputting CSV rows):
1. Every paper in your assigned partition has exactly one output row — no paper is missing
   or duplicated
2. No charting column is blank — use "NR" only for data genuinely absent from the paper,
   not for data that was hard to find
3. Concept categories are drawn from the pre-defined list in the charting framework — if you
   assigned a category not in the list, flag it with "[NEW CATEGORY]" for Methodologist review
4. Context fields (country, setting, healthcare system) are filled using standard terminology
   consistent across all studies (e.g., always "United States" not sometimes "US" and sometimes
   "USA")
5. Key findings are scoping-relevant summaries (what the study found), not statistical results
   (no effect sizes or p-values in this column)
6. Every row has a source_page reference for at least the concept category assignment
Apply any corrections inline before outputting.

OUTPUT: CSV rows matching the charting framework columns, one row per study. Include a
"source_notes" column with page references for key charting decisions.
```

---

## Step 3: Merge Charted Data

Combine all 3 extractors' outputs into `06_data_extraction/charted_data.csv`.

After merging, perform a consistency check:
- Verify that concept categories used across all 3 extractors match the pre-defined list from
  `charting_framework.md`
- Flag any "[NEW CATEGORY]" entries for Methodologist adjudication
- Standardize terminology: country names, setting descriptions, study design labels must use
  consistent terms across all rows
- Print the total number of charted studies and confirm it matches the expected count from
  `05_screening/screened_results.csv` (INCLUDE decisions only, minus any papers in
  `missing_full_texts.md`)

---

## Step 4: Dispatch Methodologist for Thematic Mapping and Evidence Gap Analysis

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Analyze the charted data from this scoping review and produce a thematic evidence map.

CHARTED DATA:
[Read and insert 06_data_extraction/charted_data.csv]

CHARTING FRAMEWORK:
[Read and insert 06_data_extraction/charting_framework.md]

RESEARCH QUESTION (PCC Framework):
[Read and insert 02_research_question/research_question.md]

PROJECT TYPE: scoping_review
REPORTING STANDARD: PRISMA-ScR

Produce the following sections:

1. FREQUENCY TABLES:
   - Number of studies per concept category
   - Number of studies per year (or year range if span is large)
   - Number of studies per country or geographic region
   - Number of studies per study design
   - Number of studies per population subgroup (as defined by PCC "Population")
   - Cross-tabulation: concept category x study design

2. EVIDENCE GAP MAP:
   Create a matrix of concept category (rows) x population or context dimension (columns).
   Each cell = number of studies. Highlight cells with 0 studies — these are evidence gaps.
   Identify at minimum:
   - Which concept categories have no or minimal (≤2) studies
   - Which population subgroups are underrepresented across all concept categories
   - Which context combinations (e.g., low-income countries, primary care settings) lack evidence

3. THEMATIC NARRATIVE MAP:
   Organize by concept category. For each category:
   - Summarize what is known (how many studies, what populations, what settings, key findings)
   - Identify within-category gaps (e.g., "5 studies in this category, all from high-income
     countries; no evidence from LMIC settings")
   - Note methodological patterns (e.g., "predominantly cross-sectional designs; no RCTs")
   Do NOT organize the narrative by individual study — organize by concept.
   Do NOT use meta-analytic language (no pooled estimates, no I-squared, no forest plots,
   no heterogeneity statistics).

4. VISUAL SUMMARY SUGGESTIONS:
   Describe specifications for figures the research team should create:
   - Bubble chart: x-axis = publication year, y-axis = concept category, bubble size =
     study count. Purpose: show temporal trends in research attention.
   - Heat map: rows = concept category, columns = country/region. Cell shading = study count.
     Purpose: geographic distribution of evidence.
   - Bar chart: concept categories ranked by study count. Purpose: relative research volume.
   Provide the data tables needed to generate each figure from charted_data.csv.

SELF-CHECK before writing synthesis_report.md:
1. Every study in charted_data.csv is represented in the frequency tables — row counts sum
   to total study count (accounting for studies assigned to multiple concept categories)
2. Evidence gap map is grounded in actual charted data — every "0 studies" cell is a true
   absence, not a missed count
3. Gap map identifies at least one genuine evidence gap (a concept x population or concept x
   context combination with zero studies)
4. Thematic narrative is organized by concept category, not by individual study
5. No meta-analytic language anywhere: no pooled estimates, no I-squared, no tau-squared,
   no forest plots, no heterogeneity statistics, no effect sizes
6. Visual summary data tables are computationally consistent with the frequency tables
Apply corrections before writing to disk.

OUTPUT: Structured synthesis report in markdown with all four sections above.
```

Write to `06_data_extraction/synthesis_report.md`.

---

## Step 5: Full Review with Retry Logic

Initialize `review_iteration = 0`. Dispatch Full Review per `references/reviewer-protocol.md`.

Provide the reviewer with:
- `06_data_extraction/charting_framework.md`
- `06_data_extraction/charted_data.csv`
- `06_data_extraction/synthesis_report.md`
- `02_research_question/research_question.md` (for PCC reference)

**Review criteria specific to scoping review charting:**
- Does the charting framework capture all three PCC components (Population, Concept, Context)?
- Are concept categories mutually exclusive and collectively exhaustive (MECE) — every included
  study fits at least one category, and category definitions do not overlap ambiguously?
- Is every study in charted_data.csv accounted for in the synthesis report frequency tables?
- Is the evidence gap map grounded in the actual charted data (not speculative — every gap
  cell corresponds to a verifiable zero in the data)?
- Is there any meta-analytic or statistical pooling language anywhere in the synthesis report?
  (pooled estimates, I-squared, tau-squared, forest plots, heterogeneity statistics, effect
  sizes, confidence intervals) — if so, flag for removal
- Is the thematic narrative organized by concept category (not by individual study)?
- Are concept categories applied consistently across all charted studies (same study
  characteristics coded the same way by different extractors)?

On **REVISE**:
1. Build a REVISION REQUIRED block: a table of reviewer findings (finding | affected study
   IDs or section | specific issue)
2. Determine scope: if charting errors, re-dispatch Data Extractor agents for affected rows
   only; if synthesis errors, re-dispatch Methodologist with the REVISION REQUIRED block
3. Update `charted_data.csv` and/or `synthesis_report.md` with corrections
4. Increment `review_iteration`; re-dispatch reviewer with revision history appended to context

On **APPROVE**: proceed to Step 6.

On **REJECT** or `review_iteration >= 2` without APPROVE: trigger full escalation per
reviewer-protocol.md (print escalation banner, list all unresolved findings, pause pipeline).

---

## Step 6: Update project.yaml

Set `stages.data_extraction: completed`.

Print:

```
Data charting complete (scoping review).

Files:
  Charting framework:    06_data_extraction/charting_framework.md
  Charted data:          06_data_extraction/charted_data.csv
  Synthesis report:      06_data_extraction/synthesis_report.md

Note: Scoping review methodology — no meta-analytic pooling performed.
Quality appraisal: [performed / not performed per JBI scoping review guidance].
```
