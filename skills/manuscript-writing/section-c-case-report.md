# Section C: Case Report Manuscript

_For `case_report` projects only. Uses CARE 2013 reporting guideline (13 core items). No PRISMA flow, no statistical pooling._

### C1: Fetch Journal Guidelines

Use WebFetch to retrieve guidelines from `journal_guidelines_url` in `project.yaml`. Extract:
- Word limits (abstract and full text)
- Required sections and order (some journals use patient-centered section headings)
- Reference style
- Patient consent statement requirements (some journals require a specific form)
- Figure/table limits

If `journal_guidelines_url` is blank: print "Journal guidelines URL not set — using CARE 2013 defaults. Please paste specific journal guidelines if available." and proceed with standard CARE structure.

### C2: Dispatch Manuscript Writer A (Introduction + Case Narrative)

**Writer A agent prompt:**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft the INTRODUCTION and CASE PRESENTATION sections for a case report.

REPORTING STANDARD: CARE 2013 (13-item Case Report Guidelines)
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary or CARE defaults]

SOURCE MATERIALS:
- Landscape report: [Read 01_literature_search/landscape_report.md]
- Research question / teaching objectives: [Read 02_research_question/research_question.md]
- Case presentation: [Read 06_data_extraction/case_presentation.md]

TITLE should:
- Indicate "case report" in the title (CARE item 1)
- Be informative: include the diagnosis/condition and key feature that makes this case novel

INTRODUCTION (CARE item 5) should:
- State what is unique or rare about this case and why it merits publication
- Briefly summarize relevant background (existing literature from landscape report)
- State the educational or clinical value (teaching objectives from research question file)
- Keep to 3-4 paragraphs; do not repeat what will be in the case narrative

PATIENT INFORMATION (CARE item 6) should include:
- De-identified demographics (age, sex — do NOT include name, DOB, or identifiable details)
- Chief complaint in the patient's words (if available from case_presentation.md)
- Relevant medical, family, and psychosocial history
- Relevant past interventions and outcomes

CLINICAL FINDINGS (CARE item 7):
- Physical examination findings with specific measurements and descriptors

OUTPUT: Title, Introduction, Patient Information, Clinical Findings in markdown.
```

### C3: Dispatch Manuscript Writer B (Timeline + Diagnostics + Treatment + Outcomes + Discussion)

**Writer B agent prompt:**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Draft the TIMELINE, DIAGNOSTIC ASSESSMENT, THERAPEUTIC INTERVENTION, FOLLOW-UP AND OUTCOMES, and DISCUSSION sections for a case report.

REPORTING STANDARD: CARE 2013
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary or CARE defaults]

SOURCE MATERIALS:
- Research question / teaching objectives: [Read 02_research_question/research_question.md]
- Case presentation: [Read 06_data_extraction/case_presentation.md]
- Landscape report (for discussion context): [Read 01_literature_search/landscape_report.md]

TIMELINE (CARE item 8):
- Chronological table or figure of important dates and events (symptom onset, clinical visits, diagnostic steps, treatments, outcomes)
- Use relative time references (e.g., "Day 0", "Week 2") rather than calendar dates for anonymization

DIAGNOSTIC ASSESSMENT (CARE item 9) should include:
- Diagnostic methods used (examination, labs, imaging, biopsy, functional tests)
- Diagnostic challenges or delays, if relevant
- Differential diagnoses considered and how they were excluded
- Final diagnosis with the basis for diagnosis clearly stated

THERAPEUTIC INTERVENTION (CARE item 10) should include:
- Types of interventions (pharmacological, surgical, procedural, lifestyle)
- How each was administered (dose, route, frequency, duration)
- Any changes to the treatment plan and the reasons for them

FOLLOW-UP AND OUTCOMES (CARE item 11) should include:
- Clinician-assessed outcomes with timepoints and measurement methods
- Patient-reported outcomes (symptoms, function, quality of life) where available
- Treatment adherence and tolerability
- Adverse effects if any occurred

DISCUSSION (CARE item 12) should include:
- Summary of the key clinical lessons from this case (2-3 specific teaching points tied to teaching objectives)
- Comparison with existing published cases or guidelines (cite relevant literature from landscape report)
- Explanation of why the diagnosis, management, or outcome was unusual or instructive
- Limitations of the case: what cannot be generalized, reporting biases
- Conclusion: concise statement of clinical take-away

PATIENT PERSPECTIVE (CARE item 13 — optional):
- If the patient provided a statement about their experience, include it as a brief section or sidebar
- If not available, omit the section and note it as not obtained

OUTPUT: Timeline, Diagnostic Assessment, Therapeutic Intervention, Follow-up and Outcomes, Discussion in markdown.
```

### C4: Merge and Self-Check

Dispatch Writer A as merge agent to combine all sections into a complete case report manuscript.

**Merge agent prompt:**

```
[Insert Manuscript Writer system prompt from agent-roles.md]

TASK: Merge the following sections into one cohesive CARE-compliant case report manuscript.

INTRODUCTION + CASE NARRATIVE (Writer A output):
[Insert Writer A's output]

TIMELINE + ASSESSMENT + TREATMENT + OUTCOMES + DISCUSSION (Writer B output):
[Insert Writer B's output]

JOURNAL GUIDELINES: [Insert guidelines summary]

Merge into a complete case report with:
1. Title page (title, authors placeholder, affiliations placeholder, corresponding author)
2. Abstract (structured if required: Background, Case presentation, Conclusions — CARE item 2)
3. Keywords (3-6 terms — CARE item 3)
4. Introduction (CARE item 5)
5. Patient Information (CARE item 6)
6. Clinical Findings (CARE item 7)
7. Timeline (CARE item 8)
8. Diagnostic Assessment (CARE item 9)
9. Therapeutic Intervention (CARE item 10)
10. Follow-up and Outcomes (CARE item 11)
11. Discussion (CARE item 12)
12. Patient Perspective (CARE item 13 — if available)
13. Informed Consent statement (CARE item 4 — placeholder: "Patient consent was obtained.")
14. Acknowledgements (placeholder)
15. Declarations (funding, conflicts — placeholder)
16. References
17. Figure legends (for timeline figure if included)

**SELF-CHECK (required before writing output to disk):**

CARE Compliance:
- [ ] CARE items 1–12 are all addressed in the manuscript
- [ ] Title includes "case report" (CARE item 1)
- [ ] Abstract covers background, case presentation, and conclusions (CARE item 2)
- [ ] Informed consent statement is present (CARE item 4 — may be a placeholder)
- [ ] Timeline is present as a table or chronological list (CARE item 8)
- [ ] Final diagnosis is explicitly stated in Diagnostic Assessment (CARE item 9)
- [ ] Treatment changes are documented with reasons (CARE item 10)

Patient Privacy:
- [ ] No identifying information present (no name, DOB, or specific calendar dates)
- [ ] Age stated as range or approximate if needed for anonymization
- [ ] Any images that could identify the patient have an anonymization note

Writing Quality:
- [ ] Tense consistent: past tense in case narrative; present tense for established clinical facts
- [ ] All abbreviations defined at first use
- [ ] All in-text citations have corresponding reference entries
- [ ] No `[CITATION NEEDED]` flags left unresolved
- [ ] References include the journal article for any landmark guidelines cited
- [ ] Word count within journal limits

If any check fails, correct before producing final output.

OUTPUT: Complete case report manuscript in markdown + references in BibTeX format.
```

Write to `07_manuscript/manuscript.md` and `07_manuscript/references.bib`.

### C4b: Proofreader Pass

Dispatch a Proofreader agent to catch linguistic, structural, and formatting issues.

**Proofreader agent prompt:**

```
[Insert Proofreader system prompt from agent-roles.md]

TASK: Proofread the following case report manuscript before peer review.

MANUSCRIPT: [Read 07_manuscript/manuscript.md]
TARGET JOURNAL: [from project.yaml]
JOURNAL GUIDELINES: [Insert fetched guidelines summary]
REPORTING STANDARD: CARE 2013 (13 items)

Perform a full proofread. In addition to standard checks, verify:
- CARE items 1–13 are all addressed
- Patient privacy: no identifying information (name, DOB, specific calendar dates)
- Timeline section uses relative dates (Day 0, Week 2), not calendar dates
- Informed consent statement present
- No statistical pooling or review-type language
- Clinical terminology is precise and consistent throughout

OUTPUT: Proofread report in the standard format.
```

Write to `07_manuscript/proofread_report.md`.

If Errors found: re-dispatch merge agent to apply corrections, then proceed.

### C5: Full Review + Update project.yaml

Dispatch Full Review per `references/reviewer-protocol.md`. Initialize `review_iteration = 0`.

**Review criteria (provide to each reviewer):**

CARE Guideline Compliance:
- Are all 12 mandatory CARE items present and adequately addressed?
- Is an informed consent statement present?
- Is the timeline table/figure present and chronologically logical?

Clinical Content:
- Is the diagnostic reasoning clearly explained and clinically sound?
- Is the treatment rationale documented (why this treatment, why changes were made)?
- Are the outcomes reported with specific measurements, not just narrative impressions?

Teaching Value:
- Does the Discussion clearly state 2-3 specific teaching points?
- Are the teaching points linked to documented case facts (not generic statements)?
- Does the Discussion compare the case to published cases or guidelines?

Patient Privacy:
- Is the case adequately anonymized (no identifying information)?

Writing:
- Is tense consistent throughout?
- Are all abbreviations defined at first use?

**Review iteration loop:**

On **APPROVE**: proceed to Step C6.

On **REVISE** (any reviewer):
- Build REVISION REQUIRED table (reviewer | CARE item | finding | location | required action)
- Re-dispatch merge agent with the table and instruction to address all findings
- Update `07_manuscript/manuscript.md`
- Increment `review_iteration += 1`
- Re-dispatch Full Review with revision history

On **REJECT** or `review_iteration ≥ 2` without APPROVE:
- Trigger full escalation per `references/reviewer-protocol.md`
- Pause pipeline for PI resolution

### C6: Update project.yaml

Set `stages.manuscript_writing: completed`.

Print:

```
✅ Case report manuscript complete!

📄 Files:
- Manuscript: 07_manuscript/manuscript.md
- References: 07_manuscript/references.bib

📋 NEXT STEPS:
1. Review the manuscript carefully — ensure clinical details are accurate and the case is fully anonymized
2. Add author names, affiliations, and contact details
3. Obtain and document patient informed consent (required by most journals)
4. Complete declarations (funding, conflicts of interest, ethics/IRB exemption)
5. Convert to the journal's required submission format (Word/LaTeX)
6. Submit!
```
