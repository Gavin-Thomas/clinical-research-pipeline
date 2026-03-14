# Inclusion/Exclusion — Diagnostic Test Accuracy

_Read and follow this file when `project_type: diagnostic_test_accuracy`. DTA reviews require PICOS framing and specific eligibility rules around spectrum bias and verification bias._

## Step 1: Dispatch Methodologist Agent

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Draft eligibility criteria for a diagnostic test accuracy systematic review using the PICOS framework.

RESEARCH QUESTION (PICOS):
[Read and insert 02_research_question/research_question.md]

PROJECT TYPE: diagnostic_test_accuracy
REPORTING STANDARD: STARD 2015

Draft criteria covering all PICOS components:
1. POPULATION (P): Patients suspected of having the target condition. Specify: spectrum of disease severity eligible, clinical setting (primary/secondary/tertiary), demographic restrictions.
2. INDEX TEST (I): The test being evaluated. Specify: test name and protocol, acceptable thresholds, blinding requirements.
3. REFERENCE STANDARD (C): The gold standard. Specify: which reference standards are acceptable; whether composite reference standards are permitted.
4. OUTCOMES (O): Sensitivity and specificity are required (or TP/FP/TN/FN data to calculate them). LR+, LR−, DOR, AUC if reported.
5. STUDY DESIGN (S): Cross-sectional or cohort studies applying both index test AND reference standard to the same participants. State explicitly:
   - Case-control designs: EXCLUDE — spectrum bias inflates diagnostic accuracy
   - Partial verification (reference standard only in index-test-positive cases): EXCLUDE — verification bias
   - Differential verification (different reference standards by test result): EXCLUDE unless separately reported

Key exclusions:
- Studies without a reference standard applied
- Retrospective reviews where reference standard incorporated index test result (incorporation bias)
- Narrative reviews, opinion pieces, editorials, animal studies

**SELF-CHECK (required before outputting criteria):**
- [ ] All 5 PICOS components explicitly defined — no vague or undefined component
- [ ] Case-control exclusion present with "spectrum bias" cited
- [ ] Partial verification exclusion present with "verification bias" cited
- [ ] Reference standard criterion specifies which gold standard(s) are acceptable
- [ ] Outcomes criterion confirms 2×2 table data must be derivable
- [ ] Study design row distinguishes cross-sectional/cohort (INCLUDE) from case-control (EXCLUDE)

OUTPUT: PICOS-structured eligibility criteria table.
```

## Step 2: Dispatch Librarian Agent

**Librarian agent prompt:**

```
[Insert Librarian system prompt from agent-roles.md]

TASK: Validate that these PICOS criteria are searchable for a DTA review.

CRITERIA:
[Insert Methodologist's output]

For each PICOS component confirm:
1. Population → MeSH terms for the condition and suspected-diagnosis context?
2. Index Test → controlled vocabulary + free-text synonyms (include brand names, test variants)?
3. Reference Standard → searchable as a keyword filter?
4. DTA methodological filter available? PubMed: "sensitivity and specificity"[MeSH] OR "ROC curve"[MeSH]; Ovid: sensitivity.mp. AND specificity.mp.
5. Criteria assessable only at full text? (blinding status, verification completeness)

Use WebSearch to verify MeSH terms. Do NOT include RCT or intervention-study filters.

SELF-CHECK:
- [ ] DTA methodological filter block confirmed for each database in scope
- [ ] MeSH terms for index test and target condition verified via WebSearch
- [ ] Explicitly noted: RCT filters must NOT be applied

OUTPUT: Annotated criteria with searchability verdict per PICOS component.
```

## Step 3: Dispatch PI Agent

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Review PICOS eligibility criteria for clinical relevance.

CRITERIA (with Librarian annotations):
[Insert Librarian's output]

Review:
1. Is the patient spectrum in Population clinically appropriate — are we studying patients where this test would actually be ordered?
2. Is the Reference Standard a true gold standard, or a pragmatic substitute? If pragmatic, flag as imperfect reference standard for the Limitations section.
3. Are the Outcomes (sensitivity, specificity) interpretable for the intended use (screening vs. diagnosis vs. rule-out)?
4. Do the design exclusions (case-control, partial verification) appropriately guard against the main accuracy biases?

OUTPUT: Approved criteria or specific revision requests.
```

## Step 4: Write Output

Write to `03_inclusion_exclusion/criteria.md`:

```markdown
# Eligibility Criteria (Diagnostic Test Accuracy — PICOS Framework)

## Inclusion Criteria

| PICOS Component | Definition | Rationale |
|-----------------|-----------|-----------|
| Population (P) | [spec — suspected condition, spectrum, setting] | [why] |
| Index Test (I) | [spec — test name, protocol, threshold] | [why] |
| Reference Standard (C) | [spec — acceptable gold standard(s)] | [why] |
| Outcomes (O) | Sensitivity + specificity (or TP/FP/TN/FN derivable); LR+, LR−, AUC if reported | Required for bivariate synthesis |
| Study Design (S) | Cross-sectional or cohort applying both index test and reference standard | Ensures paired measurement |
| Publication date | [range] | [why] |
| Language | [spec] | [why] |

## Exclusion Criteria

| Criterion | Rationale |
|-----------|-----------|
| Case-control design | Spectrum bias — artificially inflates diagnostic accuracy |
| Partial verification | Verification bias — reference standard applied only to index-positive cases |
| Differential verification | Systematic bias from inconsistent reference standard application |
| No reference standard applied | Cannot calculate accuracy without a comparator |
| [additional exclusions] | [why] |

## Notes on Searchability
[Librarian annotations — DTA filter blocks per database; full-text-only criteria (blinding, verification status)]
```

## Step 5: Quick Review + Update project.yaml

Dispatch a single Independent Reviewer per Quick Review protocol. Initialize `review_iteration = 0`.

**DTA-specific review criteria:**
- Are all 5 PICOS components (P, I, C, O, S) explicitly defined — no component vague?
- Is case-control excluded with "spectrum bias" cited?
- Is partial verification excluded with "verification bias" cited?
- Is the Reference Standard criterion specific enough to identify an acceptable gold standard?
- Do Outcomes criteria confirm 2×2 table data or sensitivity+specificity are required?

Follow the standard REVISE/REJECT iteration loop per `references/reviewer-protocol.md`.

Set `stages.inclusion_exclusion: completed` in `project.yaml`.
