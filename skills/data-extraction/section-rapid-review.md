# Data Extraction — Rapid Review Process

Rapid review data extraction uses a **single Data Extractor** (no triple-independent partitioning), a **Quick Review** (not Full Review), and **descriptive synthesis only** (no meta-analytic pooling). These are documented methodological shortcuts — record them for the manuscript Limitations section.

---

## Step 1: Dispatch Statistician for Abbreviated Template Design

Same prompt as the standard review-types path, with this modification:

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Design an abbreviated data extraction template for a rapid review.

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

PROJECT TYPE: rapid_review
REPORTING STANDARD: PRISMA (with documented methodological shortcuts)
META-ANALYSIS: false (not performed in rapid reviews)

Design a CSV extraction template capturing the minimum dataset needed for narrative synthesis:
1. Study identification: author, year, title, journal, DOI, country
2. Study design and setting
3. Population: sample size, age, condition/severity
4. Intervention/exposure and comparator
5. Primary outcome(s): effect estimates (any format as reported), p-values
6. Risk-of-bias assessment using the most appropriate abbreviated tool:
   - RCTs: first 2 RoB 2 domains only (randomization and deviations from intended interventions) — flag as "abbreviated RoB 2 for rapid review"
   - Observational studies: NOS selection and comparability domains — flag as "abbreviated NOS for rapid review"

NOTE: Omit columns for secondary meta-analytic inputs (standard errors, variance, subgroup counts)
since pooling will not be performed.

If `review_config.clinical_coding` is `true`, add clinical coding columns as specified in the
standard template design prompt.

OUTPUT: CSV header row + example row.
```

Write template to `06_data_extraction/extraction_template.csv`.

---

## Step 2: Dispatch Single Data Extractor

Dispatch **one** Data Extractor agent (not three — no partitioning). Use the same prompt as the standard review-types path, with this note prepended:

```
NOTE — RAPID REVIEW SINGLE-EXTRACTOR MODE:
You are the sole data extractor for this rapid review. There is no triple-independent
redundancy check. Apply the template rigorously and document all sources precisely.
Use "NR" only for data genuinely absent from the paper — do not skip hard-to-find data.

[Insert standard Data Extractor prompt, including SELF-CHECK block]
```

All papers are assigned to this single extractor. If the paper count exceeds 10, batch at 10 per invocation and track progress in `06_data_extraction/extraction_progress.yaml`.

---

## Step 3: Merge Extracted Data

Merge all extractor output into `06_data_extraction/extracted_data.csv`. (For rapid reviews this is trivially a direct copy since there is only one extractor — still verify the file was written.)

---

## Step 4: Quick Review with Retry Logic

Use **Quick Review** (not Full Review) per `references/reviewer-protocol.md`.
Initialize `review_iteration = 0`.

Dispatch a single Independent Reviewer. Provide the extracted data CSV plus the extraction template.

**Review criteria for rapid review extraction:**
- Does extracted data match what is in the source PDFs? Check a 20% random sample.
- Are "NR" entries genuinely not reported (not data that was hard to find)?
- Are effect sizes, CIs, and p-values transcribed character-for-character?
- Is the abbreviated risk-of-bias assessment documented (abbreviated tool flagged)?
- Are source page/table references present for primary outcomes?

On **REVISE:** build a REVISION REQUIRED block (finding | affected paper IDs | column names). Re-dispatch the single Data Extractor for flagged rows only, prepending the REVISION REQUIRED block. Update `extracted_data.csv`. Increment `review_iteration`. Re-dispatch reviewer with revision history appended. Maximum 2 iterations per Quick Review rules.

On **APPROVE:** proceed to Step 5.

On **REJECT** or `review_iteration ≥ 2` without APPROVE: trigger escalation per `references/reviewer-protocol.md` (print escalation banner, list unresolved findings, pause pipeline).

---

## Step 5: Dispatch Statistician for Descriptive Synthesis

**Statistician agent prompt:**

```
[Insert Statistician system prompt from agent-roles.md]

TASK: Produce a descriptive narrative synthesis for this rapid review.

EXTRACTED DATA:
[Read and insert 06_data_extraction/extracted_data.csv]

PROJECT TYPE: rapid_review
META-ANALYSIS: false — do NOT perform statistical pooling.

Produce:
1. STUDY CHARACTERISTICS TABLE: All included studies summarized (study ID, design, country,
   N, population, intervention/exposure, comparator, primary outcome, follow-up, abbreviated
   risk-of-bias rating). Format as Table 1.

2. NARRATIVE SYNTHESIS: Organize by outcome domain. Describe direction and magnitude of
   findings across studies in plain language. Note any apparent trends or patterns.
   Do NOT calculate pooled estimates or present forest plots — this is narrative synthesis only.

3. RISK-OF-BIAS SUMMARY: Tabulate the abbreviated risk-of-bias scores from the extracted data.
   Note proportion of studies at high/unclear/low risk per domain assessed. Add a caveat
   that abbreviated appraisal was used (single-domain RoB 2 / partial NOS) and that comprehensive
   quality assessment was not performed for this rapid review.

4. DISCREPANT FINDINGS: Flag any studies with contradictory results on the same outcome.
   Document the magnitude of disagreement and most likely methodological explanation.
   Do NOT average or dismiss conflicting values.

SELF-CHECK before writing synthesis_report.md:
1. Every study in extracted_data.csv appears in the characteristics table — none omitted
2. Narrative synthesis addresses every outcome named in the research question
3. Risk-of-bias summary is based on actual scores from extracted data, not generic statements
4. Discrepant findings, if any, are documented with specific study comparisons
5. No pooled estimates are presented — this is narrative only
6. Abbreviated appraisal caveat is included

OUTPUT: Structured synthesis report in markdown.
```

Write to `06_data_extraction/synthesis_report.md`.

---

## Step 6: Update project.yaml + Log Deviation

Set `stages.data_extraction: completed`.

Append to `07_manuscript/rapid_review_deviations.md` (create if not exists):

```markdown
## Stage 6 — Data Extraction Deviation

- **Standard practice:** Triple-independent data extraction with consensus reconciliation; comprehensive risk-of-bias appraisal (full RoB 2 or ROBINS-I) for all included studies.
- **Rapid review shortcuts:**
  1. Single Data Extractor (no redundancy check). Risk mitigated by Quick Review audit of a 20% sample.
  2. Abbreviated risk-of-bias appraisal (first 2 RoB 2 domains only for RCTs; selection + comparability NOS domains only for observational studies). Full domain assessment not performed.
  3. Descriptive narrative synthesis only — no meta-analytic pooling, no heterogeneity statistics, no GRADE assessment.
- **Potential impact:** Single-extractor errors may not be detected for all studies. Abbreviated bias assessment may miss outcome measurement and reporting bias domains. Absence of pooled estimates limits quantitative precision.
- **Reviewer:** Quick Review (1 reviewer, max 2 iterations) applied to a 20% extraction sample.
```

Print:

```
✅ Data extraction complete (rapid review — single extractor, descriptive synthesis).

Files:
  Extraction template:  06_data_extraction/extraction_template.csv
  Extracted data:       06_data_extraction/extracted_data.csv
  Synthesis report:     06_data_extraction/synthesis_report.md

⚠️  Methodological shortcuts logged to 07_manuscript/rapid_review_deviations.md
    — Include these in your manuscript Limitations section (PRISMA 2020 item 27).

Shortcuts taken:
  • Single Data Extractor (no triple-independent redundancy)
  • Abbreviated risk-of-bias appraisal
  • Descriptive synthesis only (no meta-analytic pooling or GRADE)
```
