# Section B: Quantitative / Mixed Methods Manuscript

_For all project types except `qualitative_synthesis`, `case_report`, and `diagnostic_test_accuracy`._

### Step 1: Fetch Journal Guidelines

Use WebFetch to retrieve the target journal's author guidelines from `journal_guidelines_url` in `project.yaml`.

Extract and note:
- Word/character limits (abstract, full text)
- Required sections and order
- Reference formatting style (Vancouver, APA, etc.)
- Table/figure formatting requirements
- Required supplementary materials
- Reporting checklist requirements (PRISMA, etc.)
- Structured abstract format (if required)
- Conflict of interest / funding statement requirements

If WebFetch fails, ask the user to paste the guidelines.

### Step 1b: PRISMA Flow Diagram Generation

**Skip conditions:**
- Skip entirely for `case_report` (no PRISMA flow needed)
- Skip entirely for `original_research` (uses CONSORT/STROBE flow instead)
- For `diagnostic_test_accuracy`: generate a **STARD flow diagram** instead of PRISMA (adapt box labels below to STARD stages: enrollment → index test → reference standard → analysis)
- For `qualitative_synthesis`: generate PRISMA but use qualitative-specific labels (e.g., "Studies included in qualitative synthesis" instead of "Studies included in quantitative synthesis (meta-analysis)")

#### 1b.1: Collect PRISMA Flow Numbers

Read prior stage outputs and extract the following counts:

| PRISMA Box | Source File | How to Extract |
|---|---|---|
| Records identified (total, broken down by database) | `05_screening/deduplication_report.md` | Read the pre-deduplication total and per-database counts |
| Duplicates removed | `05_screening/deduplication_report.md` | Read the duplicates-removed count |
| Records screened | `05_screening/deduplication_report.md` | Post-dedup count (= total identified − duplicates removed) |
| Records excluded at screening | `05_screening/screened_results.csv` | Count rows where the decision column = "EXCLUDE" |
| Reports sought for retrieval | `05_screening/screened_results.csv` | Count rows where the decision column = "INCLUDE" |
| Reports not retrieved | `06_data_extraction/missing_full_texts.md` | If file exists, count listed entries; else 0 |
| Reports assessed for eligibility (full-text) | `06_data_extraction/full_texts/` | Count PDF files in this directory |
| Reports excluded at full-text, with reasons | `06_data_extraction/extracted_data.csv` | Count rows where data indicates "Full text unavailable", "Excluded at full text", or similar; group by exclusion reason |
| Studies included in review | `06_data_extraction/extracted_data.csv` | Count rows with actual extracted data present |
| Studies included in quantitative synthesis (meta-analysis) | `06_data_extraction/extracted_data.csv` + `project.yaml` | Same count as above if `review_config.meta_analysis` is `true` in project.yaml; else "N/A" |

Store all numbers in a dictionary/object for use below.

#### 1b.2: Generate PRISMA 2020 Flow Diagram

Produce two outputs:

**(a) Structured markdown table** (for inline manuscript use):

```markdown
# PRISMA 2020 Flow Diagram Data

## Identification
| Source | Records |
|---|---|
| Database 1 (name) | n |
| Database 2 (name) | n |
| ... | ... |
| **Total identified** | **N** |
| Duplicates removed | n |

## Screening
| Stage | Count |
|---|---|
| Records screened | N |
| Records excluded | n |

## Retrieval
| Stage | Count |
|---|---|
| Reports sought for retrieval | N |
| Reports not retrieved | n |

## Eligibility
| Stage | Count |
|---|---|
| Reports assessed for eligibility | N |
| Reports excluded (with reasons below) | n |

### Exclusion Reasons at Full-Text
| Reason | Count |
|---|---|
| [reason 1] | n |
| [reason 2] | n |

## Included
| Stage | Count |
|---|---|
| Studies included in review | N |
| Studies included in quantitative synthesis (meta-analysis) | N or N/A |
```

**(b) Executable Python script** (`prisma_flow.py`) using matplotlib to generate a PRISMA 2020 flow diagram PNG:

The script should:
- Create a figure with boxes and arrows following the PRISMA 2020 template layout
- Left column: "Identification" → "Screening" → "Included" (database/register sources)
- Right column: "Other sources" (registers, citation searching, etc.) — populate if data available, else omit
- Each box contains the stage label and the count
- Arrows connect boxes vertically (flow down) and horizontally (exclusions branch right)
- Use a clean, publication-quality style: white boxes with thin black borders, black arrows, readable font (Arial/Helvetica, 9–10 pt)
- Save the output to `07_manuscript/figures/prisma_flow.png` at 300 DPI
- Accept an optional `--output` argument to override the save path
- For `diagnostic_test_accuracy` projects: adapt to STARD flow layout
- For `qualitative_synthesis` projects: relabel the final box to "Studies included in qualitative synthesis"

#### 1b.3: Write Outputs

- Write the markdown table to `07_manuscript/prisma_flow_data.md`
- Write the Python script to `07_manuscript/prisma_flow.py`
- Create the `07_manuscript/figures/` directory if it does not exist
- Execute the Python script to generate `07_manuscript/figures/prisma_flow.png`

If matplotlib is not available in the environment, write the script but skip execution and note in the output that the user should run `python 07_manuscript/prisma_flow.py` manually.

#### 1b.4: Pass PRISMA Data Forward

Store the PRISMA flow data (the collected numbers dictionary and the path to `prisma_flow_data.md`) so that it is available to the Writer agents in Step 2. Writers should reference these numbers directly rather than re-deriving them from source files.

---

### Step 2: Dispatch Manuscript Writer A, Writer B, and Statistician in Parallel

**Writer A agent prompt:**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft the INTRODUCTION and METHODS sections.

TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
PROJECT TYPE: [from project.yaml]
REPORTING STANDARD: [from project.yaml review_config.reporting_standard]

SOURCE MATERIALS:
- Landscape report: [Read and insert 01_literature_search/landscape_report.md]
- Research question: [Read and insert 02_research_question/research_question.md]
- Criteria: [Read and insert 03_inclusion_exclusion/criteria.md]
- Search strategy: [Read and insert 04_database_search/search_strategy.md]
- Screening results summary: [Read and insert first 20 lines of 05_screening/screened_results.csv]
- PRISMA flow data: [Read and insert 07_manuscript/prisma_flow_data.md from Step 1b]

INTRODUCTION should:
- Establish the clinical context and significance
- Summarize the current state of knowledge (from landscape report)
- Identify the gap this study addresses
- State the research question and objectives

METHODS should:
- Follow the reporting standard checklist exactly
- Describe search strategy, databases, date range
- Describe screening process and criteria
- Describe data extraction process
- Describe synthesis/analysis approach
- Include PRISMA flow diagram data from Step 1b (reference exact numbers from prisma_flow_data.md rather than re-deriving)

Follow the journal's word limits and section structure.

OUTPUT: Introduction and Methods sections in markdown.
```

**Writer B agent prompt:**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft the RESULTS and DISCUSSION sections.

TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
PROJECT TYPE: [from project.yaml]
REPORTING STANDARD: [from project.yaml review_config.reporting_standard]

SOURCE MATERIALS:
- Research question: [Read and insert 02_research_question/research_question.md]
- Landscape report (for Discussion — existing literature comparison): [Read and insert 01_literature_search/landscape_report.md]
- Screening results: [Read and insert 05_screening/screened_results.csv]
- Extracted data: [Read and insert 06_data_extraction/extracted_data.csv]
- Synthesis report: [Read and insert 06_data_extraction/synthesis_report.md]
- Meta-analysis results (if exists): [Read and insert 06_data_extraction/meta_analysis_results.md]
- PRISMA flow data: [Read and insert 07_manuscript/prisma_flow_data.md from Step 1b]

RESULTS should:
- Present search/screening flow using exact PRISMA numbers from Step 1b (reference prisma_flow_data.md)
- Study characteristics summary (Table 1)
- Main findings organized by outcome
- Include all tables and figures referenced in synthesis report
- Meta-analysis results if applicable (forest plots, heterogeneity)

DISCUSSION should:
- Summarize key findings in context of the research question
- Compare with existing literature (cite specific reviews and studies from landscape_report.md)
- Discuss strengths and limitations
- Clinical implications
- Future research directions
- Conclusion

Follow the journal's word limits and section structure.

OUTPUT: Results and Discussion sections in markdown.
```

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Prepare all tables, figures, and statistical reporting text for the manuscript.

TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines — table/figure formatting requirements]

SOURCE MATERIALS:
- Extracted data: [Read and insert 06_data_extraction/extracted_data.csv]
- Synthesis report: [Read and insert 06_data_extraction/synthesis_report.md]
- Meta-analysis results (if exists): [Read and insert 06_data_extraction/meta_analysis_results.md]
- NMA results (if `review_config.network_meta_analysis` is `true` in project.yaml): [Read and insert 06_data_extraction/nma_results.md]

Prepare:
1. Table 1: Study Characteristics (formatted per journal requirements)
2. Table 2: Summary of Findings / GRADE table (formatted per journal requirements)
3. Risk of Bias table/figure
4. PRISMA flow diagram (as structured text)
5. Forest plots (if meta-analysis — as markdown tables referencing outputs in `analysis_scripts/outputs/`)
6. If `network_meta_analysis` is true: league table (all pairwise NMA estimates with 95% CI/CrI, direct vs. indirect annotated); SUCRA/P-score ranking table; network geometry figure legend; GRADE-NMA 7-domain certainty table
7. Any additional tables/figures needed
8. Statistical reporting text snippets that Writers can embed

Format all tables per the journal's specific requirements.

OUTPUT: All tables, figures, and statistical text in markdown.
```

### Step 3: Merge Step

Dispatch Writer A as a merge agent:

**Merge agent prompt:**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Merge the following manuscript sections into one cohesive document.

INTRODUCTION + METHODS:
[Insert Writer A's output]

RESULTS + DISCUSSION:
[Insert Writer B's output]

TABLES + FIGURES + STATISTICAL TEXT:
[Insert Statistician's output]

JOURNAL GUIDELINES: [Insert fetched guidelines summary]

Merge into a single manuscript with:
1. Title page (title, authors placeholder, affiliations placeholder, corresponding author)
2. Structured abstract (per journal format)
3. Keywords
4. Introduction
5. Methods
6. Results
7. Discussion
8. Conclusion
9. Declarations (funding, conflicts, ethics — placeholder text)
10. References
11. Tables
12. Figure legends

Ensure:
- Consistent voice and tense throughout
- No redundancy between sections
- Tables/figures are referenced in text
- Word counts are within journal limits
- Reporting checklist items are addressed

**SELF-CHECK (required before writing output to disk):**

Before finalizing the manuscript, perform a self-proofreading pass. Check each item and note any issues found (correct them inline before outputting):

Grammar & Writing Quality:
- [ ] No sentence fragments, run-ons, or grammatical errors
- [ ] Consistent tense: past tense in Methods and Results; present tense for established facts and Discussion
- [ ] No first-person voice unless journal guidelines explicitly permit it
- [ ] All abbreviations defined at first use and consistent throughout
- [ ] Numbers: spelled out below 10, numerals for 10 and above, always numerals with units
- [ ] SI units used throughout unless journal specifies otherwise

Structural Completeness:
- [ ] All required sections present and in the correct order per journal guidelines
- [ ] Abstract is within word limit and in the correct format (structured vs. unstructured)
- [ ] Main text is within word limit
- [ ] No section is missing its required subsections per the applicable reporting guideline

Citation & Reference Integrity:
- [ ] Every in-text citation has a corresponding reference in the reference list
- [ ] Every reference in the reference list is cited at least once in text
- [ ] No `[CITATION NEEDED]` flags remain unresolved
- [ ] References are formatted correctly per the journal's citation style (Vancouver, APA, etc.)
- [ ] Reference list is numbered sequentially in order of citation appearance (if Vancouver)

Statistical Reporting:
- [ ] All effect estimates reported with 95% CI and p-value
- [ ] Heterogeneity metrics (I², Q, tau², prediction interval) reported for all meta-analytic results
- [ ] No p-values reported without effect size context
- [ ] Statistically significant ≠ clinically significant — text does not conflate these
- [ ] GRADE certainty ratings included in the Results or Discussion if applicable
- [ ] If `network_meta_analysis` is true: league table with all pairwise NMA estimates present; SUCRA/P-score rankings reported; consistency test results (global + node-splitting) in Methods/Results; GRADE-NMA 7-domain table (including Indirectness and Incoherence) present; network geometry and NMA forest plot figures referenced by number

Tables & Figures:
- [ ] Every table and figure is referenced in the main text by number (Table 1, Figure 1, etc.)
- [ ] Table and figure numbers are sequential and consistent with the reference list order
- [ ] All table column headers are clear and include units where applicable
- [ ] Figure legends are self-contained (reader can understand the figure without reading main text)

Reporting Guideline Compliance:
- [ ] PRISMA 2020 flow diagram data is present in Methods/Results (if systematic review)
- [ ] PRISMA checklist items are all addressed (if systematic review or meta-analysis)
- [ ] PRISMA-ScR checklist items addressed (if scoping review)
- [ ] STROBE/CONSORT/CARE items addressed (for original research/case report)
- [ ] Study registration details (PROSPERO/ClinicalTrials.gov) reported in Methods if applicable

If any self-check item fails, correct it before producing the final output.

OUTPUT: Complete manuscript in markdown + references in BibTeX format.
```

### Step 4: Write Output

Write manuscript to `07_manuscript/manuscript.md`.
Write references to `07_manuscript/references.bib`.

### Step 4b: Proofreader Pass

Dispatch a Proofreader agent to catch linguistic, structural, and formatting issues that self-checks miss.

**Proofreader agent prompt:**

```
[Insert Proofreader system prompt from agent-roles.md]

TASK: Proofread the following manuscript before it goes to peer review.

MANUSCRIPT: [Read 07_manuscript/manuscript.md]
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary — word limits, reference style, section order]
REPORTING STANDARD: [from project.yaml review_config.reporting_standard]

Perform a full proofread. Check: grammar, tense consistency, abbreviation management, number/unit conventions, citation integrity, table/figure cross-references, journal compliance (word limits, section order), and reporting guideline checklist. Also flag AI-writing artifacts.

OUTPUT: Proofread report in the standard format.
```

Write proofread report to `07_manuscript/proofread_report.md`.

If the Proofreader finds **Errors** (not just Suggestions or Queries):
- Re-dispatch the merge agent with: (1) current `manuscript.md`, (2) the Proofreader's Error findings only, (3) instruction to apply each correction
- Update `07_manuscript/manuscript.md` with corrections
- Do NOT re-run the Proofreader — proceed to Full Review

### Step 5: Full Review

Dispatch Full Review per `references/reviewer-protocol.md`. Initialize `review_iteration = 0`.

**Review criteria for this stage (provide these to each reviewer in the dispatch prompt):**

Journal Compliance:
- Does the manuscript structure match the target journal's required section order?
- Is the abstract within word limits and in the correct format (structured/unstructured per journal)?
- Is the main text within the journal's word limit?
- Are references formatted correctly per the journal's citation style?
- Are the number of tables and figures within the journal's limits?

Reporting Guideline Compliance:
- Is every applicable PRISMA 2020 item addressed (for systematic reviews and meta-analyses)?
- Is every applicable PRISMA-ScR item addressed (for scoping reviews)?
- Is every applicable STROBE/CONSORT/CARE item addressed (for original research/case reports)?
- Is the study registration cited in Methods (if applicable)?
- Is the PRISMA flow diagram data present (search results, screening, included studies at each stage)?

Writing & Internal Consistency:
- Is the tense consistent (past for Methods/Results, present for Discussion of established facts)?
- Are all abbreviations defined at first use?
- Do all in-text citations appear in the reference list, and vice versa?
- Are any `[CITATION NEEDED]` flags left unresolved?
- Is the Discussion grounded in the Results — no claims made that are not supported by the extracted data?

Statistical Reporting:
- Are all effect estimates reported with 95% CI and p-value?
- Are heterogeneity metrics (I², Q, tau², prediction interval) reported for all pooled analyses?
- Is statistical significance correctly distinguished from clinical significance?
- Are GRADE certainty ratings present (if applicable)?

Tables & Figures:
- Are all tables and figures referenced by number in the main text?
- Is table and figure numbering sequential and consistent?
- Are figure legends self-contained?

**Autonomous Quality-Metric Iteration Loop (autoresearch-inspired):**

Instead of relying solely on reviewer verdicts, track measurable quality metrics across revision cycles. This ensures the pipeline converges toward a publishable manuscript rather than cycling indefinitely.

```
Initialize:
  review_iteration = 0
  quality_log = []

LOOP:
  1. DISPATCH Full Review (two independent reviewers per reviewer-protocol.md)

  2. MEASURE quality metrics from reviewer outputs:
     - critical_count = number of findings with Severity = "Critical"
     - major_count = number of findings with Severity = "Major"
     - minor_count = number of findings with Severity = "Minor"
     - quality_score = max(0, 100 - (critical_count × 25) - (major_count × 10) - (minor_count × 2))

  3. LOG: Append to quality_log:
     | Iteration | Critical | Major | Minor | Score | Verdict | Action |

  4. EVALUATE (try → measure → keep/discard logic):
     - APPROVE (majority verdict) OR quality_score ≥ 90 with 0 critical:
       → ACCEPT manuscript. Break loop. Proceed to Step 6.
     - REVISE with quality_score improving vs. previous iteration:
       → KEEP iterating. Build REVISION REQUIRED table. Re-dispatch merge agent.
       → Increment review_iteration.
     - REVISE with quality_score NOT improving (stalled or regressing):
       → STOP iterating. Escalate to user with quality_log showing the plateau.
     - REJECT or review_iteration ≥ 3:
       → STOP. Full escalation per reviewer-protocol.md.

  5. After merge agent revision:
     - Re-dispatch Proofreader on revised manuscript (catch new errors introduced by revision)
     - If Proofreader finds new Errors: apply fixes before next review cycle
     - Continue to top of LOOP
```

Write the quality log to `07_manuscript/revision_log.md`:

```markdown
# Manuscript Revision Log

| Iteration | Critical | Major | Minor | Score | Verdict | Changes Made |
|-----------|----------|-------|-------|-------|---------|-------------|
| 0 | 2 | 5 | 8 | 24 | REVISE | Initial review |
| 1 | 0 | 2 | 4 | 72 | REVISE | Fixed 2 critical (PRISMA flow, CI reporting), 3 major |
| 2 | 0 | 0 | 3 | 94 | APPROVE | Fixed remaining major issues |

Final quality score: **94/100**
```

On escalation, print the full quality log so the user can see exactly where the manuscript stalled and what remains unresolved.

### Step 6: Update project.yaml

Set `stages.manuscript_writing: completed`.

Print:

```
✅ Manuscript complete!

📄 Files:
- Manuscript: 07_manuscript/manuscript.md
- References: 07_manuscript/references.bib
- PRISMA flow data: 07_manuscript/prisma_flow_data.md
- PRISMA flow script: 07_manuscript/prisma_flow.py
- PRISMA flow figure: 07_manuscript/figures/prisma_flow.png

📋 NEXT STEPS:
1. Review the manuscript for accuracy and clinical nuance
2. Add author names, affiliations, and contact details
3. Complete declarations (funding, conflicts of interest, ethics approval)
4. Convert to the journal's required submission format (Word/LaTeX)
5. Submit!
```
