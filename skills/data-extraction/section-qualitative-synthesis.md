# Data Extraction — qualitative_synthesis

Qualitative synthesis uses a non-quantitative workflow. The synthesis method is determined by
`review_config.synthesis_approach` in project.yaml: `thematic` (default), `framework`,
`meta_ethnography`, or `meta_aggregation`.

## Step 1: Design Qualitative Extraction Template

Dispatch Methodologist to create a template capturing:
- Study identification (ID, author, year, country, setting)
- Study design (phenomenological, grounded theory, ethnographic, thematic analysis, action research, mixed-methods)
- Participant characteristics (n, age range, condition, sampling method, data saturation reported Y/N)
- Data collection method (semi-structured interview, focus group, observation, documents, diaries)
- Analytic method and theoretical framework reported by authors
- Key themes/findings as reported by authors
- Representative verbatim participant quotes per theme (exact text — preserve original wording)
- Author interpretations and conceptual conclusions (separate field from quotes)
- Reflexivity statement from authors (if reported)
- CASP Qualitative Checklist scores (10 items, scored 0/1)

Write to `06_data_extraction/qualitative_extraction_template.md`.

## Step 2: Extract Qualitative Data (3 Parallel Extractors)

Partition papers and dispatch 3 Data Extractor agents in parallel.

**Data Extractor agent prompt (qualitative):**

```
[Insert Data Extractor system prompt from agent-roles.md]

TASK: Extract qualitative data from the assigned papers.

TEMPLATE:
[Insert qualitative_extraction_template.md]

PAPERS:
[List of assigned PDFs]

For each paper:
1. Read the full text
2. Identify all major themes or findings reported by the authors
3. For each theme: copy ≥1 verbatim participant quote — exact wording in quotation marks
   with page number; then record the author's interpretation of that theme as a separate field
4. Complete the CASP Qualitative Checklist (all 10 items, scored 0=no/unclear, 1=yes)
5. Do NOT paraphrase participant quotes — verbatim accuracy is essential for meta-synthesis

SELF-CHECK before outputting:
1. Every assigned paper has an extraction entry (none skipped)
2. Every theme has ≥1 verbatim participant quote with a page number
3. Author interpretations are in a separate field from participant quotes
4. CASP checklist fully completed (all 10 items scored)
5. No paraphrasing of participant quotes

OUTPUT: Structured extraction entries per template.
```

Merge all extractions into `06_data_extraction/extracted_quotes.csv` with columns:
`study_id, author, year, design, theme_label, participant_quote, author_interpretation, casp_score, source_page`

## Step 3: CASP Quality Appraisal

Dispatch Methodologist to compile all CASP scores into `06_data_extraction/casp_appraisal.md`.
Per JBI qualitative synthesis guidance: do not exclude studies solely on quality grounds —
instead document CASP scores and carry them forward as input to CERQual confidence ratings.
Add a brief narrative on overall quality patterns and any studies with CASP < 5/10.

## Step 4: Thematic Synthesis (Full Review + Retry)

Initialize `review_iteration = 0`. Dispatch Methodologist to perform synthesis per `synthesis_approach`:

**Thematic** (Thomas & Harden — default):
1. Line-by-line coding of verbatim participant quotes across all studies
2. Grouping codes into descriptive themes (what participants said)
3. Generating analytical themes that interpret beyond individual studies (why/how)

**Framework synthesis**:
Apply a pre-specified theoretical or conceptual framework. State the framework name and source.
Map coded data to framework categories and report emergent vs. anticipated themes.

**Meta-ethnography** (Noblit & Hare):
1. Read and re-read studies to identify key concepts and metaphors
2. Determine reciprocal translation (studies illuminate each other) vs. refutational (studies contradict)
3. Produce a line-of-argument synthesis integrating across studies

**Meta-aggregation** (JBI):
1. Extract unambiguous findings (participant quotes + author interpretation as a unit)
2. Aggregate findings into categories based on similarity of meaning
3. Synthesize categories into synthesized findings — as actionable and specific as possible

Write synthesis to `06_data_extraction/synthesis_report.md`. For each analytical theme or
synthesized finding: include supporting quotes from ≥2 studies, source study IDs, and a brief
interpretive narrative explaining the finding.

Dispatch Full Review per reviewer-protocol.md.

On **REVISE**: build REVISION REQUIRED table, re-dispatch Methodologist to address each finding,
increment `review_iteration`, re-dispatch reviewer with revision history.

On **REJECT** or `review_iteration ≥ 2` without APPROVE: trigger full escalation per
reviewer-protocol.md (escalation banner, list all unresolved findings, pause pipeline).

## Step 5: CERQual Assessment

Dispatch Methodologist to complete CERQual (Confidence in the Evidence from Reviews of Qualitative
research) for each synthesized finding. Assess 4 domains:

- **Methodological limitations**: based on CASP scores of contributing studies (weight by contribution)
- **Coherence**: how well the data supports the interpretation (close fit vs. interpretive leap)
- **Adequacy of data**: richness and quantity of contributing studies and participants
- **Relevance**: applicability of contributing studies to the review question and context

Write to `06_data_extraction/cerqual_assessment.md`. Each finding receives an overall CERQual
confidence rating: High / Moderate / Low / Very Low, with rationale per domain.

## Step 6: Update project.yaml

Set `stages.data_extraction: completed`.
