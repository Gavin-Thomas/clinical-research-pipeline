# Reviewer Protocol Reference

This file defines the two-tier review system used across all pipeline stages. Every SKILL.md file references this protocol to implement its review step. The orchestrating agent (or skill) must follow these procedures exactly.

---

## Tier 1: Quick Review (Stages 1, 2, 3, 4)

Quick Review uses a single Independent Reviewer and is designed for earlier-stage outputs where speed matters and the cost of a missed error is lower (errors can be caught in later stages).

### Procedure

1. **Dispatch a single Independent Reviewer** as a subagent after the stage output has been written to disk.
2. The reviewer reads the stage output in full and produces a structured verdict: **APPROVE**, **REVISE**, or **REJECT** (see Review Dispatch Template below for the exact prompt).
3. **Handle the verdict:**

| Verdict | Action |
|---------|--------|
| **APPROVE** | Proceed to the next stage. No further action required. |
| **REVISE** | Return the reviewer's findings table to the producing agent(s). The producing agent(s) revise the output, addressing each finding. The revised output replaces the original on disk. Then re-dispatch the same reviewer to confirm the revisions. This counts as iteration 1. |
| **REJECT** | Return the reviewer's findings table to the producing agent(s). The producing agent(s) revise the output, addressing every finding. Re-dispatch the reviewer on the revised output. This counts as iteration 1. |

4. **Iteration logic (max 2 cycles):**
   - After each REVISE or REJECT verdict, the producing agent(s) revise and the reviewer re-evaluates. Each revise-then-re-review pass counts as one iteration.
   - If the reviewer returns **APPROVE** at any point, stop and proceed to the next stage.
   - If the reviewer returns **REVISE** on re-evaluation, the producing agent(s) revise again and the reviewer re-evaluates (iteration 2).
   - If the reviewer returns **REJECT** after iteration 2, **escalate to human PI** (see Escalation Protocol below).

5. **Iteration counter tracking:** The skill must maintain a counter variable `review_iteration` starting at 0. Increment by 1 each time the reviewer is re-dispatched after a revision. When `review_iteration >= 2` and the verdict is still REJECT, trigger escalation.

### Quick Review Flowchart

```
Stage output written
       │
       ▼
  Dispatch Reviewer
       │
       ▼
   ┌─────────┐
   │ Verdict? │
   └─────────┘
    │    │    │
 APPROVE │  REJECT
    │  REVISE  │
    │    │     │
    ▼    ▼     ▼
  Done  Producing agents revise
              │
              ▼
        Re-dispatch Reviewer
        (iteration += 1)
              │
              ▼
         ┌─────────┐
         │ Verdict? │
         └─────────┘
          │    │    │
       APPROVE │  REJECT
          │  REVISE  │
          │    │     │
          ▼    │     ▼
        Done   │  iteration < 2?
               │   Yes → Revise again, re-dispatch (iteration += 1)
               │   No  → Escalate to human PI
               ▼
          iteration < 2?
           Yes → Revise again, re-dispatch (iteration += 1)
           No  → Escalate to human PI
```

---

## Tier 2: Full Review (Stages 5, 6, 7)

Full Review uses two independent reviewers dispatched in parallel, with a third tie-breaking reviewer available on disagreement. This is used for later-stage outputs where errors are costlier and harder to reverse.

### Procedure

1. **Dispatch Reviewer A and Reviewer B in parallel** as independent Agent subagents after the stage output has been written to disk. Neither reviewer sees the other's assessment. Use the Review Dispatch Template below, setting `reviewer_id` to `1` and `2` respectively.

2. Each reviewer reads the stage output in full and independently produces a structured verdict: **APPROVE**, **REVISE**, or **REJECT**.

3. **Compare verdicts and resolve:**

| Reviewer A | Reviewer B | Resolution |
|------------|------------|------------|
| APPROVE | APPROVE | **Proceed** to next stage. |
| REVISE | REVISE | **Merge** both findings tables. Producing agent(s) address all findings from both reviewers. Re-dispatch full review cycle on revised output (iteration += 1). |
| REJECT | REJECT | **Merge** both findings tables. Producing agent(s) address all findings. Re-dispatch full review cycle on revised output (iteration += 1). |
| APPROVE | REVISE | **Disagreement** — dispatch Reviewer C (tie-breaker). |
| REVISE | APPROVE | **Disagreement** — dispatch Reviewer C (tie-breaker). |
| APPROVE | REJECT | **Disagreement** — dispatch Reviewer C (tie-breaker). |
| REJECT | APPROVE | **Disagreement** — dispatch Reviewer C (tie-breaker). |
| REVISE | REJECT | **Disagreement** — dispatch Reviewer C (tie-breaker). |
| REJECT | REVISE | **Disagreement** — dispatch Reviewer C (tie-breaker). |

4. **Tie-breaker (Reviewer C):** When Reviewer A and Reviewer B disagree, dispatch Reviewer C with:
   - The original stage output.
   - Reviewer A's full assessment.
   - Reviewer B's full assessment.
   - `reviewer_id` set to `3`.

   Reviewer C renders an independent verdict that also addresses the points of disagreement. The **final verdict** is Reviewer C's verdict.

5. **After final verdict is determined (whether from agreement or tie-breaker):**

| Final Verdict | Action |
|---------------|--------|
| **APPROVE** | Proceed to next stage. |
| **REVISE** | Producing agent(s) revise, addressing all findings from all reviewers involved. Re-dispatch the full review cycle (iteration += 1). |
| **REJECT** | Producing agent(s) revise, addressing all findings from all reviewers involved. Re-dispatch the full review cycle (iteration += 1). |

6. **Iteration logic (max 3 cycles):**
   - After each non-APPROVE verdict, the producing agent(s) revise and the full review cycle (Reviewer A + Reviewer B, with Reviewer C if needed) repeats. Each full cycle counts as one iteration.
   - If the final verdict is **APPROVE** at any point, stop and proceed.
   - If the final verdict is still **REJECT** after iteration 3, **escalate to human PI** (see Escalation Protocol below).

7. **Iteration counter tracking:** The skill must maintain a counter variable `review_iteration` starting at 0. Increment by 1 each time the full review cycle is re-dispatched after a revision. When `review_iteration >= 3` and the final verdict is still REJECT, trigger escalation.

### Full Review Flowchart

```
Stage output written
       │
       ▼
  Dispatch Reviewer A ──────┐
  Dispatch Reviewer B ──┐   │
                        │   │
                        ▼   ▼
                   Both verdicts in
                        │
            ┌───────────┴───────────┐
            │                       │
        Agree?                  Disagree
            │                       │
            ▼                       ▼
       Use agreed              Dispatch Reviewer C
       verdict                 (with both reviews)
            │                       │
            ▼                       ▼
       Final verdict           Final verdict = C's verdict
            │                       │
            └───────────┬───────────┘
                        │
                        ▼
                   ┌─────────┐
                   │ Verdict? │
                   └─────────┘
                    │         │
                 APPROVE    REVISE/REJECT
                    │         │
                    ▼         ▼
                  Done    iteration < 3?
                           Yes → Revise, re-dispatch full cycle
                           No  → Escalate to human PI
```

---

## Escalation Protocol

When review iterations are exhausted without an APPROVE verdict, the pipeline must pause and alert the human Principal Investigator.

### Escalation Steps

1. **Print a warning banner** to the console:
   ```
   ╔══════════════════════════════════════════════════════════════╗
   ║  ⚠ PIPELINE PAUSED — HUMAN PI REVIEW REQUIRED              ║
   ║                                                              ║
   ║  Stage: [stage_number] — [stage_name]                        ║
   ║  Review tier: [Quick Review / Full Review]                   ║
   ║  Iterations completed: [count]                               ║
   ║  Final verdict: REJECT                                       ║
   ╚══════════════════════════════════════════════════════════════╝
   ```

2. **Print each reviewer's concerns.** For Quick Review, print the single reviewer's findings table. For Full Review, print all reviewers' findings tables (Reviewer A, Reviewer B, and Reviewer C if dispatched), clearly labeled.

3. **Print the producing agent(s)' last revision notes** so the PI can see what was attempted.

4. **Pause the pipeline.** The skill must stop execution and wait for human input. Do not proceed to the next stage. Do not retry automatically.

5. **Resume instructions:** When the human PI provides direction (via chat), the skill should follow the PI's instructions — which may include accepting the output as-is, providing specific revision guidance, or terminating the project.

---

## Revision History Archival

Every review iteration must be logged to a persistent `revision_history.md` file in the stage's output directory. This creates an auditable trail of what was flagged, what was changed, and what was accepted.

### File Location

Each stage writes its revision history to its own directory:

| Stage | Path |
|---|---|
| Stage 1 | `01_literature_search/revision_history.md` |
| Stage 2 | `02_research_question/revision_history.md` |
| Stage 3 | `03_inclusion_exclusion/revision_history.md` |
| Stage 4 | `04_database_search/revision_history.md` |
| Stage 5 | `05_screening/revision_history.md` |
| Stage 6 | `06_data_extraction/revision_history.md` |
| Stage 7 | `07_manuscript/revision_history.md` (also see `revision_log.md` for quality metrics) |

### Format

Append to the file after each review cycle (do not overwrite previous entries):

```markdown
# Revision History — Stage [N]: [Stage Name]

---

## Iteration 0 — Initial Review

**Reviewer(s):** [Quick Review: Reviewer 1 / Full Review: Reviewer A, Reviewer B]
**Verdict:** [APPROVE / REVISE / REJECT]
**Date:** [timestamp]

### Findings
| # | Severity | Finding | Section Affected |
|---|---|---|---|
| 1 | Major | [description] | [section] |
| 2 | Minor | [description] | [section] |

### Action Taken
[If APPROVE: "No revisions needed — proceeding to next stage."]
[If REVISE/REJECT: describe each change made by the producing agent(s)]

---

## Iteration 1 — Re-Review After Revision

**Reviewer(s):** [same or updated]
**Verdict:** [APPROVE / REVISE / REJECT]
**Date:** [timestamp]

### Findings
[New findings table, or "All prior findings resolved."]

### Action Taken
[Description of changes, or "Approved — proceeding."]

---

[Continue appending for each iteration...]
```

### Rules

1. **Always append, never overwrite.** Each iteration adds a new `## Iteration N` section.
2. **Record even APPROVE iterations.** An APPROVE with zero findings should still be logged with "No findings — approved."
3. **Include cross-stage consistency check results** (if applicable). If the orchestrator's cross-stage check found inconsistencies after the review approved, log those as a separate subsection:
   ```
   ### Cross-Stage Consistency Check
   - **Result:** FAIL — [description of inconsistency]
   - **Resolution:** [what the agent changed]
   ```
4. **On escalation to human PI**, log the escalation with the PI's resolution:
   ```
   ### Human PI Escalation
   - **Reason:** [review_iteration >= max without APPROVE]
   - **PI Decision:** [accept as-is / specific revision guidance / terminate]
   - **Resolution:** [what was done]
   ```

---

## Review Dispatch Template

Use this exact prompt structure when dispatching an Independent Reviewer subagent. Replace all `[bracketed]` placeholders with actual values.

### Prompt for Reviewer Dispatch

```
You are being dispatched as Independent Reviewer [reviewer_id] for Stage [stage_number]: [stage_name].

## Your System Prompt

[Insert the full Independent Reviewer system prompt from agent-roles.md, starting from "You are an independent quality reviewer..." through the end of the Output format section.]

## Review Context

**Project file:** Read project.yaml from the project root for project-level settings.
**Stage:** [stage_number] — [stage_name]
**Review tier:** [Quick Review / Full Review]
**Reviewer ID:** [reviewer_id] (1, 2, or 3)
**Iteration:** [review_iteration] (0 = first review, 1+ = re-review after revision)

## Output to Review

The stage output to review is located at:
[absolute_path_to_stage_output]

Read this file in its entirety before forming your assessment.

## Revision History (if iteration > 0)

[If this is a re-review (iteration > 0), include:
- The previous review verdict and findings table
- A summary of what the producing agent(s) changed in response
If this is the first review (iteration 0), omit this section.]

## Tie-Breaker Context (Reviewer 3 only)

[If reviewer_id is 3, include:
- Reviewer 1's full assessment (verdict, summary, findings table, standards referenced)
- Reviewer 2's full assessment (verdict, summary, findings table, standards referenced)
- Instruction: "You must render your own independent verdict and specifically address the points of disagreement between Reviewer 1 and Reviewer 2 in your Disposition section."
If reviewer_id is 1 or 2, omit this section.]

## Task

1. Read the stage output file completely.
2. Evaluate it against: (a) the Stage-Specific Checklist for Stage [stage_number] in the **Stage-Specific Review Checklists** section at the bottom of this document, (b) applicable reporting guidelines and methodological standards, (c) internal consistency and logical coherence, (d) completeness relative to what was requested.
3. Produce your review using the exact output format specified in your system prompt.
4. Your verdict must be one of: APPROVE, REVISE, or REJECT. Follow the verdict rules from your system prompt precisely.
```

### Dispatch Mechanics

- **Quick Review:** Dispatch one subagent using the template above with `reviewer_id = 1`.
- **Full Review (Reviewers A and B):** Dispatch two subagents in parallel using the template above with `reviewer_id = 1` and `reviewer_id = 2`. Do not pass either reviewer's output to the other.
- **Full Review (Reviewer C / tie-breaker):** Dispatch one subagent using the template above with `reviewer_id = 3` and the Tie-Breaker Context section populated.

### Collecting the Verdict

After the reviewer subagent completes, parse its output for the verdict line:

```
#### Verdict: [APPROVE / REVISE / REJECT]
```

Extract the verdict string and the full findings table to determine next steps per the procedures above.

---

## Summary Table

| Property | Quick Review (Stages 1-4) | Full Review (Stages 5-7) |
|----------|--------------------------|--------------------------|
| Reviewers dispatched | 1 | 2 in parallel (+1 tie-breaker if needed) |
| Max iterations | 2 | 3 |
| Tie-breaker | N/A | Reviewer C with both reviews as context |
| Escalation trigger | REJECT after 2 iterations | REJECT after 3 iterations |
| Escalation action | Pause pipeline, print concerns, wait for human PI | Pause pipeline, print all reviewers' concerns, wait for human PI |

---

## Stage-Specific Review Checklists

Use the checklist below for the relevant stage. Each item is a binary pass/fail. Flag every failing item in your findings table. A single **REJECT**-threshold finding (marked ✗✗) warrants a REJECT verdict regardless of other items.

---

### Stage 1 — Literature Search (`landscape_report.md`)

**Search Comprehensiveness**
- [ ] At least 3 distinct sources searched (PubMed/MEDLINE, Google Scholar, preprint server — or documented rationale for fewer)
- [ ] At least 5 existing systematic reviews or meta-analyses cited, or a documented explanation if fewer exist
- [ ] Search terms included MeSH-equivalent concepts and synonyms — not just the verbatim topic phrase

**Gap Quality**
- [ ] Every identified gap is specific and falsifiable (not generic "more research is needed")
- [ ] No gap is already addressed by one of the cited existing reviews (cross-check required) ✗✗
- [ ] At least one gap is actionable for the stated `project_type`

**Strategic Analysis**
- [ ] Each gap in the Gap Analysis cites a specific named review and its stated limitation — no unsubstantiated claims ✗✗
- [ ] Gap Analysis distinguishes between: unstudied areas, low-quality-evidence areas, and conflicting-evidence areas
- [ ] Opportunity Ranking feasibility is calibrated to the project type (e.g., a case report cannot address a meta-analysis-scale gap)
- [ ] Saturation Assessment does not classify any ranked research opportunity as well-covered (internal contradiction check) ✗✗

**Citation Integrity**
- [ ] Every bibliography item is referenced in the report body
- [ ] All citations include author, year, title, and journal/source
- [ ] No `[VERIFY]` flags remain unresolved in the final output

---

### Stage 2 — Research Question (`research_question.md`)

**PICO/PEO/PCC Completeness**
- [ ] All framework components are explicitly defined in the Framework table — no vague or undefined terms ✗✗
- [ ] Framework type matches `project_type` in `project.yaml`: PICO for systematic_review/meta_analysis/rapid_review; PCC for scoping_review; PICo for qualitative_synthesis; PICOS for diagnostic_test_accuracy ✗✗
- [ ] No component is left as "N/A" without documented justification

**Novelty**
- [ ] Gap alignment cites a specific named review and section heading — not a generic novelty claim ✗✗
- [ ] Web search for near-identical published reviews was performed (last 3 years); result documented in `research_question.md`
- [ ] If a near-identical review was found, the question has been scoped or framed to avoid duplication

**Feasibility and Clinical Significance**
- [ ] Feasibility justification is grounded in the evidence base size described in the landscape report — not generic
- [ ] Clinical Significance section states a concrete unmet need (epidemiological impact, treatment gap, or knowledge gap)
- [ ] Anticipated Challenges section is present and project-type-specific

**Downstream Compatibility**
- [ ] `pico` block in `project.yaml` has been updated with the finalized components
- [ ] The question is narrow enough to guide concrete inclusion/exclusion criteria (Stage 3)

**Qualitative Synthesis (conditional — if `project_type: qualitative_synthesis`)**
- [ ] PICo table has all three components explicitly defined: Population (P), phenomenon of Interest (I), and Context (Co) ✗✗
- [ ] No PICO-style fields present — specifically: no Intervention, Comparator, or quantitative Outcome in the framework table ✗✗
- [ ] phenomenon of Interest is operationally specific (identifies the exact experience, behaviour, or process — not generic "experiences of treatment") ✗✗
- [ ] Synthesis approach recommended (thematic / meta-ethnography / framework / meta-aggregation) with rationale matching `synthesis_approach` in project.yaml

**Diagnostic Test Accuracy (conditional — if `project_type: diagnostic_test_accuracy`)**
- [ ] PICOS table has all five components defined: Population (suspected disease population with spectrum description), Index Test (named with protocol/threshold), Comparator/Reference Standard (named with validity justification), Outcome (accuracy metrics — sensitivity, specificity, AUC), Study Design (cross-sectional or consecutive cohort) ✗✗
- [ ] Index test is named specifically (not generic "a diagnostic test") with protocol or threshold described ✗✗
- [ ] Reference standard is explicitly named and its validity as accepted gold standard for the condition is justified ✗✗
- [ ] Spectrum bias concern documented: Population component states the expected patient spectrum (consecutive cohort vs. case-control selected) ✗✗
- [ ] No treatment-effect language in any PICOS component (no OR, RR, HR, survival outcomes — accuracy metrics only) ✗✗
- [ ] Partial verification risk addressed: states whether all patients are expected to receive the reference standard, or acknowledges partial-verification as a potential concern

---

### Stage 3 — Inclusion/Exclusion Criteria (`criteria.md`)

**PICO Alignment**
- [ ] Every PICO/PEO/PCC component from `research_question.md` maps to at least one inclusion criterion ✗✗
- [ ] No inclusion criterion is internally contradicted by an exclusion criterion ✗✗
- [ ] Exclusion criteria add information beyond simply negating inclusion criteria

**Operationalizability**
- [ ] Every criterion can be assessed from a title/abstract without requiring full-text access (or explicitly flagged as "full-text only" with rationale)
- [ ] No criterion relies on subjective clinical judgement without an objective proxy measure
- [ ] The Librarian searchability annotation is present for each criterion

**Design and Scope**
- [ ] Study design criteria match `project_type` (e.g., `meta_analysis` requires controlled designs; `scoping_review` should be broad)
- [ ] Date range restriction is present with a documented clinical rationale
- [ ] Language restriction is stated with brief rationale (or absence of restriction is noted)
- [ ] Criteria are not so broad that they risk yielding >500 results, or so narrow they risk zero results

**Conditional**
- [ ] If `review_config.prospero_registration: true`, `prospero_protocol.md` exists and covers all 10 PROSPERO sections ✗✗

**Original Research (conditional — if `project_type: original_research`)**

_Note: For original research, Stage 3 produces `study_protocol.md`, not `criteria.md`. The standard PICO Alignment, Operationalizability, and Design and Scope checks above do NOT apply. Use the checklist below instead._

- [ ] Study design is named (cross-sectional, cohort, case-control, RCT, etc.) with a justification matching the specific research question aim — "it fits the topic" is not sufficient ✗✗
- [ ] Primary outcome is operationalized: names the measurement tool or data source AND the exact time point — not just the outcome name ✗✗
- [ ] Every covariate in the adjustment set has an explicit confounder justification: associated with both the exposure and outcome through a pathway outside the causal mechanism ✗✗
- [ ] Mediators (if present) are identified and excluded from the primary adjustment set — not conflated with confounders ✗✗
- [ ] Sample size justification present — must be a power calculation, feasibility estimate, or precision estimate; completely absent is a REJECT trigger ✗✗
- [ ] Ethics section covers all four items: IRB review type with regulatory basis, privacy/HIPAA approach, consent approach, and registration status ✗✗
- [ ] Eligibility criteria are operational for a chart reviewer or coordinator using standard data access — no criterion requires clinical judgement unavailable from the data source
- [ ] Protocol aligns with the research question and study design formulated in Stage 2

---

### Stage 4 — Database Search Strategy (`search_strategy.md`)

**Coverage**
- [ ] Every database listed in `project.yaml` has a complete, numbered search strategy ✗✗
- [ ] At least 3 concept blocks are present (Population, Intervention/Exposure, Outcome or comparator)
- [ ] Each concept block has ≥3 free-text synonyms including truncated forms (e.g., `cardio*`)

**Technical Correctness**
- [ ] Boolean logic is structurally correct — no orphaned AND/OR operators, no mismatched parentheses ✗✗
- [ ] MeSH/Emtree terms were verified via WebSearch — no unverified controlled vocabulary ✗✗
- [ ] Database-specific syntax is correct: PubMed uses `[MeSH Terms]`/`[All Fields]`; Ovid uses `/`/`.mp.`; Embase uses `/exp`
- [ ] Filters (date range, language, publication type) applied as a final AND step — not embedded in concept blocks

**Reproducibility**
- [ ] Each strategy is copy-paste reproducible with no ambiguous shorthand ✗✗
- [ ] If a landscape search strategy from Stage 1 was available, key terms from it are incorporated

---

**Original Research — Data Collection Instruments (conditional — if `project_type: original_research`)**

_For original_research, Stage 4 produces `04_database_search/data_collection_instruments.md` instead of a search strategy. The checklist items above (Coverage, Technical Correctness, Reproducibility) do NOT apply. Use only the items below._

- [ ] Data collection form includes every variable (primary outcome, secondary outcomes, key confounders) explicitly named in `study_protocol.md` — no protocol variable absent without documented rationale ✗✗
- [ ] Every categorical variable has a codebook entry with exhaustive valid codes (not "other" as the only non-primary option) ✗✗
- [ ] Every continuous variable has a plausible numerical range or units specified in the codebook
- [ ] At least one data quality check (range check or logic check) is defined for each required or conditional field ✗✗
- [ ] `participant_id` field is present and specified as anonymized (no PHI) ✗✗
- [ ] The collection form is operationalizable: every field can be populated by a research coordinator or chart reviewer using the codebook alone — no field requires clinical judgment not explained in the instruments

---

### Stage 5 — Abstract Screening (`screening_results.csv`)

**Completeness**
- [ ] Every abstract from the input file has exactly one decision row — no missing or duplicated abstract IDs ✗✗
- [ ] Three screener columns present (Screener 1, Screener 2, Screener 3) plus a Reconciled Decision column
- [ ] Agreement rate is reported in the output or summary note

**Decision Quality**
- [ ] Every EXCLUDE decision cites a specific criterion from `criteria.md` by name — not "not relevant" or "out of scope" ✗✗
- [ ] UNCERTAIN is used only for genuine information absence, not borderline hedging
- [ ] Borderline abstracts are defaulted to INCLUDE (full-text is the safety net) — check for excessive EXCLUDE rates (>60% EXCLUDE is a flag)

**Conflict Resolution**
- [ ] All 3-way disagreements are flagged as CONFLICT with a substantive reconciliation_notes entry ✗✗
- [ ] No CONFLICT row is left unresolved

**Deduplication**
- [ ] `05_screening/deduplication_report.md` exists ✗✗
- [ ] Report states the pre-deduplication record total (sum across all database exports) and post-deduplication count proceeding to screening ✗✗
- [ ] Duplicate count is ≥ 0 (no negative or implausible figures); post-deduplication count ≤ pre-deduplication total
- [ ] The record count in the report matches the actual row count screened in `screened_results.csv`
- [ ] If any `POSSIBLE_DUPLICATE` records are present in `screened_results.csv`, each has a note in the `reasoning` field explaining the fuzzy-match basis

**Original Research — Data Cleaning (conditional — if `project_type: original_research`)**

_Note: For original research, Stage 5 is the **data cleaning and eligibility filtering** stage. It produces `05_screening/cleaned_data.csv`, `05_screening/excluded_participants.csv`, `05_screening/cleaning_log.md`, and `05_screening/data_profile.md`. The standard Completeness, Decision Quality, Conflict Resolution, and Deduplication checks above do NOT apply. Use only the items below._

- [ ] Row count reconciliation: `cleaned_data.csv` row count + `excluded_participants.csv` row count equals the raw data row count minus exact duplicates removed — silent participant loss is a critical data integrity error ✗✗
- [ ] Every row in `excluded_participants.csv` cites a specific, named criterion from `study_protocol.md` — generic reasons ("does not meet criteria", "not eligible") are not acceptable ✗✗
- [ ] No imputation in `cleaned_data.csv`: all missing values remain null/blank/NaN — any filled value where the protocol mandates missing is a REJECT trigger (imputation is the Stage 6 Statistician's decision) ✗✗
- [ ] Data quality flags are present for out-of-range numeric values: every variable with a documented plausible range in `study_protocol.md` has corresponding OUT_OF_RANGE flags applied in `cleaned_data.csv`, not silently removed or corrected ✗✗
- [ ] HIGH_MISSINGNESS flag applied to every variable with >20% missingness (as documented in `data_profile.md`)
- [ ] POSSIBLE_DUPLICATE rows are retained in `cleaned_data.csv` with the POSSIBLE_DUPLICATE flag column populated — they must NOT be removed at this stage ✗✗
- [ ] All column names in `cleaned_data.csv` correspond to variable names in `study_protocol.md` OR are documented in `cleaning_log.md` as renamed (original name → new name, with rationale) ✗✗
- [ ] `cleaning_log.md` documents every structural change: each exclusion (row ID + criterion), duplicate removal (row IDs), rename, and recode — no undocumented change
- [ ] `data_profile.md` exists and explicitly lists any variables named in `study_protocol.md` that are absent from the raw data file — missing variables must not be silently omitted

---

**Qualitative Synthesis (conditional — if `project_type: qualitative_synthesis`)**
- [ ] Every screened record has a non-blank `qualitative_method` column entry ✗✗
- [ ] All INCLUDE decisions are for qualitative or mixed-methods studies only — no pure quantitative studies among included records ✗✗
- [ ] Mixed-methods studies are coded INCLUDE (not EXCLUDE) when ≥1 qualitative component is present ✗✗
- [ ] EXCLUDE decisions for quantitative designs cite the qualitative-design criterion specifically, not generic phrases ("not relevant", "out of scope")
- [ ] PICo criterion alignment is documented in the reconciliation notes for each INCLUDE decision

**Diagnostic Test Accuracy (conditional — if `project_type: diagnostic_test_accuracy`)**
- [ ] Every INCLUDE and UNCERTAIN row has a non-blank `threshold_reported` field ✗✗ (value or "NR" — never blank)
- [ ] No case-control design study appears as INCLUDE without a documented exception — case-control is an exclusion criterion for spectrum bias in DTA reviews ✗✗
- [ ] Partial verification studies (reference standard applied only to index-test-positive cases) are EXCLUDE if stated as an exclusion criterion in criteria.md — check reconciliation_notes for any partial-verification CONFLICT ✗✗
- [ ] UNCERTAIN decisions cite one of the three valid DTA UNCERTAIN triggers: (a) unclear whether both index test and reference standard were applied, (b) patient spectrum ambiguity (case-control vs. consecutive cohort not determinable), or (c) threshold absent but plausibly present in full text — UNCERTAIN is not used for borderline population or outcome fit
- [ ] Reconciliation notes for design-related CONFLICTs explicitly address spectrum bias (case-control enrollment) and partial verification, not just generic "design disagreement"
- [ ] No INCLUDE decision for a study where the abstract makes clear that a reference standard was not applied (single-test studies, index-test-only validation studies)

---

### Stage 6 — Data Extraction (`extracted_data.csv` + `synthesis_report.md`)

**Extraction Completeness**
- [ ] Every study in the included-studies list has exactly one row in `extracted_data.csv` ✗✗
- [ ] "NR" used only for data genuinely absent from the source paper — not as a shortcut
- [ ] Effect sizes, CIs, and p-values transcribed character-for-character with page/table references in `source_notes`
- [ ] If `clinical_coding: true`, all coding columns (ICD-10, ICD-11, CPT, SNOMED) are filled or marked `[UNSPECIFIED]` ✗✗

**Synthesis Quality**
- [ ] Study characteristics table in `synthesis_report.md` covers all studies in `extracted_data.csv` ✗✗
- [ ] Narrative synthesis addresses every outcome specified in `research_question.md`
- [ ] Risk-of-bias summary references actual quality scores — no generic "quality was variable" statements ✗✗
- [ ] Discrepant findings across studies are documented in a "Discrepant Findings" subsection
- [ ] No fabricated statistics — every value must be traceable to an extraction column

**Meta-analysis (conditional — if `meta_analysis: true`)**
- [ ] Forest plot and funnel plot files exist in `analysis_scripts/outputs/`
- [ ] Heterogeneity reported: I², Q-statistic, tau² ✗✗
- [ ] Subgroup analyses pre-specified in `project.yaml` are present ✗✗
- [ ] GRADE certainty assessment table is present ✗✗
- [ ] Analysis script exists and is executable (correct library imports, no placeholder variables)

**Original Research (conditional — if `project_type: original_research`)**

_Note: For original research, Stage 6 produces `synthesis_report.md` (analysis report) and analysis scripts — not `extracted_data.csv`. The standard Extraction Completeness and Synthesis Quality checks above do NOT apply. Use the checklist below instead._

- [ ] `synthesis_report.md` covers ALL pre-specified analyses from `study_protocol.md`: primary analysis, every secondary outcome, all pre-specified subgroup analyses, all pre-specified sensitivity analyses — none silently omitted ✗✗
- [ ] Table 1 (participant characteristics) is present and correctly formatted: continuous variables as mean ± SD or median (IQR); categorical as n (%); columns by study group (if applicable) plus total ✗✗
- [ ] For RCT: Table 1 does NOT include p-values for baseline comparisons — testing pre-randomization differences is methodologically incorrect (CONSORT 2010 requirement) ✗✗
- [ ] Primary outcome reported with BOTH unadjusted and adjusted estimates, each with 95% CI and exact p-value (not "p < 0.05" or "p = NS") ✗✗
- [ ] For RCT binary outcomes: absolute risk reduction AND NNT or NNH reported alongside the relative effect measure (RR or OR) ✗✗
- [ ] For observational designs: all confounders named in `study_protocol.md` are included in the primary adjusted model; no confounder is silently dropped ✗✗
- [ ] Missing data: pattern described for the primary outcome and all key predictors; if >5% missing for primary outcome or key predictor, sensitivity analysis with multiple imputation is present alongside complete-case analysis ✗✗
- [ ] Subgroup analyses (if pre-specified in `project.yaml`): all present; reported with interaction p-value per subgroup variable — NOT individual within-subgroup p-values ✗✗
- [ ] Sensitivity analyses (if pre-specified in `project.yaml`): all present; results compared to primary analysis; substantial differences (point estimate shifts >20% or CI crosses null) explicitly flagged
- [ ] Analysis script (`analysis_scripts/primary_analysis.py` or `.R`) exists, has all required imports, no placeholder variables, reads from `cleaned_data.csv`, saves output figures to `analysis_scripts/outputs/` ✗✗
- [ ] No meta-analytic or review language in the report: no "included studies", "pooled estimate", "I²", "PRISMA" — this is primary research analysis, not a systematic review ✗✗

**Diagnostic Test Accuracy (conditional — if `project_type: diagnostic_test_accuracy`)**
- [ ] QUADAS-2 assessment is complete for every included study: all 4 bias domains (Patient Selection, Index Test, Reference Standard, Flow & Timing) rated Low/High/Unclear, plus all 3 applicability concern domains ✗✗
- [ ] Bivariate model data captured: sensitivity, specificity, and their variances are extractable from the extracted rows (or raw TP/FP/FN/TN counts present) ✗✗
- [ ] SROC curve output file exists in `analysis_scripts/outputs/` ✗✗
- [ ] Sensitivity/specificity forest plots (separate plots) exist in `analysis_scripts/outputs/` ✗✗
- [ ] Likelihood ratios (LR+, LR−) and diagnostic odds ratio (DOR) with 95% CI are reported in `synthesis_report.md`
- [ ] Threshold effect assessed (Spearman correlation of logit sensitivity and logit (1–specificity)) and result documented
- [ ] Publication bias assessed using Deeks' funnel plot asymmetry test (requires ≥10 studies; otherwise documented as infeasible)
- [ ] GRADE-DTA certainty assessment is present for both sensitivity and specificity estimates ✗✗

**Qualitative Synthesis (conditional — if `project_type: qualitative_synthesis`)**
- [ ] Every included qualitative study has an entry in `extracted_data.csv` with: design, setting, participants, data collection method, analytic approach, key themes/findings ✗✗
- [ ] CASP Qualitative Checklist scores are recorded for every study ✗✗
- [ ] `synthesis_report.md` documents the synthesis method (thematic synthesis / meta-ethnography / framework synthesis / meta-aggregation) and the analytical process step by step
- [ ] Descriptive themes and analytical themes (or second-order and third-order concepts for meta-ethnography) are clearly distinguished ✗✗
- [ ] CERQual confidence assessments are present for each synthesised finding: methodological limitations, coherence, adequacy, relevance ✗✗
- [ ] A Summary of Qualitative Findings (SoQF) table is present in `synthesis_report.md` — one row per synthesised finding with CERQual rating and explanation ✗✗
- [ ] Reflexivity statement regarding researcher positionality documented (or flagged as AI-agent limitation)

**Network Meta-Analysis (conditional — if `review_config.network_meta_analysis` is `true`)**
- [ ] `nma_results.md` exists and contains NMA output ✗✗
- [ ] Network geometry documented: all treatment nodes and direct comparison edges enumerated; number of studies per edge stated; sparse edges (single-study comparisons) explicitly flagged ✗✗
- [ ] Consistency assessment performed: global design-by-treatment interaction test (χ² and p-value) reported; local node-splitting for ≥3 clinically important pairwise comparisons documented; if any p < 0.1, source of inconsistency investigated ✗✗
- [ ] League table present: all pairwise NMA estimates (point estimate, 95% CI or CrI); cells annotated to distinguish direct-evidence from indirect-only estimates ✗✗
- [ ] Treatment ranking: SUCRA (frequentist) or P-score (Bayesian) reported for every treatment; mean rank and rank probability histogram described ✗✗
- [ ] Heterogeneity: between-study variance (tau²) and I² for the overall network model reported
- [ ] GRADE-NMA: 7-domain table present — 5 standard GRADE domains plus (a) Indirectness: indirect evidence contribution quantified per comparison; downgraded if predominantly indirect; (b) Incoherence: downgraded if global or local inconsistency detected ✗✗
- [ ] Sensitivity analysis excluding high-RoB studies performed; SUCRA/P-score rankings compared to main analysis; treatment hierarchy stability stated ✗✗
- [ ] All 4 required output files exist in `analysis_scripts/outputs/`: `network_geometry.png`, `nma_forest.png`, `sucra_ranking.png`, `nma_league_table.csv` ✗✗
- [ ] Analysis script (`analysis_scripts/nma_analysis.R` or `.py`) exists with correct library imports and no placeholder variables

---

### Stage 7 — Manuscript (`manuscript.md`)

**Journal Compliance**
- [ ] Word count is within the target journal's stated limit (check `review_config.target_journal`)
- [ ] Abstract format (structured/unstructured) matches journal requirements
- [ ] All mandatory sections required by the journal are present
- [ ] Author contribution statement and acknowledgements sections are present

**Reporting Guideline Compliance**
- [ ] Appropriate reporting guideline checklist completed: PRISMA 2020 (systematic_review/meta_analysis), PRISMA-ScR (scoping_review), STROBE (original_research cohort/cross-sectional), CONSORT (RCT), CARE (case_report), ENTREQ (qualitative_synthesis), STARD 2015 (diagnostic_test_accuracy) ✗✗
- [ ] PRISMA flow diagram is referenced and consistent with screening numbers ✗✗
- [ ] Study registration number (PROSPERO, ClinicalTrials.gov) cited in Methods if applicable

**Writing & Internal Consistency**
- [ ] Verb tense is consistent (past tense for methods/results, present for interpretation)
- [ ] Abbreviations defined on first use and used consistently thereafter
- [ ] Numbers follow APA convention (spell out <10 unless attached to unit; use numerals ≥10)
- [ ] No unresolved `[CITATION NEEDED]`, `[VERIFY]`, or `[INSERT]` placeholder flags ✗✗

**Statistical Reporting**
- [ ] All effect estimates reported with 95% CI and exact p-value ✗✗
- [ ] Heterogeneity metrics (I², Q, tau²) reported for all pooled analyses
- [ ] GRADE certainty rating for each outcome is stated in text and/or summary-of-findings table
- [ ] No data in Results that was not pre-specified in Methods (post-hoc analyses flagged as such)

**Tables & Figures**
- [ ] All tables and figures numbered sequentially (Table 1, Table 2…; Figure 1, Figure 2…) ✗✗
- [ ] Every table/figure has a self-contained legend (reader should not need to read text to interpret)
- [ ] Every table/figure referenced in the text is included; no unreferenced tables/figures

**Original Research Manuscript (conditional — if `project_type: original_research`)**
- [ ] No PRISMA flow diagram present — CONSORT (RCT) or STROBE (observational) participant flow used instead ✗✗ (PRISMA in an original research manuscript indicates the agent used the wrong template)
- [ ] Reporting guideline applied: CONSORT 2010 (if RCT) or STROBE 2007 (if observational) — checklist items addressed in Methods ✗✗
- [ ] Study registration number (ClinicalTrials.gov or equivalent) cited in Methods, or absence documented with justification
- [ ] IRB/ethics approval number and institution cited in Methods ✗✗
- [ ] Table 1 (baseline participant characteristics) present; continuous variables reported as mean ± SD or median (IQR); categorical as n (%) ✗✗
- [ ] For RCT: Table 1 does NOT include p-values for baseline comparisons (pre-randomization differences are expected by chance — testing them is methodologically incorrect)
- [ ] Primary outcome reported with both unadjusted and adjusted effect estimates (95% CI and exact p-value for each) ✗✗
- [ ] For RCT: absolute risk difference and NNT or NNH reported alongside relative effect measure (RR or OR)
- [ ] Subgroup analyses (if pre-specified) report interaction p-values — not separate within-subgroup p-values ✗✗
- [ ] Missing data pattern described; imputation method stated if used; complete-case sensitivity analysis present if imputation was performed
- [ ] Discussion correctly uses associative language for observational studies ("associated with", "linked to") — no causal claims without RCT design ✗✗
- [ ] Results section contains no review-type language ("included studies", "search strategy", "abstract screening", "PRISMA") ✗✗

**Diagnostic Test Accuracy Manuscript (conditional — if `project_type: diagnostic_test_accuracy`)**
- [ ] STARD 2015 checklist is included as a supplement with all 30 items addressed ✗✗
- [ ] QUADAS-2 traffic light plot is referenced and file exists ✗✗
- [ ] SROC curve figure is present and referenced in Results ✗✗
- [ ] Sensitivity and specificity forest plots (separate) are present and referenced ✗✗
- [ ] Bivariate model summary point (sensitivity + specificity with 95% confidence region) reported in Results ✗✗
- [ ] Threshold effect analysis result reported in Methods/Results (Spearman ρ and p-value)
- [ ] LR+, LR−, and DOR with 95% CI reported; clinical utility framing present in Discussion
- [ ] Fagan nomogram referenced (if pre- and post-test probability calculation is included)
- [ ] GRADE-DTA certainty ratings for sensitivity and specificity stated in Results or SoF table ✗✗

**Qualitative Synthesis Manuscript (conditional — if `project_type: qualitative_synthesis`)**
- [ ] ENTREQ checklist is included as a supplement with all 21 items addressed ✗✗
- [ ] PICo framework (Population, phenomenon of Interest, Context) stated in Introduction/Methods ✗✗
- [ ] Synthesis approach (thematic synthesis, meta-ethnography, framework synthesis, or meta-aggregation) and its steps described in Methods ✗✗
- [ ] Findings section presents synthesised themes/categories with supporting quotations or translations — not merely a summary of individual studies ✗✗
- [ ] CERQual confidence rating table (Summary of Qualitative Findings) present as Table or Supplement ✗✗
- [ ] Reflexivity statement present in Methods (researcher backgrounds, how interpretive assumptions were managed) ✗✗
- [ ] Transferability statement in Discussion (settings to which findings may or may not apply)

**Network Meta-Analysis Manuscript (conditional — if `review_config.network_meta_analysis` is `true`)**
- [ ] Network geometry figure present, sequentially numbered, and referenced by number in Results ✗✗
- [ ] League table (all pairwise NMA estimates with 95% CI/CrI, direct vs. indirect evidence annotated) present as a Table or supplementary file ✗✗
- [ ] SUCRA or P-score ranking plot present and referenced in Results ✗✗
- [ ] Statistical Methods section describes NMA model (frequentist graph-theoretical `netmeta` or Bayesian `gemtc`/`BUGSnet`); if Bayesian: convergence criteria reported (R-hat < 1.01, ESS > 1,000)
- [ ] Consistency test results reported in Methods/Results: global design-by-treatment interaction test (χ², df, p) and node-splitting results for key comparisons ✗✗
- [ ] GRADE-NMA certainty table present with all 7 domains (including Indirectness and Incoherence); downgrading rationale stated per domain ✗✗
- [ ] Sensitivity analysis (excluding high-RoB studies) described in Methods and results reported; treatment hierarchy stability discussed in Discussion
- [ ] Discussion interprets treatment rankings with appropriate uncertainty framing (indirect evidence limitations, sparse network edges)
