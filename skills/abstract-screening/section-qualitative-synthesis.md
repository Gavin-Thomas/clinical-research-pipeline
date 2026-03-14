# Abstract Screening — Qualitative Synthesis Process

_Read and follow this file when `project_type: qualitative_synthesis`. Qualitative synthesis screening uses the same 3-extractor parallel structure as systematic reviews, but applies a PICo-based decision framework and qualitative-specific screening criteria. The goal is to identify studies that report original qualitative data (verbatim participant quotes, author-interpreted themes, experiential findings) relevant to the phenomenon of interest._

---

## Step 1: Dispatch 3 Data Extractor Agents in Parallel

Use the same parallel dispatch structure as the Review Types process. **Replace the decision framework block** in the Data Extractor prompt with:

```
DECISION RULES (Qualitative synthesis — PICo framework):

STEP A — Study design filter (apply first; terminate early if quantitative):
- EXCLUDE immediately: RCTs, cohort studies, case-control studies, cross-sectional surveys with no qualitative component, quantitative systematic reviews, economic analyses, protocol papers, animal studies.
  Cite reason as: "Study design: no qualitative data reported"
- INCLUDE: Clearly qualitative studies — semi-structured interviews, in-depth interviews, focus groups, ethnography, grounded theory, phenomenology, participatory action research, narrative inquiry, or thematic analysis of primary observational/interview data.
- UNCERTAIN (design): Mixed-methods studies — UNCERTAIN if the abstract suggests a distinct qualitative component that may stand alone. Do NOT exclude mixed-methods solely for containing quantitative data. State the apparent qualitative method in your reasoning.

STEP B — PICo relevance filter (only if study design is qualitative or uncertain):
- P (Population): Does the studied population match the population defined in criteria.md?
- I (Phenomenon of Interest): Does the study explore the experience, perception, behaviour, or process defined as the phenomenon of interest in criteria.md?
- Co (Context): Is the study conducted in a context (setting, culture, healthcare system, time period) consistent with the eligibility criteria?
- If all three PICo components match → INCLUDE. If any are clearly outside → EXCLUDE with specific PICo component cited. If information is insufficient → UNCERTAIN.

Default bias: When the abstract is borderline, prefer INCLUDE. Full-text review resolves design and focus uncertainty.

EXTRA OUTPUT FIELD — add to each row:
  qualitative_method: The specific qualitative method identified in the abstract (e.g., "semi-structured interviews", "focus groups", "ethnography", "grounded theory", "phenomenology", "thematic analysis", "mixed-methods", "NR").
  This field is used by Stage 6 to select the appropriate synthesis approach.
```

**SELF-CHECK (add to standard checklist):**
6. Every EXCLUDE citing "Study design" is genuinely quantitative — verify no qualitative component was mentioned.
7. Mixed-methods studies are UNCERTAIN (not EXCLUDE) if a qualitative component exists.
8. `qualitative_method` is populated for every INCLUDE and UNCERTAIN row.

**Output CSV columns:**
`id, title, decision, criterion, qualitative_method, reasoning, confidence`

---

## Step 2: Dispatch Methodologist for Reconciliation

Same majority-vote reconciliation as the Review Types process. Add to the Methodologist prompt:

```
ADDITIONAL INSTRUCTIONS for qualitative synthesis reconciliation:

When reconciling disagreements:
1. Study design disputes (quantitative vs. qualitative): If ≥2 extractors agree on study design classification, use that decision. If all 3 disagree on whether a mixed-methods study has a useable qualitative component, mark CONFLICT — a human decision is needed.
2. Do NOT exclude a study solely because it uses multiple methods. Only exclude if the qualitative component is absent or contributes no primary data (themes, quotes).
3. For the `qualitative_method` field in the reconciled output: use the most specific label agreed upon by ≥2 extractors; if all 3 differ, use the broadest applicable term and note the disagreement.
```

**Reconciled CSV columns:**
`id, title, final_decision, qualitative_method, vote_count, conflict_flag, reconciliation_notes`

---

## Step 3: Write Output

Write/append reconciled results to `05_screening/screened_results.csv`.
Update `05_screening/screening_progress.yaml` if batching.

---

## Step 4: Full Review (final batch only)

Initialize `review_iteration = 0`. Dispatch Full Review per `references/reviewer-protocol.md`.

**Review criteria for qualitative synthesis screening (replace standard criteria):**
- Are qualitative study designs correctly identified? No quantitative-only study should appear as INCLUDE.
- Are mixed-methods studies treated as UNCERTAIN (not excluded) when a qualitative component is present?
- Is the `qualitative_method` field populated for all INCLUDE and UNCERTAIN rows?
- Are PICo components applied consistently — especially the phenomenon of interest and context boundaries?
- Does the conflict rate suggest the criteria.md needs clarification about acceptable qualitative designs?

Follow the same REVISE/REJECT iteration loop per `references/reviewer-protocol.md`.

---

## Step 5: Update project.yaml + Manual Handoff

Set `stages.abstract_screening: completed`.

Print:

```
✅ Abstract screening complete (qualitative synthesis). Results saved to 05_screening/screened_results.csv

📊 Summary (for PRISMA flow diagram):
- Records identified (pre-dedup): [N] — see deduplication_report.md
- Duplicates removed: [N]
- Records screened: [N_post_dedup]
- Included: [N]
- Excluded: [N] (breakdown: [N] study design / [N] population / [N] phenomenon / [N] context)
- Conflicts for human review: [N]
- Inter-rater agreement: [kappa]

Qualitative methods in included studies:
[Tabulate qualitative_method field: method → count]

📋 MANUAL STEP REQUIRED:
1. Review CONFLICT items in screened_results.csv
2. Pull full-text PDFs for all INCLUDED articles
3. Place PDFs in: 06_data_extraction/full_texts/

When you're done, run /data-extraction to continue.
```
