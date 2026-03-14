# Data Extraction — case_report

Case report data extraction uses the **PI** and **Manuscript Writer** agents to structure a clinical
case presentation per CARE (CAse REport) guidelines. Unlike review-type extractions that pull
quantitative data from many papers, this process distills clinical materials (notes, labs, imaging,
pathology) into a single structured case narrative with literature context.

---

## Step 1: PDF/Clinical Materials Pre-Check

Read all files in `06_data_extraction/full_texts/` (clinical notes, imaging reports, lab results,
pathology reports, clinical photographs, or any other clinical materials provided).

For each file:
1. Read page 1 using: `Read(file_path, pages="1")`
2. If the returned text is empty, very short (<50 characters), or garbled: flag as likely scanned/image-only

Write a readability report to `06_data_extraction/pdf_readability.md`:

```
| Filename | Pages | Readable | Issue |
|----------|-------|----------|-------|
| clinical_notes_2024.pdf | 5 | Yes | — |
| pathology_report.pdf | 2 | No | Scanned image — no text layer |
| imaging_report.pdf | 3 | Yes | — |
```

If any files fail the pre-check, print immediately:

```
⚠️ [N] file(s) appear to be scanned images without a text layer:
  - [filename] — scanned/image-only (no extractable text)

These cannot be read by the pipeline. Options:
1. Run OCR (Adobe Acrobat → Recognize Text, or free: ocrmypdf)
2. Replace with a text-based version from the source system
3. Manually transcribe and place a .txt version in 06_data_extraction/full_texts/

Remaining [N] readable file(s) will be processed now.
```

Continue with readable files only — do not wait for the user to fix unreadable ones.

---

## Step 2: Dispatch PI Agent for Clinical Feature Extraction

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Review all clinical materials for this case report and extract the key clinical
features, diagnostic timeline, and teaching points.

CLINICAL MATERIALS:
[Read and insert each readable file from 06_data_extraction/full_texts/]

RESEARCH QUESTION / TEACHING POINTS:
[Read and insert 02_research_question/research_question.md]

PROJECT CONFIGURATION:
[Read project.yaml — note clinical_coding setting]

Extract the following elements from the clinical materials:

1. **Patient demographics** (age range, sex — de-identified; e.g., "woman in her 40s")
2. **Chief complaint and history of present illness** — presenting symptoms, duration,
   progression, prior treatments attempted
3. **Past medical and surgical history** relevant to the case (comorbidities, medications,
   allergies that bear on the diagnosis or management)
4. **Key physical examination findings** — relevant positives AND pertinent negatives
   that informed the differential
5. **Diagnostic workup timeline** — every test ordered, in strict chronological order:
   - Date or relative timing (Day 1, Week 2, etc.)
   - Test name (lab, imaging modality, biopsy type)
   - Key result or finding
   - Clinical decision triggered by that result
6. **Diagnosis** — final diagnosis with supporting evidence
   [If `review_config.clinical_coding` is `true` in project.yaml: include ICD-10-CM code(s)
   for the primary diagnosis and any secondary diagnoses; add SNOMED CT concept ID if available;
   note any coding ambiguities in a `clinical_coding_notes` field]
7. **Treatment and management decisions** — what was done, dosing/parameters where applicable,
   and rationale for selection over alternatives
8. **Outcome and follow-up** — clinical response, duration of follow-up, any adverse events,
   current status
9. **Teaching points** — cross-reference with the teaching points identified in Stage 2
   (research_question.md); ensure each is addressed with supporting clinical evidence from
   this case
10. **What makes this case reportable** — classify into one or more:
    - Unusual or atypical presentation of a known condition
    - Rare diagnosis (incidence, prevalence if known)
    - Diagnostic pitfall or delay (what was initially missed and why)
    - Unexpected treatment response (positive or negative)
    - Novel association between conditions or exposures
    - New technique, procedure, or therapeutic approach
    - Challenging differential diagnosis with educational value

SELF-CHECK (perform before writing output):
1. All CARE checklist timeline elements are covered — no gaps between presentation and outcome
2. No protected health information (PHI) present: no patient names, dates of birth, medical
   record numbers, institutional identifiers, specific dates (use relative timing instead),
   or any other HIPAA identifier
3. Every teaching point from Stage 2 research_question.md is explicitly addressed with
   clinical evidence from this case — none omitted
4. Diagnostic workup is in strict chronological order — verify sequence
5. Pertinent negatives are documented (not just positive findings)
6. If clinical_coding is true: every diagnosis has an ICD-10-CM code assigned
Apply corrections inline before writing output.

OUTPUT: Structured clinical feature extraction in markdown, organized by the 10 categories above.
Write to 06_data_extraction/case_presentation.md under the heading "## Clinical Feature Extraction".
```

Write to `06_data_extraction/case_presentation.md`.

---

## Step 3: Dispatch Manuscript Writer for Case Structuring per CARE Guidelines

**Manuscript Writer agent prompt:**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Structure the extracted clinical features into a case presentation following
CARE (CAse REport) guidelines. Transform the PI's clinical extraction into a
publication-ready case narrative.

CLINICAL FEATURE EXTRACTION:
[Read and insert 06_data_extraction/case_presentation.md — the PI's output from Step 2]

RESEARCH QUESTION / TEACHING FOCUS:
[Read and insert 02_research_question/research_question.md]

REPORTING STANDARD: CARE guidelines (Gagnier et al., 2013; updated Riley et al., 2017)

Structure the case presentation with the following CARE-compliant sections:

1. **Patient Information**
   - De-identified demographics (age descriptor, sex)
   - Main symptoms and presenting complaint
   - Relevant medical history, comorbidities, and current medications
   - Family history if relevant to the case

2. **Clinical Findings**
   - Physical examination findings organized by system
   - Relevant positive findings that support the diagnosis
   - Pertinent negative findings that narrowed the differential
   - Clinical photographs or dermoscopic findings (reference file names if provided
     in clinical materials; do NOT embed images)

3. **Timeline**
   - Chronological table format:
     ```
     | Timepoint | Event | Finding / Action | Clinical Significance |
     |-----------|-------|------------------|----------------------|
     | Day 0 | Initial presentation | [complaint, exam] | Triggered workup for... |
     | Day 3 | Lab results returned | [key values] | Narrowed differential to... |
     | Week 2 | Biopsy performed | [histopathology result] | Confirmed diagnosis of... |
     ```
   - Every entry must flow logically to the next — no unexplained gaps
   - Include both diagnostic and therapeutic milestones

4. **Diagnostic Assessment**
   - Initial differential diagnosis with clinical reasoning for each considered diagnosis
   - Diagnostic tests performed with results (reference values where appropriate)
   - Process of elimination: which diagnoses were ruled out and by what evidence
   - Final diagnosis with the definitive evidence supporting it

5. **Therapeutic Intervention**
   - Treatment selected with specific details (drug name, dose, route, frequency, duration;
     or procedure name with technique details)
   - Rationale for treatment selection over alternatives (cite evidence if available)
   - Changes or adjustments made during the course of treatment and why
   - Any concomitant therapies

6. **Follow-up and Outcomes**
   - Clinician-assessed outcomes (objective measures, clinical response grading)
   - Patient-reported outcomes if documented in clinical materials
   - Adverse events or complications (including "none" if uneventful)
   - Duration and status at last follow-up
   - Long-term management plan if applicable

7. **Discussion Points** (preliminary — to be expanded in manuscript stage)
   - Key clinical lessons from this case
   - How this case advances understanding of the condition
   - Practical implications for clinicians encountering similar presentations
   - Literature context placeholders: [TO BE EXPANDED WITH LANDSCAPE REPORT IN STEP 4]

SELF-CHECK (perform before writing to disk):
1. CARE checklist compliance — verify each of the 13 CARE checklist items is addressed:
   □ Title (keywords identifying it as a case report)
   □ Key words (relevant to the case)
   □ Abstract (structured: introduction, case, discussion, conclusion)
   □ Introduction (brief background, why this case is reportable)
   □ Patient information (demographics, symptoms, history)
   □ Clinical findings (exam)
   □ Timeline (chronological)
   □ Diagnostic assessment (differential reasoning, tests, diagnosis)
   □ Therapeutic intervention (type, dosage, duration)
   □ Follow-up and outcomes (clinical and patient-reported)
   □ Discussion (strengths, limitations, relevant literature)
   □ Patient perspective (if available)
   □ Informed consent (note: flag if not documented in clinical materials)
2. Timeline is strictly chronological — no date/event is out of order
3. No PHI present — re-verify: no names, DOB, MRN, institutional identifiers, exact dates
   (all dates converted to relative timepoints: Day 0, Week 2, Month 3, etc.)
4. Patient consent for publication is flagged if not found in clinical materials:
   add a note "⚠️ Patient consent for publication not documented in provided materials —
   must be obtained before submission"
Apply corrections before writing to disk.

OUTPUT: Replace the contents of 06_data_extraction/case_presentation.md with the full
CARE-structured case presentation. Retain the PI's extraction as an appendix section
("## Appendix: Raw Clinical Feature Extraction") at the bottom of the file for audit trail.
```

Overwrite `06_data_extraction/case_presentation.md` with the structured output (PI extraction
preserved as appendix).

---

## Step 4: Dispatch PI for Literature Context

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Provide literature context for this case report by comparing the case to
existing published literature.

CASE PRESENTATION:
[Read and insert 06_data_extraction/case_presentation.md — the Writer's structured output]

LANDSCAPE REPORT:
[Read and insert 01_literature_search/landscape_report.md]

RESEARCH QUESTION:
[Read and insert 02_research_question/research_question.md]

Using the landscape report and your clinical expertise, produce:

1. **Comparison to Published Cases**
   - How does this patient's presentation compare to published case reports or series
     of the same condition?
   - Are there demographic, clinical, or outcome patterns that align with or diverge
     from the literature?
   - If similar cases have been reported, what distinguishes this one?

2. **Novelty and Instructive Value**
   - What specific aspect of this case adds to the existing body of knowledge?
   - Is this the first report of [specific finding], or does it reinforce an emerging pattern?
   - What diagnostic or therapeutic lesson is generalizable beyond this single patient?

3. **Clinical Implications for Practice**
   - What should clinicians take away from this case?
   - Are there changes to diagnostic workup, differential diagnosis, or management
     that this case supports?
   - Should this case change clinical suspicion thresholds for the condition?

SELF-CHECK:
1. Every claim about the literature is traceable to a specific source in landscape_report.md
   or a cited reference — no unsupported assertions
2. The novelty claim is specific, not generic ("first report of X in Y population"
   vs. "this is an interesting case")
3. Clinical implications are actionable and grounded in the case evidence

OUTPUT: Append to 06_data_extraction/case_presentation.md under the heading
"## Literature Context and Clinical Implications" with the three subsections above.
Do NOT overwrite existing content — append only.
```

Append to `06_data_extraction/case_presentation.md`.

---

## Step 5: Quick Review

Initialize `review_iteration = 0`.

Dispatch Quick Review per `references/reviewer-protocol.md`.

**Review criteria for case_report data extraction (provide these to the reviewer in the dispatch prompt):**

Clinical Completeness and CARE Compliance:
- Is the clinical timeline complete and strictly chronological — no unexplained gaps between
  presentation and final outcome?
- Are all 13 CARE guideline elements addressed (title, keywords, abstract, introduction,
  patient information, clinical findings, timeline, diagnostic assessment, therapeutic
  intervention, follow-up/outcomes, discussion, patient perspective, informed consent)?
- Is the diagnostic assessment logically structured — differential considered, tests performed
  with results, process of elimination documented, final diagnosis supported by evidence?

Patient Privacy:
- Is patient privacy maintained throughout — no names, dates of birth, medical record numbers,
  institutional identifiers, or specific calendar dates?
- Are all dates expressed as relative timepoints (Day 0, Week 2, Month 3)?
- Could the combination of demographic details + diagnosis + institution allow re-identification?
  (Flag if the case is so rare that de-identification may be insufficient.)

Teaching Points and Novelty:
- Are the teaching points clinically grounded and distinct from each other (not restating
  the same point in different words)?
- Do the teaching points map to those identified in Stage 2 (research_question.md)?
- Is the "what makes this case reportable" claim specific and supported by the clinical evidence?

Literature Context:
- Are literature comparisons traceable to sources in landscape_report.md?
- Is the novelty claim specific (not "this is an interesting case")?
- Are clinical implications actionable for the target audience?

Diagnosis and Workup:
- Is the final diagnosis supported by the documented workup (not asserted without evidence)?
- Are pertinent negative findings documented alongside positives?
- If clinical coding is enabled: are ICD-10-CM codes present and appropriate for the diagnosis?

On **REVISE** verdict:
1. Build a REVISION REQUIRED table: finding | affected section | required action
2. Determine which agent owns the section:
   - Clinical feature issues (missing history, exam findings, timeline gaps) → re-dispatch PI
   - Structural/CARE compliance issues (formatting, section organization, narrative flow) →
     re-dispatch Manuscript Writer
   - Literature context issues → re-dispatch PI
3. Re-dispatch the appropriate agent(s) with the REVISION REQUIRED table prepended to their
   original prompt; each finding must be addressed explicitly
4. Update `06_data_extraction/case_presentation.md` with corrections
5. Increment `review_iteration`; re-dispatch reviewer with revision history appended

On **APPROVE**: proceed to Step 6.

On **REJECT** or `review_iteration >= 2` without APPROVE: trigger full escalation per
`references/reviewer-protocol.md`:

```
🚨 ESCALATION — Case Report Data Extraction

Review iteration limit reached without APPROVE.

Unresolved findings:
[List each unresolved finding from the reviewer's latest verdict]

Files requiring PI attention:
  - 06_data_extraction/case_presentation.md

Action required: PI must manually review and resolve the flagged issues before
the pipeline can proceed to manuscript writing.
```

Pause pipeline — do not proceed to Step 6.

---

## Step 6: Update project.yaml

Set `stages.data_extraction: completed`.

Print:

```
✅ Case report data extraction complete.

Output: 06_data_extraction/case_presentation.md

Sections written:
  • Patient Information (de-identified)
  • Clinical Findings (exam, pertinent negatives)
  • Timeline (chronological table)
  • Diagnostic Assessment (differential, workup, final diagnosis)
  • Therapeutic Intervention (treatment details, rationale)
  • Follow-up and Outcomes
  • Discussion Points
  • Literature Context and Clinical Implications

CARE guideline compliance: verified by Quick Review
PHI check: passed (no identifiers detected)

Next stage: manuscript writing (Stage 7)
```
