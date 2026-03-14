---
name: inclusion-exclusion
description: |
  Use when the user wants to define inclusion and exclusion criteria, set study eligibility,
  or determine what studies to include in a review. Also triggers for study protocol design
  in original research projects.
  Trigger on phrases like "inclusion criteria", "exclusion criteria", "eligibility",
  "what studies to include", "study protocol", or "selection criteria".
---

# Inclusion/Exclusion Criteria (Stage 3)

Define inclusion and exclusion criteria (or study protocol for original research) using AI agent teams.

## Prerequisites

- `02_research_question/research_question.md` must exist
- Read `project.yaml` for `project_type` and `review_config`

## Process — Review Types (systematic_review, scoping_review, rapid_review, meta_analysis)

### Step 1: Dispatch Methodologist Agent

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Draft inclusion and exclusion criteria for the following research question.

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

PROJECT TYPE: [from project.yaml]
REPORTING STANDARD: [from project.yaml review_config.reporting_standard]
DATE RANGE: [from project.yaml review_config.date_range]

**If `project_type` is `scoping_review`, use the PCC framework (Population, Concept, Context) instead of PICO. Replace items 2–4 below with: CONCEPT (what topic, category, or phenomenon is being mapped — keep broad), CONTEXT (what setting, country, time period, or healthcare system is in scope), and omit a quantitative Outcome criterion entirely. Scoping reviews map breadth of evidence, not narrow outcomes.**

Draft criteria covering:
1. POPULATION: Who/what is included? Age, setting, condition specifics.
2. INTERVENTION/EXPOSURE (or CONCEPT for scoping reviews): What interventions, exposures, or topics are included? For scoping reviews, frame as "What is the concept being mapped?" — keep broad.
3. COMPARATOR (or CONTEXT for scoping reviews): What comparisons are acceptable? For scoping reviews, replace with CONTEXT: what setting or circumstances bound the scope?
4. OUTCOMES (omit for scoping reviews): What outcomes must be reported for inclusion? Not required for scoping reviews.
5. STUDY DESIGN: What study types are included? For scoping reviews: all study designs typically included.
6. TIME FRAME: Publication date range.
7. LANGUAGE: English only? Other languages?
8. EXCLUSION CRITERIA: Specific reasons for exclusion (duplicates, conference abstracts only, animal studies, etc.)

For scoping reviews: criteria should be broader — the goal is to map the literature, not filter narrowly. Use PCC structure in the output table (rows: Population, Concept, Context, Study design, Publication date, Language) — omit Intervention/Exposure, Comparator, and Outcomes rows.

**SELF-CHECK (required before outputting criteria):**

Verify each item below. Correct any failures inline before producing output — do not pass failing criteria to the Librarian:

- [ ] Every PICO/PEO component from the research question maps to at least one inclusion criterion
- [ ] Each criterion is operationalizable: a screener can apply it consistently from a title/abstract (no criterion requires subjective clinical judgement to apply)
- [ ] No internally contradictory pairs (e.g., "RCTs only" in inclusion AND "studies without a control group excluded" in exclusion — these are redundant, not contradictory, but verify logic)
- [ ] Exclusion criteria do not duplicate inclusion criterion negations — exclusions add information, they don't just restate what is already outside the inclusion set
- [ ] Study design criteria are appropriate for the project type: `meta_analysis` → prioritize RCTs/controlled studies; `scoping_review` → broad, all study types unless clearly irrelevant; `systematic_review` → explicitly justified
- [ ] Date range is present and justified by clinical rationale (e.g., "from [year] — introduction of [intervention]") not arbitrary
- [ ] Language restriction (English only vs. multilingual) is stated with a brief rationale

OUTPUT: Structured criteria table with clear, operationalizable definitions for each criterion.
```

### Step 2: Dispatch Librarian Agent

**Librarian agent prompt:**

```
[Insert Librarian system prompt from agent-roles.md]

TASK: Validate that these criteria are searchable in standard databases.

CRITERIA:
[Insert Methodologist's output]

For each criterion, confirm:
1. Can this be mapped to MeSH terms, Emtree terms, or keyword searches?
2. Are there any criteria that would be impossible to screen from title/abstract alone?
3. Suggest modifications if any criteria are unsearchable or impractical.

**SELF-CHECK (required before outputting annotations):**

- [ ] Every criterion has been annotated — no criterion left without a searchability verdict
- [ ] Use WebSearch to verify that key terms (condition, intervention, population) have confirmed MeSH headings — do not guess MeSH terms
- [ ] Any criterion flagged as "full-text only" must include a specific reason and an alternative abstract-level proxy screeners can use in the interim
- [ ] If ≥3 criteria are flagged as full-text only, add a summary note recommending the PI consider simplifying — this volume of deferred screening will inflate full-text retrieval burden significantly

OUTPUT: Annotated version of the criteria with searchability notes.
```

### Step 3: Dispatch PI Agent

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Review these inclusion/exclusion criteria for clinical relevance.

CRITERIA (with Librarian annotations):
[Insert Librarian's output]

Review and confirm:
1. Are the criteria clinically meaningful?
2. Do they capture the right population and interventions?
3. Are they too broad (will include irrelevant studies) or too narrow (will miss important studies)?
4. Final approval or requested changes.

OUTPUT: Final approved criteria or specific revision requests.
```

### Step 4: Write Output

Write to `03_inclusion_exclusion/criteria.md` using the appropriate structure for the project type:

**For `systematic_review`, `rapid_review`, `meta_analysis` (PICO structure):**

```markdown
# Inclusion/Exclusion Criteria

## Inclusion Criteria

| Criterion | Definition | Rationale |
|-----------|-----------|-----------|
| Population | [spec] | [why] |
| Intervention/Exposure | [spec] | [why] |
| Comparator | [spec or N/A] | [why] |
| Outcomes | [spec] | [why] |
| Study design | [spec] | [why] |
| Publication date | [range] | [why] |
| Language | [spec] | [why] |

## Exclusion Criteria

| Criterion | Rationale |
|-----------|-----------|
| [spec] | [why] |

## Notes on Searchability
[Librarian annotations on any criteria that require screening at full-text stage only]
```

**For `scoping_review` (PCC structure):**

```markdown
# Eligibility Criteria (Scoping Review — PCC Framework)

## Inclusion Criteria

| PCC Component | Definition | Rationale |
|---------------|-----------|-----------|
| Population (P) | [spec — who is the review about?] | [why] |
| Concept (C) | [spec — what topic, category, or phenomenon is being mapped?] | [why] |
| Context (Co) | [spec — what setting, country, time period, or system is in scope?] | [why] |
| Study design | All designs unless specified otherwise | Scoping reviews map breadth |
| Publication date | [range] | [why] |
| Language | [spec] | [why] |

## Exclusion Criteria

| Criterion | Rationale |
|-----------|-----------|
| [spec] | [why] |

## Notes on Searchability
[Librarian annotations — note that Context is often not directly searchable via MeSH and must be applied at screening]
```

### Step 5: PROSPERO Protocol Generation (conditional)

If `project.yaml` has `review_config.prospero_registration: true`:

Dispatch Methodologist agent to generate a PROSPERO-ready protocol summary:

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Generate a PROSPERO registration protocol summary.

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

INCLUSION/EXCLUSION CRITERIA:
[Read and insert 03_inclusion_exclusion/criteria.md]

PROJECT.YAML SETTINGS:
[Insert relevant fields: databases, date_range, reporting_standard, subgroup_analyses,
sensitivity_analyses, meta_analysis, network_meta_analysis]

Produce a document structured for PROSPERO submission with the following sections
(matching PROSPERO form fields):
1. Review title
2. Review question (PICO/PEO)
3. Searches (databases, date range, any restrictions)
4. Eligibility criteria (population, intervention, comparator, outcomes, study design)
5. Information sources and search strategy (note: full strategy in supplement)
6. Study selection (process, number of screeners, conflict resolution)
7. Data extraction (process, number of extractors, template basis)
8. Risk of bias / quality assessment (tool and rationale)
9. Strategy for data synthesis (narrative / meta-analytic; heterogeneity handling)
10. Analysis of subgroups or subsets (if pre-specified)

OUTPUT: Structured markdown document ready to copy-paste into PROSPERO fields.
```

Write to `03_inclusion_exclusion/prospero_protocol.md`.

Print:
```
📋 PROSPERO protocol generated: 03_inclusion_exclusion/prospero_protocol.md

Register your protocol at: https://www.crd.york.ac.uk/prospero/
Tip: Register BEFORE running database searches (Stage 4). Most journals require
pre-registration to have occurred prior to search execution.
```

### Step 6: Quick Review + Update project.yaml

Dispatch a single Independent Reviewer per Quick Review protocol. Initialize `review_iteration = 0`.

**Review criteria for this stage:**
- Are all PICO/PEO components from `research_question.md` addressed in the criteria? (alignment check)
- Is each criterion operationalizable — can a screener apply it consistently from a title/abstract alone, or does it require full-text?
- Are any criteria internally contradictory (e.g., inclusion requires RCTs, exclusion removes studies without a control group)?
- Are any criteria so broad they will generate an unmanageable number of results (>1000 expected)?
- Are any criteria so narrow they risk excluding important studies?
- For `meta_analysis`: do the outcome and study design criteria ensure sufficient homogeneity for quantitative pooling?
- For `scoping_review`: are criteria broad enough to map the landscape, not arbitrarily restrictive?
- For `network_meta_analysis` (if `review_config.network_meta_analysis: true`): do inclusion criteria permit indirect comparisons (multiple treatment arms accepted)?
- Is the language restriction justified or should grey literature be included?

**Review iteration loop:**

On **APPROVE**: proceed to update `project.yaml`.

On **REVISE**:
- Build a REVISION REQUIRED table:
  | Finding | Criterion affected | Issue | Required action |
  |---------|-------------------|-------|----------------|
- Re-dispatch Methodologist to address each finding — update `criteria.md` directly, do not re-run the full Librarian/PI loop unless a criterion is being fundamentally changed
- Increment `review_iteration += 1`
- Re-dispatch reviewer with revision history prepended to context

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Print escalation banner listing all reviewer findings across iterations and the specific unresolved issues
- Pause pipeline and present to user for resolution before proceeding

Set `stages.inclusion_exclusion: completed` in `project.yaml`.

## Process — qualitative_synthesis

Qualitative synthesis uses the **PICo framework** (Population, phenomenon of Interest, Context) — not PICO. There is no Intervention, Comparator, or quantitative Outcome. The eligibility criteria define which qualitative studies address the phenomenon of interest in the target population and context.

### Step 1: Dispatch Methodologist Agent

**Methodologist agent prompt:**

```
[Insert Methodologist system prompt from agent-roles.md]

TASK: Draft eligibility criteria for a qualitative evidence synthesis using the PICo framework.

RESEARCH QUESTION (PICo):
[Read and insert 02_research_question/research_question.md]

PROJECT TYPE: qualitative_synthesis
REPORTING STANDARD: [from project.yaml — typically ENTREQ]
SYNTHESIS APPROACH: [from project.yaml review_config.synthesis_approach — thematic / framework / meta_ethnography / meta_aggregation]

Draft criteria covering:
1. POPULATION (P): Who is the study about? Specify age group, condition, role (e.g., patients, caregivers, clinicians), and any restrictions (diagnosis, disease stage, setting).
2. PHENOMENON OF INTEREST (I): What experience, perception, process, or behaviour must the study explore? Be specific — vague phenomena produce unmanageable result sets. Distinguish the target phenomenon from related-but-excluded topics.
3. CONTEXT (Co): What setting, culture, healthcare system, country, or time period is eligible? Specify any restrictions and the rationale (e.g., "high-income country settings only — because care access differences would make findings non-transferable").
4. STUDY DESIGN: Qualitative or mixed-methods with a stand-alone qualitative component. Enumerate acceptable designs: semi-structured interviews, in-depth interviews, focus groups, ethnography, grounded theory, phenomenology, thematic analysis of primary data, participatory action research, narrative inquiry. Explicitly exclude: RCTs, cohort studies, case-control studies, quantitative surveys with no qualitative data, systematic reviews.
5. TIME FRAME: Publication date range with rationale.
6. LANGUAGE: Which languages are eligible? State rationale.
7. EXCLUSION CRITERIA: Conference abstracts (insufficient data), dissertations if full text unavailable, studies reporting only quantitative findings from a mixed-methods design, studies with participants outside the defined population.

**SELF-CHECK (required before outputting criteria):**
- [ ] All three PICo components (P, I, Co) are explicitly operationalized — not vague umbrella terms
- [ ] Phenomenon of Interest is specific enough that a screener can distinguish in-scope from adjacent phenomena from title/abstract
- [ ] Study design criteria name the acceptable qualitative methods; mixed-methods handling is stated explicitly
- [ ] No PICO-style Intervention, Comparator, or quantitative Outcome fields appear — these are not applicable
- [ ] Language and date restrictions stated with rationale
- [ ] Exclusion criteria do not duplicate the negation of inclusion criteria; they add specific non-obvious exclusion rules

OUTPUT: PICo-structured eligibility criteria table with operationalizable definitions.
```

### Step 2: Dispatch Librarian Agent

**Librarian agent prompt:**

```
[Insert Librarian system prompt from agent-roles.md]

TASK: Validate that these PICo criteria are searchable in standard databases.

CRITERIA:
[Insert Methodologist's output]

For each PICo component, confirm:
1. Can the Population be mapped to MeSH terms or keyword searches?
2. Can the Phenomenon of Interest be captured with available controlled vocabulary + free-text synonyms?
3. Are there qualitative study design filters available for each target database (PubMed qualitative filter, Ovid qualitative filter)?
4. Are there criteria that can only be confirmed at full-text (e.g., whether a mixed-methods paper has a useable qualitative component)?

Note: Context (Co) is often not searchable via MeSH — advise whether to add it as a keyword filter or handle at screening.

SELF-CHECK:
- [ ] Use WebSearch to confirm MeSH terms for key Population and Phenomenon descriptors
- [ ] Verify that a qualitative study design filter exists for each database in scope
- [ ] Flag full-text-only criteria explicitly with interim abstract-level proxy instructions

OUTPUT: Annotated criteria with searchability verdict per criterion.
```

### Step 3: Dispatch PI Agent

Same as Review Types path. PI reviews for clinical/substantive relevance of the PICo components and ensures the phenomenon of interest is appropriately scoped.

### Step 4: Write Output

Write to `03_inclusion_exclusion/criteria.md` using this PICo structure:

```markdown
# Eligibility Criteria (Qualitative Synthesis)

## Inclusion Criteria

| PICo Component | Definition | Rationale |
|----------------|-----------|-----------|
| Population (P) | [spec] | [why] |
| Phenomenon of Interest (I) | [spec — specific experience/behaviour/process] | [why] |
| Context (Co) | [spec — setting, country, healthcare system] | [why] |
| Study design | Qualitative studies: [list accepted methods]. Mixed-methods: included if qualitative component stands alone. | Qualitative data required |
| Publication date | [range] | [why] |
| Language | [spec] | [why] |

## Exclusion Criteria

| Criterion | Rationale |
|-----------|-----------|
| Quantitative-only study designs (RCTs, cohort, case-control, surveys) | No qualitative data to synthesize |
| [additional exclusions] | [why] |

## Notes on Searchability
[Librarian annotations — qualitative study design filters per database, any full-text-only criteria]
```

### Step 5: Quick Review + Update project.yaml

Same Quick Review loop as Review Types. **Replace review criteria with PICo-specific criteria:**

- Are all three PICo components (P, I, Co) explicitly operationalized — not vague?
- Is the Phenomenon of Interest specific enough to distinguish in-scope from adjacent topics?
- Does the study design criteria correctly include qualitative designs while excluding quantitative?
- Is the mixed-methods handling rule stated explicitly?
- For `synthesis_approach: meta_ethnography`: are criteria scoped broadly enough to allow reciprocal/refutational translation (i.e., conceptually diverse studies needed, not just similar findings)?
- For `synthesis_approach: meta_aggregation` (JBI): are criteria specific enough to yield studies with unambiguous findings (JBI requires clear findings, not emergent themes)?

Follow the same REVISE/REJECT iteration loop. Set `stages.inclusion_exclusion: completed` in `project.yaml`.

---

## Process — Original Research

For `project_type: original_research`, this stage produces a study protocol (not eligibility criteria for a literature review). The protocol requires a structured study design, operationalized participant eligibility, variable definitions, sample size justification, data collection plan, and ethical framework.

Read `skills/inclusion-exclusion/section-original-research.md` and follow it exactly.

## Process — diagnostic_test_accuracy

DTA reviews require PICOS framing and specific eligibility rules around spectrum bias and verification bias. Do NOT route DTA reviews through the generic PICO path above.

Read `skills/inclusion-exclusion/section-dta.md` and follow it exactly.

---

## Process — Case Report

This stage is skipped for case reports. Print:

```
Stage 3 (Inclusion/Exclusion Criteria) is skipped for case reports.
Run /database-search-build to continue, or skip to /data-extraction.
```
