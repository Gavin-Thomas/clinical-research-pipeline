# Abstract Screening — Rapid Review Process

_Read and follow this file when `project_type: rapid_review`. Rapid review screening uses a **single Data Extractor** (no triple-independent redundancy) with a **10% verification audit** and **Quick Review** (not Full Review). These are documented methodological shortcuts — record them in the manuscript Limitations section per PRISMA 2020 item 27._

---

## Step 1: Dispatch Single Data Extractor

Dispatch **one** Data Extractor agent (not three) using the same prompt as the systematic review process, with this note prepended:

```
NOTE — RAPID REVIEW SINGLE-SCREENER MODE:
You are the sole screener for this rapid review. Apply the criteria rigorously.
Use UNCERTAIN liberally rather than EXCLUDE when information is borderline.
Your output will undergo a 10% verification audit by a Methodologist.
```

Use the same Data Extractor prompt structure as the Review Types process (criteria.md, research_question.md context, INCLUDE/EXCLUDE/UNCERTAIN decision rules, SELF-CHECK block).

**Output CSV columns:** `id, title, decision, criterion, reasoning, confidence`

---

## Step 2: Dispatch Methodologist for 10% Verification Audit

After the Data Extractor's CSV is received, select a random 10% sample (minimum 20 records, or all records if total ≤ 20). Dispatch a Methodologist agent:

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Rapid review verification audit — check a 10% random sample of screening decisions.

ELIGIBILITY CRITERIA:
[Read and insert 03_inclusion_exclusion/criteria.md]

SCREENING DECISIONS TO AUDIT (10% sample):
[Insert sampled CSV rows with id, title, abstract_text, decision, criterion, reasoning]

For each sampled record:
1. Verify that the decision is consistent with the stated criteria.
2. Flag any decision that appears incorrect (wrong decision, wrong criterion cited, vague reasoning).
3. If ≥10% of sampled records are flagged: recommend re-screening the full set before proceeding.
   If <10% flagged: note the findings and approve proceeding.

OUTPUT:
- audit_pass: true | false (true if <10% of sampled records are flagged)
- flagged_records: list of IDs with correct_decision and reason for each
- audit_summary: overall assessment in 2-3 sentences
```

**If `audit_pass: false`:** Re-dispatch the single Data Extractor on the full set, prepending the Methodologist's flagged records as correction context. Run the audit again (max 1 re-audit).

**If `audit_pass: true`:** Proceed to Step 3.

---

## Step 3: Write Output

Write results to `05_screening/screened_results.csv`.

Add a note at the top of the CSV (as a comment row starting with `#`):

```
# Rapid review — single screener with 10% Methodologist verification audit.
# Methodological shortcut documented for manuscript Limitations section.
# Audit result: [audit_pass] — [N] records sampled, [N] flagged.
```

Update `05_screening/screening_progress.yaml` if batching.

---

## Step 4: Quick Review (on final batch only)

After all batches are complete, use **Quick Review** (not Full Review) per project-types.md. Initialize `review_iteration = 0`. Dispatch a single Independent Reviewer per `references/reviewer-protocol.md`.

**Review criteria for rapid review screening:**
- Are INCLUDE/EXCLUDE decisions consistent with the stated criteria?
- Were obviously eligible studies excluded?
- Is UNCERTAIN used appropriately rather than EXCLUDE for ambiguous cases?
- Is the single-screener audit trail documented?

On **REVISE:** Re-screen only the flagged abstracts using the single Data Extractor with the reviewer's findings prepended. Increment `review_iteration`. Maximum 2 iterations per reviewer-protocol.md Quick Review rules.

On **REJECT** or `review_iteration ≥ 2` without APPROVE: escalate per `reviewer-protocol.md`.

---

## Step 5: Update project.yaml + Manual Handoff

Set `stages.abstract_screening: completed`.

Append to `07_manuscript/rapid_review_deviations.md` (create if not exists):

```markdown
## Stage 5 — Abstract Screening Deviation

- **Standard practice:** Dual or triple-independent abstract screening with consensus reconciliation.
- **Rapid review shortcut:** Single-screener with 10% random verification audit by an independent Methodologist.
- **Potential impact:** Inter-rater agreement not calculable; single-screener errors may not be detected. Risk mitigated by liberal use of UNCERTAIN and bias toward INCLUDE.
- **Audit result:** [audit_pass] — [N]% of sampled records flagged; [N] corrections applied.
```

Print:

```
✅ Abstract screening complete (rapid review — single screener). Results: 05_screening/screened_results.csv

📊 Summary (for PRISMA flow diagram):
- Records identified (pre-dedup): [N] — see deduplication_report.md
- Duplicates removed: [N]
- Records screened: [N_post_dedup]
- Included after screening: [N]
- Excluded after screening: [N]
- 10% audit: [audit_pass] — [N] records audited, [N] corrections applied

⚠️  Methodological shortcut logged to 07_manuscript/rapid_review_deviations.md
    (Include this in your Limitations section per PRISMA 2020 item 27)

📋 MANUAL STEP REQUIRED:
1. Pull full-text PDFs for all INCLUDED articles (see screened_results.csv for DOIs/PMIDs)
2. Place PDFs in: 06_data_extraction/full_texts/

When done: "I've added the PDFs to the full_texts folder — please continue"
```
