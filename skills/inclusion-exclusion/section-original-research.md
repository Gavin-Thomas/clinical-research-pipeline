# Inclusion/Exclusion — Original Research Study Protocol

_Read and follow this file when `project_type: original_research`. For original research, Stage 3 produces a study protocol rather than eligibility criteria for a literature review. The protocol defines the study design, participant eligibility, variables, sample size, data collection, and ethical framework._

---

## Step 1: Dispatch Methodologist Agent

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Design a detailed study protocol for the following original research question.

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

LANDSCAPE REPORT (for feasibility context):
[Read and insert 01_literature_search/landscape_report.md]

PROJECT SETTINGS:
[Read project.yaml — relevant fields: topic, target_journal, review_config if present]

---

**PART 1: STUDY DESIGN SELECTION**

Choose the most appropriate study design and justify it. Select from the list below and state why it fits the research question aim (prevalence, incidence, association, causation, prediction, descriptive):

- **Retrospective chart review / cross-sectional study**: existing medical records or registries answer the question; no prospective data collection; typically fastest to IRB approval; cannot establish temporality.
- **Prospective cohort**: temporal relationships matter (exposure → outcome); participants followed forward; longer timeline and active data collection required.
- **Case-control**: studying rare outcomes (cases enrolled because they have the outcome); efficient for rare diseases; susceptible to recall bias and selection bias.
- **Cross-sectional survey**: prevalence questions or associations at a single time point; cannot establish temporality; fast and low-cost.
- **Randomized controlled trial (RCT)**: testing an intervention with random allocation; highest internal validity for intervention questions; requires ethics approval, consent, and randomization infrastructure.
- **Prospective registry-based study**: tracking outcomes for a defined cohort with standardized data collection over time.

State:
1. Selected design and its formal name
2. Why this design fits the specific research aim (not just the topic)
3. At least one key limitation introduced by this design (e.g., confounding in observational studies, selection bias in case-control, loss to follow-up in cohorts)

**PART 2: PARTICIPANT ELIGIBILITY**

Define eligibility in two structured tables:

Inclusion criteria — specify:
- Primary condition / diagnosis (with operational definition: ICD-10 code range, validated diagnostic criteria, or clinical threshold)
- Age range (with rationale if restricted)
- Clinical setting (inpatient / outpatient / both; institution type)
- Time period for data collection (start and end dates; rationale for period choice)
- Any required baseline characteristics (minimum disease severity, confirmed diagnosis method, etc.)

Exclusion criteria — specify:
- Conditions or characteristics that would confound the primary outcome (name the specific confounding pathway)
- Insufficient data: which minimum data fields must be present for inclusion
- Prior participation in related studies (if relevant)
- Any contraindications (for intervention studies)

**PART 3: VARIABLES AND OUTCOMES**

Define each item below explicitly — vague descriptions ("we will measure X") are not acceptable:

Primary outcome:
- Variable name
- Operationalization: exact measurement tool, data source (ICD code / lab assay / validated questionnaire name and subscale), and unit of measurement
- Time point: when the outcome is measured relative to enrollment or exposure

Secondary outcomes (table format):
| Outcome | Operationalization | Time Point |

Independent variable(s) / exposure:
- Exact definition
- How it is categorized (continuous? binary? ordinal cut-points?)
- Reference category

Covariates and confounders:
- List each variable that will be adjusted for in the primary analysis
- For each, state why it is a confounder: it is associated with BOTH the exposure AND the outcome through a pathway outside the causal mechanism of interest
- Do NOT include a variable simply because it was available in the data — every covariate requires this justification

Mediators (if any):
- Distinguish from confounders — do NOT adjust for mediators in the primary analysis (adjusting for mediators blocks the causal pathway of interest)

**PART 4: SAMPLE SIZE CONSIDERATIONS**

Provide ONE of:
- Formal power calculation (preferred): state α level, desired power (1−β), expected effect size and its source (cite a pilot study, prior literature, or clinical minimally important difference), allocation ratio, and resulting required N per group.
- Feasibility-based justification: state the likely available dataset size and compare to published minimum sample requirements for the statistical method (e.g., "EPV ≥ 10 for logistic regression requires N ≥ [X] given [Y] predictors").
- Precision-based estimate (prevalence studies): state target margin of error (±X%) around expected prevalence of Y%, yielding N = Z.

State the final expected total N (and per-group N if applicable).

**PART 5: DATA COLLECTION AND SOURCES**

State:
- Data source(s): EHR system (name), REDCap database, paper forms, national registry, claims data — be specific about the actual system
- Collection procedure: retrospective automated pull vs. manual chart review vs. prospective entry — who performs collection
- Standardization: ICD-10 coding, lab value normalization, questionnaire scoring procedures
- Quality checks planned during collection: range validation, required-field enforcement, duplicate participant_id detection

**PART 6: ETHICS AND REGULATORY CONSIDERATIONS**

State all four items:
1. IRB review type: full review / expedited / exempt — with the specific regulatory basis (e.g., 45 CFR 46.104(d)(4) for retrospective de-identified data; prospective with consent requires full review)
2. Privacy/HIPAA: de-identified dataset (Safe Harbor or Expert Determination) / Limited Data Set (with DUA) / full PHI (requires HIPAA Authorization or Waiver)
3. Consent: waiver of informed consent (retrospective studies often qualify) / written consent / assent with parental consent — with justification
4. Registration: ClinicalTrials.gov registration required for RCTs (mandatory for journal publication) and recommended for prospective observational studies; state whether registration has occurred and the NCT number if applicable

---

**SELF-CHECK (required before outputting the protocol):**

Verify each item below. Correct failures inline before producing output — do not pass a failing protocol to the PI reviewer.

- [ ] **Study design is specifically named and justified** — states the design name, why it matches the research question aim, and names at least one key design limitation
- [ ] **Primary outcome is operationalized** — names the measurement tool/data source AND the exact time point; "we will measure X" without these specifics is a FAIL
- [ ] **Every covariate is justified as a confounder** — explicitly associated with both the exposure and outcome; no variable listed purely because it was measured
- [ ] **Mediators distinguished from confounders** — if any mediators are present, they are not in the adjustment set
- [ ] **Sample size justification is present** — must be a power calculation, feasibility estimate, or precision estimate; absent justification is a FAIL
- [ ] **Ethics covers all four items** — IRB review type, privacy/HIPAA approach, consent approach, and registration status; "IRB approval will be obtained" alone is insufficient
- [ ] **Eligibility criteria are operational** — a chart reviewer or coordinator can apply them using standard data access; no criterion requires clinical judgement unavailable from the data source

OUTPUT: Structured study protocol in markdown with all 6 parts above.
```

---

## Step 2: Dispatch PI Agent for Review

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Review this study protocol for clinical feasibility and scientific rigor.

STUDY PROTOCOL:
[Insert Methodologist's output]

Assess:
1. **Clinical feasibility**: Is this study performable at a clinical centre with standard resources? Are the proposed data sources actually available? Is the patient population realistic for the setting described?
2. **Primary outcome relevance**: Is the primary outcome clinically meaningful — would a practicing clinician or patient care about this result?
3. **Confounder completeness**: Are there important clinical confounders that the Methodologist omitted? Draw on domain knowledge — list any confounders missing from the covariate table and explain the confounding pathway.
4. **Study design fit**: Is the proposed design the right choice, or would another design more directly answer the question with less bias?
5. **Eligibility edge cases**: Are there common patient presentations that the eligibility criteria would ambiguously include or exclude? Name specific examples if so.

OUTPUT: Approval with specific comments integrated into the protocol, or revision requests listing specific changes needed by section.
```

---

## Step 3: Write Output

After PI approval (or after PI requests addressed by Methodologist), write to `03_inclusion_exclusion/study_protocol.md`:

```markdown
# Study Protocol

**Project:** [from project.yaml topic]
**Date:** [today's date]
**Version:** 1.0

## 1. Study Design

[Selected design name + justification + key limitations]

## 2. Participant Eligibility

### Inclusion Criteria

| Criterion | Definition | Data Source |
|-----------|-----------|-------------|
| Primary condition | [ICD-10 range / diagnostic criteria] | [EHR / registry] |
| Age | [range with rationale if restricted] | [EHR] |
| Clinical setting | [inpatient / outpatient / both] | [EHR] |
| Time period | [start–end dates; rationale] | [EHR / registry] |
| [additional criteria] | [operationalizable definition] | [source] |

### Exclusion Criteria

| Criterion | Definition | Rationale |
|-----------|-----------|-----------|
| [criterion] | [operationalizable definition] | [confounding pathway or data quality reason] |

## 3. Variables and Outcomes

### Primary Outcome

- **Variable:** [name]
- **Operationalization:** [measurement tool / ICD code / assay / questionnaire + subscale]
- **Time point:** [when measured relative to enrollment/exposure]

### Secondary Outcomes

| Outcome | Operationalization | Time Point |
|---------|-------------------|-----------|
| [name] | [tool/source] | [when] |

### Independent Variable / Exposure

| Variable | Definition | Categories | Reference Category |
|----------|-----------|-----------|-------------------|
| [name] | [definition] | [value list] | [reference] |

### Covariates and Confounders

| Variable | Confounder Justification |
|----------|--------------------------|
| [name] | Associated with [exposure] via [pathway] AND with [outcome] via [pathway] |

### Mediators (if applicable)

| Variable | Mediating Pathway | Analysis Note |
|----------|------------------|---------------|
| [name] | [exposure → mediator → outcome] | Not adjusted for in primary analysis |

## 4. Sample Size

[Power calculation / feasibility estimate / precision estimate — must state final N]

**Final expected sample size:** N = [total] ([n per group] per group if applicable)

## 5. Data Collection

- **Data source(s):** [EHR system / REDCap / registry / paper forms]
- **Collection procedure:** [retrospective pull / prospective entry / survey — who collects]
- **Standardization:** [ICD-10 coding / lab normalization / questionnaire scoring]
- **Quality checks:** [range validation, required-field enforcement, duplicate detection]

## 6. Ethics and Regulatory

- **IRB review type:** [full / expedited / exempt] — [regulatory basis]
- **Privacy/HIPAA:** [de-identified / Limited Data Set / full PHI with authorization]
- **Consent:** [waiver / written consent / assent + parental consent] — [justification]
- **Registration:** [ClinicalTrials.gov required: yes / no; status: not yet registered / NCT#______]
```

---

## Step 4: Quick Review + Update project.yaml

Initialize `review_iteration = 0`. Dispatch Quick Review per `references/reviewer-protocol.md`.

**Original-research-specific review criteria for Stage 3:**
- Is the study design named and matched to the research question aim (prevalence, association, causation, prediction, descriptive)?
- Is the primary outcome operationalized with a specific measurement tool/data source AND a time point — not just named?
- Are all listed covariates justified as confounders with an explicit confounding pathway — not just listed because they were measured?
- Are mediators (if present) distinguished from confounders and excluded from the primary adjustment set?
- Is a sample size justification present — one of: power calculation, feasibility estimate, or precision estimate?
- Do ethics cover all four items: IRB review type + regulatory basis, privacy/HIPAA approach, consent approach, registration status?
- Are eligibility criteria operational for a chart reviewer using standard data access — no criterion requires unavailable clinical judgement?
- Does the protocol align with the research question and study design formulated in Stage 2?

**REVISE loop:**

On **REVISE**:
1. Build REVISION REQUIRED table:
   | Finding | Section | Issue | Required action |
   |---------|---------|-------|----------------|
2. Re-dispatch Methodologist to address each finding — update `study_protocol.md` directly; do not re-run the full PI loop unless a fundamental design element changes
3. Increment `review_iteration += 1`
4. Re-dispatch reviewer with revision history prepended to context

On **APPROVE**: proceed to update `project.yaml`.

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Print escalation banner listing all reviewer findings across iterations and the specific unresolved issues
- Pause pipeline and present to user for resolution

Set `stages.inclusion_exclusion: completed` in `project.yaml`.

Print:
```
✅ Stage 3 complete — study protocol written to 03_inclusion_exclusion/study_protocol.md

Next: Run /database-search-build to design your data collection instrument (Stage 4),
or run /abstract-screening to proceed to data collection (Stage 5).
```
