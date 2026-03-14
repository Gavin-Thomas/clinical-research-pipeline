---
name: research-question
description: |
  Use when the user wants to formulate a research question, refine a study question,
  develop a PICO/PEO question, or needs help defining what to study.
  Trigger on phrases like "research question", "what should we study", "PICO",
  "formulate a question", "define the study question", or "what's a good question".
---

# Research Question Formulation (Stage 2)

Formulate a novel, well-structured research question using AI agent teams.

## Prerequisites

- `01_literature_search/landscape_report.md` must exist
- Read `project.yaml` for `topic` and `project_type`

## Project Type Routing

Read `project.yaml`. Route to the appropriate process based on `project_type`:

- `case_report` → [Process — Case Report](#process--case-report)
- `case_series` → [Process — Case Report](#process--case-report)
- `rapid_review` → [Process — Rapid Review](#process--rapid-review)
- `original_research` → [Process — Original Research](#process--original-research)
- `qualitative_synthesis` → [Process — Qualitative Synthesis](#process--qualitative-synthesis)
- `diagnostic_test_accuracy` → [Process — Diagnostic Test Accuracy](#process--diagnostic-test-accuracy)
- All others (systematic_review, scoping_review, meta_analysis) → [Process — Review Types](#process--review-types)

---

## Process — Review Types

### Step 1: Dispatch PI Agent

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Based on the landscape report below, propose 3-5 candidate research questions.

LANDSCAPE REPORT:
[Read and insert 01_literature_search/landscape_report.md]

PROJECT TYPE: [from project.yaml]

For each candidate question:
1. State the question in PICO format (Population, Intervention/Exposure, Comparison, Outcome) or PEO format (Population, Exposure, Outcome) as appropriate
2. Explain WHY this question addresses an identified gap
3. Assess feasibility: Is there likely enough published literature to answer this?
4. Rate novelty: How likely is it this hasn't been published recently?

OUTPUT: Numbered list of 3-5 candidate questions with the above details.
```

### Step 2: Dispatch Methodologist Agent

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Evaluate the following candidate research questions for methodological soundness.

PROJECT TYPE: [from project.yaml]

CANDIDATE QUESTIONS:
[Insert PI's output]

For each question, assess:
1. CLARITY: Is the question specific enough to be answerable?
2. FEASIBILITY: Can this be studied with the proposed project type ([project_type])?
3. MEASURABILITY: Are the outcomes well-defined and measurable?
4. STUDY DESIGN FIT: Does this question fit a [project_type] methodology?
5. RECOMMENDATION: Rank-order the questions. Recommend the top candidate with justification.

OUTPUT: Structured evaluation of each question + final recommendation.
```

### Step 3: Dispatch PI for Final Selection

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Make the final research question selection.

CANDIDATE QUESTIONS:
[Insert PI's original proposals]

METHODOLOGIST EVALUATION:
[Insert Methodologist's output]

Review the Methodologist's assessment. Select and refine the final research question.
Present it in full PICO/PEO format with:
1. The final question statement
2. PICO/PEO breakdown
3. Justification (why this question, why now)
4. Expected contribution to the field

**SELF-CHECK (required before producing output):**

Before finalizing, verify each item and correct inline if any fail:
1. PICO/PEO completeness: Every component explicitly defined — no vague terms (e.g., "patients" must specify age/condition/setting; "outcomes" must name specific measures with timepoints)
2. Gap alignment: The selected question directly addresses a named gap from the landscape report — cite the gap by section heading or title; do not state "addresses a gap" without the specific citation
3. Novelty check: Perform a web search for this exact question published as a systematic review or meta-analysis in the last 3 years. If a near-identical review exists and is not a gap you are explicitly extending, revise scope or framing before presenting
4. Feasibility consistency: The feasibility justification is calibrated to the evidence base described in the landscape report — do not claim "abundant evidence" if the landscape report found fewer than 15 relevant studies
5. Project type fit: The PICO/PEO/PCC framework matches the `project_type` in project.yaml (e.g., do not use PEO for an intervention question; do not propose an RCT-focused question for a scoping review)

Correct any failures inline. Do not present the question to the user until all self-check items pass.
```

### Step 3.5: Present to User for Confirmation

Before writing to disk, present the final selected question to the user:

```
📋 Proposed Research Question:

[Final question statement]

PICO/PEO breakdown:
  P: [Population]
  I/E: [Intervention or Exposure]
  C: [Comparator, if applicable]
  O: [Primary outcome(s)]

Justification: [1-2 sentences on gap and significance]

Does this question capture your intent? You can:
  - Confirm ("yes, proceed") to continue
  - Adjust specific PICO components ("change the comparator to...")
  - Select a different candidate question from the list above
```

Wait for user confirmation before proceeding to Step 4. If the user requests adjustments, apply them directly (do not re-run the full agent loop) and re-present. If the user selects a different candidate question, proceed with that question as-is.

### Step 4: Write Output

Write the final research question to `02_research_question/research_question.md`.

Format:

```markdown
# Research Question

## Primary Question
[Single sentence]

## Framework
| Component | Definition |
|-----------|-----------|
| Population (P) | [spec] |
| Intervention/Exposure (I/E) | [spec] |
| Comparator (C) | [spec or N/A] |
| Outcome(s) (O) | [primary, then secondary] |

## Clinical Significance
[2-3 sentences]

## Anticipated Challenges
- [bullet list]

## Expected Contribution
[1-2 sentences on novelty and field contribution]
```

### Step 5: Quick Review

Dispatch a single Independent Reviewer per Quick Review protocol. Initialize `review_iteration = 0`.

**Review criteria for this stage:**
- Is the question clearly stated in proper PICO/PEO/PCC format with each component operationally defined?
- Does it genuinely address a gap identified in the landscape report (cite the specific gap section)?
- Perform a web search: has this exact question been published as a systematic review or meta-analysis in the last 3 years? If yes, flag and recommend adjusting scope or framing.
- Is the question answerable with the chosen project type (`[project_type]`)?
- Are the outcomes measurable and clearly defined?

**Review iteration loop:**

On **APPROVE**: proceed to Step 6.

On **REVISE**:
- Build a REVISION REQUIRED table:
  | Finding | PICO Component Affected | Specific Issue |
  |---------|------------------------|----------------|
- Apply all listed revisions directly to `research_question.md` (do not re-run the full PI/Methodologist agent loop)
- Increment `review_iteration += 1`
- Re-dispatch reviewer with: (1) updated `research_question.md`, (2) list of revisions made addressing each finding, (3) prior reviewer findings in context

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Print escalation banner with all reviewer findings, revision history, and reason for escalation
- Pause pipeline and present to user for resolution before proceeding to Step 6

### Step 6: Update project.yaml

Set `stages.research_question: completed`.
Update `pico` block in `project.yaml` with the finalized PICO/PEO components from `research_question.md`.

---

## Process — Case Report

For case reports and case series, there is no systematic research question. Instead, the PI frames the teaching objectives and clinical significance of the case(s). This section handles both `case_report` and `case_series` — for case series, the PI should frame objectives around the collective pattern across cases.

### Step 1: Dispatch PI Agent

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Based on the landscape review below, frame the teaching objectives for this case report.

LANDSCAPE REPORT:
[Read and insert 01_literature_search/landscape_report.md]

PROJECT TYPE: case_report

Produce:
1. CASE TEACHING POINTS: 3-5 key learning objectives this case will demonstrate
2. CLINICAL SIGNIFICANCE: Why is this case worth publishing? (unusual presentation,
   rare diagnosis, unexpected treatment response, important diagnostic pitfall, etc.)
3. RELATED LITERATURE: What existing case reports or guidelines does this case relate to
   or extend?
4. CARE GUIDELINE NOTE: Confirm that CARE reporting guideline applies and list which
   CARE checklist items will require particular attention for this case.

OUTPUT: Structured teaching objectives document.
```

### Step 2: Write Output

Write to `02_research_question/research_question.md` (use this path for pipeline consistency, but content is teaching objectives).

### Step 3: Quick Review

Dispatch a single Independent Reviewer per Quick Review protocol. Initialize `review_iteration = 0`.

**Review criteria for this stage (case_report):**
- Are ≥3 specific, distinct teaching points stated (not generic observations)?
- Is the clinical significance compelling and grounded in concrete case features (unusual presentation, rare diagnosis, diagnostic pitfall, unexpected treatment response)?
- Does each CARE checklist item flagged for attention have a specific, actionable concern noted — not just a name?
- Are the related literature references real and relevant?

**Review iteration loop:**

On **APPROVE**: proceed to Step 4.

On **REVISE**:
- Apply all listed revisions directly to `research_question.md`
- Increment `review_iteration += 1`
- Re-dispatch reviewer with: (1) updated `research_question.md`, (2) list of revisions made addressing each finding, (3) prior reviewer findings in context

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Print escalation banner with all reviewer findings, revision history, and reason for escalation
- Pause pipeline and present to user for resolution before proceeding to Step 4

### Step 4: Update project.yaml

Set `stages.research_question: completed`.

---

## Process — Rapid Review

Abbreviated process: single PI pass, no Methodologist evaluation, user confirmation retained.

### Step 1: Dispatch PI Agent (Combined)

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Formulate a focused research question for a rapid review on this topic.

LANDSCAPE REPORT:
[Read and insert 01_literature_search/landscape_report.md]

PROJECT TYPE: rapid_review
CONSTRAINT: The question must be narrow enough to be answerable within a rapid review
timeframe — prefer a focused population + single intervention + primary outcome.

Propose ONE primary research question in PICO format. Also note:
1. Why this question is feasible for a rapid review (estimated evidence base size)
2. Any methodological shortcuts that will be documented as limitations
3. The primary outcome only (secondary outcomes are out of scope for rapid review)

OUTPUT: Single candidate question with PICO breakdown and feasibility rationale.
```

### Step 2: Present to User for Confirmation

Same as Step 3.5 in the Review Types process (present the single candidate for confirmation or adjustment).

### Step 3: Write Output

Write to `02_research_question/research_question.md`.

### Step 4: Quick Review

Dispatch a single Independent Reviewer per Quick Review protocol. Initialize `review_iteration = 0`.

**Review criteria for this stage (rapid_review):**
- Is the PICO question narrow enough to be answerable within a rapid review scope (focused population + single intervention + primary outcome only)?
- Are the documented methodological shortcuts explicitly named (not vague — e.g., "single-reviewer screening" rather than "streamlined methods")?
- Is the feasibility rationale grounded in a specific estimated evidence base size from the landscape survey?
- Is the question answerable without the full systematic review apparatus (dual extraction, comprehensive database coverage)?

**Review iteration loop:**

On **APPROVE**: proceed to Step 5.

On **REVISE**:
- Apply revisions directly to `research_question.md`
- Increment `review_iteration += 1`
- Re-dispatch reviewer with updated file and summary of changes made

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger escalation per `references/reviewer-protocol.md` and pause for user input

### Step 5: Update project.yaml

Set `stages.research_question: completed`.

---

## Process — Original Research

For original research, the research question is a hypothesis or primary aim, not a gap in the review literature.

### Step 1: Dispatch PI Agent

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Formulate 2-3 candidate primary research aims for an original research study.

LANDSCAPE REPORT:
[Read and insert 01_literature_search/landscape_report.md]

PROJECT TYPE: original_research

For each candidate aim:
1. State as a primary hypothesis (null and alternative) or primary research question
2. Specify the study design that would best answer it (cross-sectional, cohort,
   case-control, RCT, retrospective chart review, etc.)
3. Identify key independent and dependent variables
4. Assess feasibility: data source (chart review? prospective enrollment?),
   estimated sample availability, ethical considerations (IRB, consent)
5. Novelty: is this approach meaningfully different from prior work?

OUTPUT: 2-3 candidate aims with the above details.
```

### Step 2: Dispatch Methodologist Agent

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Evaluate these candidate research aims for methodological feasibility.

PROJECT TYPE: original_research

CANDIDATE AIMS:
[Insert PI's output]

For each aim:
1. Is the study design appropriate for the question?
2. Is the sample size likely achievable?
3. Are the variables operationalizable (can be measured)?
4. What are the primary confounders that must be controlled?
5. RECOMMENDATION: Rank and select the most feasible and impactful aim.

OUTPUT: Structured evaluation + recommended primary aim.
```

### Step 3: Present to User for Confirmation

Present the recommended primary aim and study design to the user for confirmation before writing output.

### Step 4: Write Output

Write to `02_research_question/research_question.md`.

### Step 5: Quick Review

Dispatch a single Independent Reviewer per Quick Review protocol. Initialize `review_iteration = 0`.

**Review criteria for this stage (original_research):**
- Is the primary hypothesis stated as a testable null/alternative pair, or as a specific primary aim with a defined dependent variable?
- Does the proposed study design logically match the research question (e.g., a causal question cannot be answered by a cross-sectional design without appropriate caveats)?
- Are both independent and dependent variables operationally defined (not just named)?
- Is the feasibility assessment realistic: does the stated data source (chart review, prospective enrollment, existing registry) plausibly yield the required sample size?
- Are IRB/consent considerations noted where required (any human subjects research)?

**Review iteration loop:**

On **APPROVE**: proceed to Step 6.

On **REVISE**:
- Apply revisions directly to `research_question.md`
- Increment `review_iteration += 1`
- Re-dispatch reviewer with updated file, summary of changes made, and prior reviewer findings in context

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Print escalation banner and pause for user resolution before proceeding

### Step 6: Update project.yaml

Set `stages.research_question: completed`.

---

## Process — Qualitative Synthesis

For `qualitative_synthesis`, use the **PICo framework** — not PICO. No intervention, comparator, or quantitative outcome.

Follow the same Step 1–Step 6 flow as Review Types with these modifications:

**PI agent modifications:** Generate 2-3 candidate PICo questions. For each, define:
- P (Population): Age, condition, role, setting — specific enough to bound the search.
- I (phenomenon of Interest): The specific experience, perspective, or process under study. Be precise — not "experiences of diabetes" but "experiences of self-management decision-making in type 2 diabetes."
- Co (Context): Setting, country/region, or healthcare system.
- State the question as: "What are [P]'s [I] in [Co]?"
- Do NOT include Intervention, Comparator, or Outcome fields.
- SELF-CHECK: No PICO language ("intervention", "comparator", "effect size"); phenomenon of Interest is specific; feasibility grounded in landscape report.

**Methodologist evaluation additions:** Assess synthesis approach fit (thematic synthesis for heterogeneous qualitative evidence; meta-ethnography for interpretive/conceptual questions; framework synthesis if theory-driven; meta-aggregation for descriptive action-oriented findings). Recommend the most appropriate approach.

**User confirmation format:** Present PICo breakdown (P / I / Co) and recommended synthesis approach before writing to disk.

**Output format:** `research_question.md` uses PICo table (not PICO table); Framework section labels: Population / phenomenon of Interest / Context.

**Review criteria (qualitative_synthesis):**
- All three PICo components operationally defined?
- No PICO language (no interventions, comparators, quantitative outcomes)?
- Is qualitative synthesis the correct methodology (question concerns experience/perspective — not clinical effect)?
- Is the recommended synthesis approach (thematic / meta-ethnography / framework / meta-aggregation) appropriate?

**project.yaml update:** Set `stages.research_question: completed`. Map pico fields: `intervention_exposure` → phenomenon of Interest, `comparator` → Context, `outcomes` → `"N/A — qualitative synthesis"`.

---

## Process — Diagnostic Test Accuracy

For `diagnostic_test_accuracy`, use the **PICOS framework**: Population (patients suspected of having the condition), Index test, Comparator (reference standard), Outcomes (sensitivity, specificity, AUC), Study design.

Follow the same Step 1–Step 6 flow as Review Types with these modifications:

**PI agent modifications:** Frame one PICOS question. Define each component:
- P: Suspected condition + clinical setting + expected disease spectrum/severity. Spectrum bias is a DTA-critical concern — define the patient spectrum explicitly.
- I (Index test): Name, protocol/modality variant, single threshold or multiple thresholds of interest.
- C (Reference standard): Gold standard name; note if imperfect comparator. Flag if partial verification (not all patients receiving the reference standard) is likely.
- O: Sensitivity and specificity as primary; at minimum one of AUC, LR+, LR−, or DOR as secondary.
- S: Cross-sectional paired designs as reference; note if cohort designs are also eligible.
- Flag: spectrum bias concern (yes/no + rationale); partial verification concern (yes/no).
- SELF-CHECK: All 5 PICOS components defined; index test named specifically (not "imaging"); reference standard justified; accuracy outcomes as proportions not generic "performance"; spectrum bias and partial verification addressed.

**Methodologist evaluation additions:** Assess: (1) reference standard validity / differential verification bias risk; (2) whether population spectrum definition controls spectrum bias; (3) synthesis method fit — bivariate model (Reitsma — default) vs. HSROC (if threshold effect likely); (4) which QUADAS-2 domains are likely High/Unclear across this evidence base.

**User confirmation format:** Present PICOS breakdown + planned synthesis model (bivariate/HSROC with one-line justification) + spectrum bias note before writing to disk.

**Output format:** `research_question.md` uses PICOS table; Framework section labels: Population / Index test / Reference standard / Accuracy outcomes / Study design.

**Review criteria (diagnostic_test_accuracy):**
- All 5 PICOS components operationally defined?
- Reference standard is true gold standard or best available with justification?
- Spectrum bias addressed in the population definition?
- Synthesis model (bivariate vs. HSROC) justified?
- Question avoids treatment-effect language (DTA asks "how accurately does this test identify the condition?" — not "does this test improve outcomes")?

**project.yaml update:** Set `stages.research_question: completed`. Map pico fields: I → index test, C → reference standard, O → accuracy outcomes.
