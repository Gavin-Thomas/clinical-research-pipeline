# Abstract Screening — Diagnostic Test Accuracy Process

_Read and follow this file when `project_type: diagnostic_test_accuracy`. DTA screening uses the same 3-extractor parallel structure as systematic reviews, but applies a PICOS-based decision framework with DTA-specific design exclusions (case-control, partial verification) and a `threshold_reported` capture field._

---

## Step 1: Dispatch 3 Data Extractor Agents in Parallel

Use the same parallel dispatch structure as the Review Types process. **Replace the decision framework block** in the Data Extractor prompt with:

```
DECISION RULES (DTA-specific — replace the generic INCLUDE/EXCLUDE/UNCERTAIN block):

- INCLUDE: Study applied both the index test AND the reference standard to the same participants
  (or nearly all participants); study design is cross-sectional, prospective cohort, or retrospective cohort

- EXCLUDE (cite the specific criterion from criteria.md):
  • Case-control design — high spectrum bias risk (typical exclusion criterion: "Study design")
  • Partial verification: reference standard applied only to index-test-positive cases — cite "Verification bias" criterion
  • Population clearly outside PICOS scope

- UNCERTAIN: Abstract describes the index test but does not confirm whether the reference standard
  was also applied; or threshold information is absent but plausibly present in full text.
  When in doubt, INCLUDE — full-text review resolves threshold and verification questions.

ADD an extra field to each output row:
  threshold_reported: The numeric threshold mentioned in the abstract (e.g., "≥7 out of 10",
  "AUC ≥ 0.80"), or "NR" if not stated. This helps Stage 6 document threshold heterogeneity.
```

**Output CSV columns** (add `threshold_reported` after `criterion`):
`id, title, decision, criterion, threshold_reported, reasoning, confidence`

Use the same SELF-CHECK block as the Review Types process, plus:
- Verify: every INCLUDE/UNCERTAIN row has a non-blank `threshold_reported` (value or "NR" — never blank).
- Verify: case-control designs are EXCLUDE with "spectrum bias" cited, not UNCERTAIN.

---

## Step 2: Dispatch Methodologist for Reconciliation

Same majority-vote reconciliation as the Review Types process. Add to the Methodologist prompt:

```
ADDITIONAL INSTRUCTIONS for DTA reconciliation:

`reconciliation_notes` must document disagreements about:
- Study design classification (cross-sectional vs. case-control vs. retrospective cohort)
- Partial verification concerns (reference standard not applied to all participants)
- Threshold ambiguity (abstract suggests threshold but does not report it explicitly)

For the `threshold_reported` field in the reconciled output: use the most specific value agreed upon by ≥2 extractors; if all 3 differ, use "NR" and note the discrepancy.
```

**Reconciled CSV columns:**
`id, title, final_decision, threshold_reported, vote_count, conflict_flag, reconciliation_notes`

---

## Step 3: Write Output

Write/append reconciled results to `05_screening/screened_results.csv`.
Update `05_screening/screening_progress.yaml` if batching.

---

## Step 4: Full Review (final batch only)

Initialize `review_iteration = 0`. Dispatch Full Review per `references/reviewer-protocol.md`.

**Review criteria for DTA screening (replace standard criteria):**
- Were case-control designs correctly excluded (spectrum bias)? Any case-control in INCLUDE is a REJECT trigger. ✗✗
- Were partial-verification studies excluded if that criterion was listed in criteria.md?
- Is the `threshold_reported` field populated (value or "NR") for all INCLUDE/UNCERTAIN rows?
- Are UNCERTAIN decisions limited to genuine design ambiguity or absent threshold — not used as a hedge for clearly out-of-scope studies?
- Do conflict reconciliation notes address spectrum bias and partial verification specifically?

Follow the same REVISE/REJECT iteration loop per `references/reviewer-protocol.md`.

---

## Step 5: Update project.yaml + Manual Handoff

Set `stages.abstract_screening: completed`.

Print:

```
✅ Abstract screening complete (diagnostic test accuracy review). Results saved to 05_screening/screened_results.csv

📊 Summary (for PRISMA flow diagram):
- Records identified (pre-dedup): [N] — see deduplication_report.md
- Duplicates removed: [N]
- Records screened: [N_post_dedup]
- Included: [N]
- Excluded: [N] (breakdown: [N] case-control design / [N] partial verification / [N] population / [N] other)
- UNCERTAIN (for full-text clarification): [N]
- Conflicts for human review: [N]
- Inter-rater agreement: [kappa]

Threshold reporting in included studies:
- Threshold explicitly reported in abstract: [N]
- Threshold not reported (NR — check full text): [N]

📋 MANUAL STEP REQUIRED:
1. Review CONFLICT items in screened_results.csv
2. Pull full-text PDFs for all INCLUDED and UNCERTAIN articles (DOIs/PMIDs in screened_results.csv)
   — UNCERTAIN articles require full-text confirmation before final inclusion
3. Place PDFs in: 06_data_extraction/full_texts/

When you're done, run /data-extraction to continue.
```
