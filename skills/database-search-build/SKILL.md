---
name: database-search-build
description: |
  Use when the user wants to build a database search strategy, create search strings,
  translate searches between databases, or prepare search queries for PubMed/MEDLINE/Embase.
  Also triggers for data collection instrument design in original research projects.
  Trigger on phrases like "search strategy", "search string", "PubMed search",
  "MEDLINE search", "Embase search", "build the search", or "database query".
---

# Database Search Build (Stage 4)

Build copy-paste-ready database search strategies using the Librarian agent.

## Prerequisites

- `02_research_question/research_question.md` must exist
- `03_inclusion_exclusion/criteria.md` must exist
- Read `project.yaml` for `databases` list

## Process — Review Types

### Step 1: Dispatch Librarian Agent

**Librarian agent prompt:**

```
[Insert Librarian system prompt from agent-roles.md]

TASK: Build complete, copy-paste-ready search strategies for the following databases.

LANDSCAPE SURVEY CONTEXT (Stage 1 output — use to seed MeSH terms and synonyms):
[Read 01_literature_search/landscape_report.md. Extract and insert:
  - The "Preliminary Search Terms" section (exact vocabulary used in key existing papers)
  - The "Recommended Databases" section (confirms database selection)
  - Any controlled vocabulary terms or MeSH headings mentioned in the landscape survey.
Use these terms as the foundation for concept blocks — they reflect how the best existing
papers on this topic are indexed, which maximizes recall.]

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

INCLUSION/EXCLUSION CRITERIA:
[Read and insert 03_inclusion_exclusion/criteria.md]

TARGET DATABASES: [from project.yaml databases list]
DATE RANGE: [from project.yaml review_config.date_range]

PROJECT-TYPE DATABASE AND FILTER ADAPTATIONS (apply before building strategies):

**If `project_type` is `qualitative_synthesis`:**
- Mandatory additional databases (add these even if not listed in project.yaml): CINAHL (EBSCO), PsycINFO (Ovid or ProQuest). Recommended: AMED (Allied and Complementary Medicine Database), Sociological Abstracts.
- Concept structure: Use PICo — Block 1 = Population terms, Block 2 = Phenomenon of Interest terms (experiences, perceptions, attitudes, barriers, facilitators, lived experience, perspectives, views, etc.). Do NOT create Comparator or Outcome concept blocks — these do not apply to qualitative synthesis.
- Add a qualitative study design filter as the final AND step in each database strategy (after date/language limits):
  - PubMed: `("qualitative research"[MeSH Terms] OR "interviews as topic"[MeSH Terms] OR "focus groups"[MeSH Terms] OR "thematic analysis"[tiab] OR "grounded theory"[tiab] OR "phenomenol*"[tiab] OR "ethnograph*"[tiab] OR "narrative research"[tiab] OR "qualitative stud*"[tiab])`
  - Ovid MEDLINE: `(qualitative research/ OR interviews as topic/ OR focus groups/ OR thematic analysis.mp. OR grounded theory.mp. OR phenomenol*.mp. OR ethnograph*.mp.)`
  - CINAHL: `(MH "Qualitative Studies" OR MH "Interviews" OR MH "Focus Groups" OR TX "thematic analysis" OR TX "grounded theory" OR TX "phenomenolog*" OR TX "ethnograph*")`
  - PsycINFO: `(qualitative research/ OR interviews/ OR focus groups/ OR thematic analysis.mp. OR grounded theory.mp. OR phenomenolog*.mp.)`
- Do NOT apply RCT, systematic review, clinical trial, or controlled trial publication type limits — these exclude qualitative studies.

**If `project_type` is `diagnostic_test_accuracy`:**
- Concept structure: Use PICOS — Block 1 = Population (patients suspected of having the condition), Block 2 = Index Test terms (name of test + variants + brand names). Do NOT create a separate Comparator block for the reference standard — it is not a useful search concept and will be confirmed at full-text screening.
- Add a DTA methodological filter as the final AND step in each database strategy (after date/language limits):
  - PubMed: `("sensitivity and specificity"[MeSH Terms] OR "ROC Curve"[MeSH Terms] OR "predictive value of tests"[MeSH Terms] OR "diagnostic accuracy"[tiab] OR "likelihood ratio*"[tiab] OR "area under the curve"[tiab] OR "AUROC"[tiab])`
  - Ovid MEDLINE: `(sensitivity and specificity/ OR ROC curve/ OR predictive value of tests/ OR diagnostic accuracy.mp. OR likelihood ratio*.mp. OR area under the curve.mp.)`
  - Embase: `(sensitivity and specificity/exp OR ROC curve/exp OR diagnostic accuracy/exp OR likelihood ratio*.mp.)`
- Do NOT apply RCT, systematic review, intervention-study, or clinical trial publication type limits — these incorrectly exclude diagnostic accuracy studies.

**For all other project types:** No adaptations needed — use the standard PICO concept structure with the databases in project.yaml.

For EACH database, produce:

1. **Concept table**: Break the research question into 2-4 key concepts. For each concept, list:
   - Controlled vocabulary terms (MeSH for PubMed/MEDLINE, Emtree for Embase)
   - Free-text synonyms and variants
   - Truncation and wildcard usage

2. **Search strategy**: Line-numbered Boolean search blocks. Example format:
   ```
   PubMed Search Strategy
   1. "artificial intelligence"[MeSH Terms]
   2. "machine learning"[All Fields]
   3. "deep learning"[All Fields]
   4. #1 OR #2 OR #3
   5. "dermatology"[MeSH Terms]
   6. "skin diseases"[MeSH Terms]
   7. #5 OR #6
   8. #4 AND #7
   9. #8 AND ("2015/01/01"[Date - Publication] : "2026/12/31"[Date - Publication])
   ```

3. **Database-specific syntax notes**: Highlight any syntax differences between databases.

4. **Filters applied**: Date range, language, study type filters.

SELF-CHECK (perform before writing output):
1. Every database in the TARGET DATABASES list has a complete, numbered strategy — none are partially built or omitted
2. Boolean logic: each OR block contains synonyms for exactly one concept; AND connects concept blocks; no operator is orphaned or misplaced; parentheses nest correctly
3. MeSH/Emtree terms: use WebSearch to verify any term you are uncertain about — do not guess; note deprecated terms with current replacements
4. Free-text coverage: each concept block contains ≥3 synonyms or variants, including truncated forms (e.g., `dermatolog*`, `immuno*`)
5. Database-specific syntax: PubMed uses `[MeSH Terms]` and `[All Fields]`; Ovid uses `/` for MeSH and `.mp.` for free text; Embase uses `/exp` for exploded terms — confirm each strategy uses the correct syntax for its database
6. Filters: date range and language/study-type filters are applied as the final AND step (not embedded inside concept blocks)
7. The strategy is reproducible: a researcher could copy-paste it and run it without modification
8. For `qualitative_synthesis`: CINAHL is included in the strategies; a qualitative study design filter block is the final AND step in each strategy; no RCT or controlled trial publication type limit is present
9. For `diagnostic_test_accuracy`: a DTA methodological filter block (sensitivity and specificity MeSH or equivalent) is the final AND step in each strategy; no RCT, systematic review, or intervention-study publication type limit is present
Apply corrections before outputting.

OUTPUT: Complete search strategy document with all databases.
```

### Step 2: Write Output

Write to `04_database_search/search_strategy.md`.

### Step 3: Quick Review with Retry Logic

**Review criteria for this stage:**
- Is the Boolean logic correct (no orphaned operators, proper nesting)?
- Are MeSH/Emtree terms appropriate and current?
- Do free-text terms cover relevant synonyms?
- Is the syntax correct for each specific database?
- Are date and language filters properly applied?

Initialize `review_iteration = 0`. Dispatch Quick Review per `references/reviewer-protocol.md`.

On **REVISE** verdict:
1. Build a REVISION REQUIRED block: table of reviewer findings (finding | database | line numbers affected)
2. Re-dispatch the Librarian with the original task prompt + REVISION REQUIRED block prepended; Librarian must address each finding explicitly and output a corrected strategy
3. Overwrite `04_database_search/search_strategy.md` with corrected version
4. Increment `review_iteration`; re-dispatch reviewer with revision history in context

On **APPROVE**: proceed to Step 4.

On **REJECT** or `review_iteration ≥ 2` without APPROVE: trigger full escalation per reviewer-protocol.md.

### Step 4: Update project.yaml + Manual Handoff

Set `stages.database_search: completed`.

Print to user:

```
Search strategies built and saved to 04_database_search/search_strategy.md

MANUAL STEP REQUIRED:
1. Open each database (PubMed, MEDLINE/Ovid, Embase, etc.)
2. Copy-paste the corresponding search strategy from search_strategy.md
3. Execute the search
4. Export results (accepted formats: PubMed CSV, RIS, or plain text)
5. Place exported files in: 04_database_search/abstracts/

When you're done, run /abstract-screening to continue.
```

## Process — Original Research

For `project_type: original_research`, this stage builds data collection instruments:

### Step 1: Dispatch Methodologist Agent

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt]

TASK: Build data collection instruments for the study described in the protocol below.

STUDY PROTOCOL:
[Read and insert 03_inclusion_exclusion/study_protocol.md]

Produce three linked deliverables:

1. DATA COLLECTION FORM — A structured table listing every field to be collected:
   - Include EVERY variable named in the study protocol (independent variables,
     dependent/outcome variables, confounders, and covariates)
   - For each field: field_name (snake_case), label (human-readable), data_type
     (text / integer / decimal / date / boolean / categorical), valid_values_or_range,
     required (yes/no), section (Demographics / Diagnosis / Exposure / Outcome /
     Confounders / Administrative)
   - Add a participant_id field (anonymized integer — no PHI) as the first field
   - Add a data_entry_date and data_entry_by field at the end

2. VARIABLE CODEBOOK — For each categorical variable, enumerate the valid codes:
   | Variable | Code | Label | Notes |
   For continuous variables: specify min/max plausible range and units

3. DATA QUALITY CHECKS — List the validation rules a REDCap or spreadsheet
   system should enforce:
   - Range checks: e.g., "age must be between 0 and 120"
   - Logic checks: e.g., "if diabetes = 1, then HbA1c_value is required"
   - Required fields: list which fields must be non-empty for a record to be valid
   - Uniqueness check: participant_id must be unique across all rows

**SELF-CHECK (required before producing output):**

Verify each item below and correct any failures inline:

- [ ] Every variable explicitly named in `study_protocol.md` (primary outcome,
      secondary outcomes, key confounders) has a corresponding field in the
      collection form — no protocol variable is absent without documented rationale
- [ ] Every categorical variable has a codebook entry with exhaustive valid codes
      (no code = "other unspecified" as the only option)
- [ ] Every continuous variable has a plausible numerical range in the codebook
      (prevents grossly out-of-range data entry errors)
- [ ] At least one logic check is defined if the protocol has conditional
      data elements (e.g., "if condition X, collect additional variable Y")
- [ ] The collection form is operationalizable by a research coordinator or chart
      reviewer — no field requires clinical judgment not explained in the codebook
- [ ] participant_id, data_entry_date, and data_entry_by fields are present

Correct any failures inline. Do not output instruments that fail these checks.

OUTPUT: Three sections in markdown: (1) Data Collection Form table, (2) Variable
Codebook, (3) Data Quality Checks list.
```

### Step 2: Write Output

Write to `04_database_search/data_collection_instruments.md`.

### Step 3: Quick Review

Initialize `review_iteration = 0`. Dispatch Quick Review per `references/reviewer-protocol.md`.

**Review criteria for this stage (original_research data collection instruments):**
- Does the collection form include every variable named in `study_protocol.md`?
- Is every categorical variable in the codebook with exhaustive valid codes?
- Are data quality checks present (range checks, required fields, logic checks)?
- Is the participant_id field anonymized (no PHI)?
- Could a research coordinator apply this form without additional clinical guidance?

**Retry logic:**

On **REVISE**:
1. Build a REVISION REQUIRED table:
   | Finding | Field/Section | Issue | Required action |
   |---------|--------------|-------|----------------|
2. Re-dispatch Methodologist with the original prompt + REVISION REQUIRED table prepended; agent must address each finding explicitly and re-output revised instruments to disk
3. Increment `review_iteration += 1`
4. Re-dispatch reviewer with the revision history prepended to context

On **APPROVE**: proceed to Step 4.

On **REJECT** or `review_iteration >= 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Print escalation banner with all reviewer findings and unresolved issues
- Pause pipeline for human PI input before proceeding

### Step 4: Update project.yaml + Manual Handoff

Set `stages.database_search: completed` in `project.yaml`.

Print:

```
Data collection instruments saved to 04_database_search/data_collection_instruments.md

MANUAL STEP REQUIRED:
1. Collect data per the study protocol and instruments
2. Place raw data files in: 04_database_search/abstracts/

When you're done, run /abstract-screening to continue.
```

## Process — Case Report

This stage is skipped for case reports. Print:

```
Stage 4 (Database Search Build) is skipped for case reports.
Place your clinical materials in 06_data_extraction/full_texts/ and run /data-extraction.
```

## Edge Cases

### Zero Results

If the user reports that executing a search strategy returned zero results:

1. Re-dispatch the Librarian with this additional instruction:
   ```
   The user reported zero results for [database]. Diagnose the likely cause and generate a revised strategy.
   Possible causes to check:
   - MeSH/Emtree terms not mapped to correct controlled vocabulary
   - Overly restrictive AND combination (too many concept blocks)
   - Date range too narrow
   - Study type filter excluding relevant designs
   Provide: (a) diagnosis of most likely cause, (b) revised search strategy with at least 2 fallback options ranging from moderate to broad
   ```
2. Write revised strategy to `04_database_search/search_strategy_revised.md`.
3. Notify user with the diagnosis and ask them to run the revised strategy.

### Too Many Results (>500)

If the user reports returning >500 results and wants to narrow the search:

1. Re-dispatch the Librarian:
   ```
   The search returned [N] results, which is too many for efficient screening.
   Generate a more specific strategy by applying 2-3 of the following narrowing techniques:
   - Add a study design filter (e.g., randomized controlled trial [pt])
   - Restrict to a tighter date range
   - Narrow the population concept with more specific MeSH terms
   - Add a setting or severity qualifier
   Aim for a target yield of 100-300 results. Explain each change made.
   ```
2. Write narrowed strategy to `04_database_search/search_strategy_narrowed.md`.
3. Note: if the user prefers to screen all results, standard batching in the abstract-screening stage will handle it — narrowing is optional.

### Database Access Failures

If the user reports being unable to access a database (e.g., no Embase subscription):

- Record the inaccessible database under `review_config.databases_skipped` in `project.yaml`
- Add a PRISMA flow note: "Database X was unavailable — search limited to [available databases]"
- Note this as a study limitation in `04_database_search/search_strategy.md`
