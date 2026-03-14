---
name: abstract-screening
description: |
  Use when the user wants to screen abstracts, apply inclusion/exclusion criteria to search results,
  title/abstract screening, or determine which studies to include in a review.
  Also triggers for data cleaning in original research projects.
  Trigger on phrases like "screen abstracts", "screen the results", "apply criteria",
  "title abstract screening", "which studies to include", or "filter the results".
---

# Abstract Screening (Stage 5)

Screen abstracts against inclusion/exclusion criteria using a team of 3 independent Data Extractors.

## Prerequisites

- `03_inclusion_exclusion/criteria.md` must exist
- Files must exist in `04_database_search/abstracts/` (user's manual step)
- Read `project.yaml` for `project_type`

## Abstract Parsing

Read all files in `04_database_search/abstracts/` and parse based on file extension:

- **`.csv` files**: Expect columns for ID/PMID, Title, Authors, Abstract, Journal, Year, DOI. Read with the Read tool. Parse rows as individual abstracts.

- **`.ris` files**: Parse RIS tags (TY, TI, AU, AB, JO, PY, DO). Each record delimited by `ER  -`.

- **`.nbib` files** (PubMed/NLM export format — the default PubMed export): Parse the NLM flat-file format. Each record begins with a `PMID-` tag and records are separated by blank lines. Extract these tags:
  - `PMID- ` → id (prefix with "PMID", e.g., "PMID12345678")
  - `TI  - ` → title (may span multiple continuation lines — concatenate lines that do not start with a 4-char tag)
  - `AU  - ` → authors (multiple `AU` lines per record — join with "; ")
  - `AB  - ` → abstract_text (may span multiple continuation lines)
  - `JT  - ` → journal (full journal title; fall back to `TA  - ` abbreviated title if `JT` absent)
  - `DP  - ` → year (extract the first 4-digit number from the date string, e.g., "2023 Jan" → 2023)
  - `AID - ` (with suffix `[doi]`) → doi (strip the trailing ` [doi]`; prefer `AID` over `LID`; fall back to `LID - ` with `[doi]` suffix if `AID` absent)
  - If a continuation line starts with whitespace (no 4-char tag), append it to the previous field's value.

- **`.txt` files**: First check whether the file contains PubMed-format tags by scanning for a line matching `PMID- `. If found, parse as NLM flat-file format (same rules as `.nbib` above). Otherwise, treat as a single-abstract file: the filename is the identifier and the entire file content is the abstract_text.

Compile a unified list of abstracts with fields: `id, title, authors, abstract_text, journal, year, doi`.

## Deduplication

After parsing all abstract files into the unified list, automatically deduplicate before screening. This produces the PRISMA pre/post-deduplication counts required for PRISMA flow reporting.

### Deduplication Logic

Apply in two passes:

**Pass 1 — DOI-based exact deduplication:**
For all records with a non-empty DOI field: normalize DOIs to lowercase and strip leading/trailing whitespace. Group records sharing the same normalized DOI. Within each DOI group: keep the record with the most non-empty fields; mark remaining duplicates as DUPLICATE.

**Pass 2 — Fuzzy title+year deduplication (records without DOI):**
For records with no DOI after Pass 1: compare every unmatched pair. Normalize titles: lowercase, remove punctuation, collapse whitespace. Two records are likely duplicates if: same year AND title similarity ≥ 85% (character-overlap ratio: `2 × shared_chars / (len(title_A) + len(title_B))`). Within each match group: keep the most complete record; mark the rest DUPLICATE.
If title similarity is 70–84%: flag as `POSSIBLE_DUPLICATE` — do NOT remove. Keep in the screening pool and add `[POSSIBLE DUPLICATE of record [id]]` to the `reasoning` field at screening time.

### Deduplication Output

Write `05_screening/deduplication_report.md`:

```markdown
# Deduplication Report

| Metric | Count |
|--------|-------|
| Records identified (all databases, pre-deduplication) | [N] |
| Records removed — DOI-exact duplicates | [N] |
| Records removed — fuzzy title+year duplicates | [N] |
| Possible duplicates flagged for human review | [N] |
| **Records proceeding to screening** | **[N]** |

## Database Breakdown (pre-deduplication)
| File | Format | Records |
|------|--------|---------|
| [filename] | PubMed / Ovid / Embase | [N] |

## Duplicate Groups Removed
| Kept ID | Duplicate ID(s) | Method | Basis |
|---------|-----------------|--------|-------|
| [id] | [id, id] | DOI-exact | doi:10.xxxx/xxxx |
| [id] | [id] | Fuzzy-title | "Effect of X…" — 93% match, year [YYYY] |

## Possible Duplicates (human review recommended)
| Record A | Record B | Similarity | Titles |
|----------|----------|------------|--------|
| [id] | [id] | 78% | [title A] / [title B] |
```

Print before proceeding to screening:
```
📋 Deduplication complete:
  Records parsed: [N_pre]  →  Records for screening: [N_post]
  Removed: [N_dup] duplicates ([N_doi] DOI-exact, [N_fuzzy] fuzzy title+year)
  Flagged as possible duplicates: [N_possible] (kept — verify at full-text stage)
  PRISMA counts saved to: 05_screening/deduplication_report.md
```

## Edge Cases (Check Before Processing)

### Empty Abstracts Folder

If `04_database_search/abstracts/` contains no files, print and stop:

```
❌ No abstract files found in 04_database_search/abstracts/

Expected formats: .csv, .ris, or .txt files exported from PubMed/Ovid/Embase.
Please complete Manual Step 1 (database searching) before running abstract screening.

If you have placed files elsewhere, move them to: 04_database_search/abstracts/
```

### Parse Failures

If a record cannot be parsed (missing title, missing abstract, malformed RIS tag):
- Record it in `05_screening/parse_failures.csv` with columns: `raw_id, filename, reason`
- Skip it in the screening pass (do not assign INCLUDE/EXCLUDE)
- Report count of parse failures in the final summary message
- If >10% of records fail to parse, warn the user that the file may be in an unsupported format

### Screening Volume Context

Typical high-quality systematic reviews screen 1,000–3,000 records after deduplication.
Scoping reviews may screen 2,000–5,000+. Approximately 90% of records are excluded at
title/abstract screening (3–5% final inclusion rate is normal). The pipeline must handle
these volumes without recommending users narrow searches that are appropriately sensitive.

### Volume-Aware Handling by Project Type

After deduplication, check `project_type` and apply the appropriate guidance:

| project_type | Expected range | Flag if below | Warn (may be broad) | Alert (very large) |
|---|---|---|---|---|
| systematic_review / meta_analysis | 200–2,000 | <100 (may miss studies) | >2,000 | >5,000 |
| scoping_review | 500–5,000 | <200 | >5,000 | >10,000 |
| rapid_review | 50–500 | <30 | >500 | >1,000 |
| qualitative_synthesis | 100–1,000 | <50 | >1,000 | >2,000 |
| diagnostic_test_accuracy | 100–1,000 | <50 | >1,000 | >2,000 |

Actions:
- **Below expected range**: Print warning — search may be too restrictive; check if key databases were missed or search terms too narrow. Do NOT auto-recommend narrowing.
- **Within expected range**: Proceed normally. Print: "📊 [N] records to screen — typical for a [project_type]."
- **Warn threshold**: Print advisory — screening is feasible but substantial. Suggest the user confirm they want to proceed or consider adding study design filters. Do NOT recommend narrowing for systematic_review or meta_analysis — comprehensive search is methodologically required.
- **Alert threshold**: Print alert — screening volume is very large. Offer options: (a) proceed with full screening (pipeline handles it), (b) add filters to reduce volume, (c) consider whether this should be a scoping review instead. For scoping_review, large volumes are expected — just confirm.

### Inclusion Rate Monitoring

After the first 2 batches (not just batch 1), calculate the running inclusion rate:
- If inclusion rate > 50% AND N > 200: flag — criteria may be too permissive (expected: 5–15% inclusion)
- If inclusion rate < 2% AND N > 200: flag — criteria may be too restrictive or search strategy misaligned with PICO
- Save an interim summary `05_screening/screening_summary_interim.md` after every 5 batches with running totals (screened, included, excluded, conflicts, running inclusion rate)

## Batching

Batch size scales with screening volume to balance thoroughness and efficiency:

| Total records | Batch size | Rationale |
|---|---|---|
| ≤100 | 50 | Small set — standard batching |
| 101–500 | 75 | Moderate — slightly larger batches |
| 501–2,000 | 100 | Large — reduce total batch count |
| >2,000 | 150 | Very large — maximize throughput while maintaining screening quality |

If the total number of abstracts exceeds the batch size:
1. Create `05_screening/screening_progress.yaml` with `total_items`, `batch_size: [from table above]`, `batches_completed: 0`, `estimated_batches: [total/batch_size]`
2. Process one batch per invocation
3. Each subsequent invocation reads `screening_progress.yaml` and picks up the next batch
4. Append results to `05_screening/screened_results.csv` (don't overwrite previous batches)
5. Print progress after each batch: "📊 Batch [N]/[total_batches] complete — [screened]/[total] records screened ([included] included so far)"

## Project-Type Routing

> **Check `project_type` in `project.yaml` before proceeding:**
>
> | project_type | Instructions |
> |---|---|
> | `qualitative_synthesis` | Read `skills/abstract-screening/section-qualitative-synthesis.md` and follow it exactly |
> | `diagnostic_test_accuracy` | Read `skills/abstract-screening/section-dta.md` and follow it exactly |
> | `rapid_review` | Read `skills/abstract-screening/section-rapid-review.md` and follow it exactly |
> | `original_research` | Read `skills/abstract-screening/section-original-research.md` and follow it exactly |
> | `case_report` | This stage is skipped — see Process — Case Report below |
> | all others (systematic_review, scoping_review, meta_analysis) | Continue with Process — Review Types below |

Read the appropriate sub-file now (if applicable), then execute its steps in full.

---

## Process — Review Types (systematic_review, scoping_review, meta_analysis)

### Step 1: Dispatch 3 Data Extractor Agents in Parallel

For each batch of abstracts, dispatch 3 Data Extractor agents simultaneously. Each gets ALL abstracts in the batch.

**Data Extractor agent prompt (same for all 3):**

```
[Insert Data Extractor system prompt from agent-roles.md]

TASK: Screen the following abstracts against the inclusion/exclusion criteria.

RESEARCH QUESTION & PICO CONTEXT:
[Read and insert 02_research_question/research_question.md]
Use this to resolve borderline UNCERTAIN cases: ask "Is this abstract plausibly studying
the population, intervention/exposure, and outcomes defined in the PICO above?"

INCLUSION/EXCLUSION CRITERIA:
[Read and insert 03_inclusion_exclusion/criteria.md]

ABSTRACTS TO SCREEN:
[Insert batch of abstracts]

For EACH abstract, produce a CSV row:
id, title, decision, criterion, reasoning, confidence

Where:
- decision: INCLUDE | EXCLUDE | UNCERTAIN
- criterion: The specific inclusion/exclusion criterion that determined the decision
- reasoning: 1-2 sentence explanation
- confidence: HIGH | MEDIUM | LOW

Guidance for UNCERTAIN decisions: use UNCERTAIN only when the abstract genuinely lacks
the information needed to apply a criterion — not when the study seems "tangentially related".
When in doubt, prefer INCLUDE over EXCLUDE; the full-text review stage is the safety net.

Screen independently. Do not assume other screeners' decisions.

OUTPUT: CSV with header row followed by one row per abstract.

SELF-CHECK before finalizing your output:
1. Coverage: Every abstract in the batch has exactly one decision row — no gaps, no duplicates.
2. Criterion alignment: Each EXCLUDE decision cites a specific exclusion criterion from criteria.md — not a vague "not relevant" rationale.
3. UNCERTAIN discipline: UNCERTAIN is only used when the abstract genuinely lacks the information to apply a criterion — not as a hedge for borderline cases.
4. Confidence calibration: HIGH confidence requires a clear criterion match; LOW confidence should accompany UNCERTAIN or genuinely close calls.
5. INCLUDE bias check: Verify that when the abstract was borderline, INCLUDE was assigned (not EXCLUDE) — the full-text stage is the safety net.
Correct any violations inline before submitting output.
```

### Step 2: Dispatch Methodologist for Reconciliation

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Reconcile screening results from 3 independent data extractors.

EXTRACTOR 1 RESULTS:
[Insert Extractor 1's CSV output]

EXTRACTOR 2 RESULTS:
[Insert Extractor 2's CSV output]

EXTRACTOR 3 RESULTS:
[Insert Extractor 3's CSV output]

For each abstract:
1. Apply MAJORITY VOTE: If 2+ extractors agree, that's the decision
2. Flag CONFLICTS: If all 3 disagree, mark as CONFLICT for human review
3. Calculate AGREEMENT: Report overall inter-rater agreement

OUTPUT: Reconciled CSV with columns:
id, title, final_decision, vote_count, conflict_flag, reconciliation_notes

Also report: total screened, included, excluded, conflicts, agreement rate.

SELF-CHECK before finalizing your output:
1. Completeness: Every abstract ID from all three extractor inputs appears exactly once in your reconciled CSV — no missing rows.
2. Vote accuracy: Spot-check 5 random rows and confirm final_decision matches the majority vote (2+ extractors).
3. Conflict detection: Confirm that every 3-way disagreement (all 3 extractors differ) is marked as CONFLICT with vote_count reflecting the split.
4. Agreement rate: Verify the reported agreement rate is arithmetically consistent with the conflict count (agreements / total screened ≈ 1 − conflicts/total).
5. Notes quality: Every CONFLICT row must have a reconciliation_notes entry describing what the disagreement was about (e.g., "Extractor 1 excluded on population; Extractors 2–3 included pending full-text review").
Correct any errors inline before outputting the final CSV.
```

### Step 3: Write Output

Write/append reconciled results to `05_screening/screened_results.csv`.
Update `05_screening/screening_progress.yaml` if batching.

### Step 4: Full Review (on final batch only)

After all batches are complete, initialize `review_iteration = 0`. Dispatch Full Review per `references/reviewer-protocol.md`.

Select a random ~20% sample of screening decisions. Provide the sample + original abstracts + criteria to reviewers.

**Review criteria for this stage:**
- Are INCLUDE/EXCLUDE decisions consistent with the stated criteria?
- Were any clearly eligible studies excluded?
- Were any clearly ineligible studies included?
- Is the agreement rate acceptable (kappa > 0.6)?

**On REVISE verdict:** Re-screen only the flagged abstracts using a single Data Extractor. Prepend the reviewer's findings table to the extractor prompt as:

```
REVISION REQUIRED — Reviewer findings (iteration [review_iteration]):
[Insert reviewer's findings table with flagged IDs, original decisions, and reviewer concerns]

For each flagged abstract, provide a revised decision and explicitly address the reviewer's concern in the reasoning column.
```

Update `05_screening/screened_results.csv` with revised rows. Increment `review_iteration`. Re-dispatch reviewer with the full updated results and a revision history block.

**On REJECT verdict or `review_iteration >= 2` with non-APPROVE:** Escalate per `reviewer-protocol.md` — print full escalation banner with all reviewer findings, revision history, and pipeline pause. Do not auto-revise further; human intervention required.

### Step 5: Unpaywall OA PDF Auto-Discovery

After screening is complete, automatically check Unpaywall for open-access PDFs of all included studies. Read `references/api-integrations.md` for endpoint details.

**Process:**

1. Read `05_screening/screened_results.csv` — filter to rows where `final_decision = INCLUDE`
2. For each included study with a non-empty DOI:
   - WebFetch: `https://api.unpaywall.org/v2/{DOI}?email=pipeline@research.edu`
   - Parse the JSON response:
     - `is_oa` → boolean
     - `best_oa_location.url_for_pdf` → direct PDF URL (may be null even if is_oa is true)
     - `best_oa_location.url_for_landing_page` → fallback landing page
     - `best_oa_location.host_type` → "publisher" or "repository"
   - Rate limit: add a 200ms delay between requests
   - If WebFetch fails for a DOI, log the failure and continue

3. Write results to `05_screening/oa_pdf_links.csv`:
   ```csv
   record_id,doi,is_oa,pdf_url,landing_page_url,host_type
   ```

4. Count: `n_oa` = OA PDFs found; `n_total` = total included with DOIs

### Step 6: Update project.yaml + Manual Handoff

Set `stages.abstract_screening: completed`.

Print:

```
✅ Abstract screening complete. Results saved to 05_screening/screened_results.csv

📊 Summary (for PRISMA flow diagram):
- Records identified (databases, pre-dedup): [N] — see deduplication_report.md for breakdown
- Duplicates removed: [N]
- Records screened: [N_post_dedup]
- Included after screening: [N]
- Excluded after screening: [N]
- Conflicts (for human review): [N]
- Inter-rater agreement: [kappa]

🔓 Open Access: [n_oa]/[n_total] PDFs freely available (see oa_pdf_links.csv)

📋 MANUAL STEP REQUIRED:
1. Review any CONFLICT items in screened_results.csv
2. Download OA PDFs from links in: 05_screening/oa_pdf_links.csv
3. For remaining paywalled articles, use institutional access or ILL
4. Place ALL PDFs in: 06_data_extraction/full_texts/

When you're done, run /data-extraction to continue.
```

---

## Process — scoping_review

Same agent structure as systematic review (3 extractors + Methodologist reconciliation), but the extractor prompt changes. Replace the screening prompt's decision framework:

Instead of `INCLUDE | EXCLUDE | UNCERTAIN`, use:
- decision: `MAP` (relevant to the scope) | `OUT_OF_SCOPE` | `UNCERTAIN`
- Add a `category` column to the CSV output for charting: extractors assign each abstract to one or more topic categories based on the research question's scope

The Methodologist reconciliation step reconciles both the MAP/OUT_OF_SCOPE decision AND the category assignments via majority vote.

Output CSV columns: `id, title, decision, category, criterion, reasoning, confidence`

---

## Process — case_report

This stage is skipped for case reports. The skill should detect `project_type: case_report` and print:

```
ℹ️ Stage 5 (Abstract Screening) is skipped for case reports.
Place your clinical materials in 06_data_extraction/full_texts/ and run /data-extraction.
```
