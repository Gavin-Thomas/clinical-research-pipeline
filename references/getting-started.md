# Getting Started with the Clinical Research Pipeline

**Target audience:** Academic researchers — graduate students, clinical research coordinators, junior clinician-scientists, and PIs familiar with systematic review methodology, PICO frameworks, and biomedical database searching.

---

## What This Pipeline Does

This plugin coordinates a team of AI agents to execute the full evidence synthesis workflow — from scoping the literature to producing a submission-ready manuscript. It is designed to automate everything that does not require physical access to paywalled databases or your institutional credentialing.

**Methodological capabilities:**
- PRISMA 2020–compliant systematic reviews and meta-analyses
- PRISMA-ScR–compliant scoping reviews
- ENTREQ-compliant qualitative evidence synthesis (thematic synthesis, meta-ethnography, framework synthesis, meta-aggregation; CERQual confidence assessment)
- STARD 2015–compliant diagnostic test accuracy (DTA) reviews (QUADAS-2 appraisal, bivariate model/HSROC synthesis, SROC curves, sensitivity/specificity forest plots)
- STROBE-compliant original research write-ups and CARE-compliant case reports
- Random-effects meta-analytic pooling (DerSimonian-Laird, REML) with heterogeneity metrics (I², Q, tau², prediction intervals)
- GRADE and GRADE-DTA certainty-of-evidence assessment and Summary of Findings tables
- Dual-independent data extraction with consensus resolution
- Risk-of-bias appraisal (RoB 2 for RCTs, ROBINS-I for non-randomized studies, NOS, JBI checklists, CASP Qualitative, QUADAS-2 for diagnostic accuracy studies)
- Executable Python/R analysis scripts (forest plots, funnel plots, bivariate/HSROC DTA analysis, subgroup analyses, sensitivity analyses)
- Multi-database search construction: PubMed/MEDLINE, Ovid MEDLINE, Embase, CINAHL (with MeSH/Emtree/CINAHL headings, Boolean logic, field tags)
- API-assisted automation: PubMed E-utilities (search yield testing, volume estimation), Semantic Scholar (semantic search, citation snowballing), OpenAlex (cross-database volume checks), Unpaywall (open-access PDF auto-discovery)
- Autonomous refinement loops: search strategies auto-tested and adjusted for yield; manuscript revision cycles driven by measurable quality metrics with convergence tracking

---

## The AI Agent Team

Each stage is executed by specialized subagents:

| Role | Responsibilities |
|------|-----------------|
| **Librarian** | Literature landscape scanning, MeSH/Emtree search construction, database selection |
| **Principal Investigator (PI)** | PICO/PEO/PCC formulation, clinical significance assessment, scope decisions |
| **Methodologist** | Eligibility criteria development, reporting guideline compliance (PRISMA/STROBE/CARE), bias tool selection |
| **Data Extractor (×3)** | Triple-independent abstract screening and full-text data extraction with consensus |
| **Statistician** | Extraction template design, meta-analytic pooling, heterogeneity analysis, GRADE, executable analysis scripts |
| **Programmer** | Code review and validation of all Python/R analysis scripts — catches bugs, verifies statistical method implementation, ensures scripts run end-to-end |
| **Manuscript Writer (×2)** | Dual-independent IMRAD drafting, journal-specific formatting |
| **Proofreader** | Independent linguistic and formatting quality assurance — grammar, tense, abbreviations, citation integrity, reporting guideline checklist, AI-writing artifact detection |
| **Independent Reviewer (×2+1)** | Two-tier quality review (Quick Review for stages 1–4, Full Review for stages 5–7), with a tie-breaking third reviewer on disagreement |

---

## AI vs. Human Decisions

**The pipeline handles autonomously:**
- Research question refinement (PICO/PEO components, scope boundaries)
- MeSH term identification, synonym expansion, Boolean logic construction
- Abstract screening against your eligibility criteria
- Data extraction from full-text PDFs you provide
- Risk-of-bias appraisal for each included study
- Statistical synthesis and generation of analysis code
- GRADE certainty assessment
- Manuscript drafting and journal formatting
- Self-review and revision cycles (up to 3 rounds before escalating to you)

**You must do these steps (physically impossible for the AI):**
1. **Run the database searches** — PubMed, Ovid, Embase require authenticated sessions. The pipeline builds the search strings and auto-tests PubMed yield via E-utilities API; you execute and export them.
2. **Retrieve full-text PDFs** — the pipeline auto-checks Unpaywall for open-access PDFs (typically finds 30–60%), so you only need to retrieve the remaining paywalled articles via institutional access or ILL.

**You should verify (AI assists, human judges):**
- Clinical accuracy of the research question and PICO boundaries
- Inclusion/exclusion decisions on edge cases the AI flags as UNCERTAIN
- Extracted data for key outcomes (especially safety data and primary endpoints) against source papers
- All claims in the manuscript that require domain expertise beyond the extracted data

---

## Pipeline Stages at a Glance

| Stage | Agents | Output | Manual? |
|-------|--------|--------|---------|
| 1. Literature Search | Librarian + PI | `01_literature_search/landscape_report.md` | No |
| 2. Research Question | PI + Methodologist | `02_research_question/research_question.md` | No |
| 3. Inclusion/Exclusion | Methodologist + PI | `03_inclusion_exclusion/criteria.md` | No |
| 4. Database Search Build | Librarian | `04_database_search/search_strategy.md` | **YES — you run searches** |
| 5. Abstract Screening | Data Extractors (×3) | `05_screening/screened_results.csv` | **YES — you retrieve PDFs** |
| 6. Data Extraction | Data Extractors (×3) + Statistician | `06_data_extraction/` | No |
| 7. Manuscript Writing | Writers (×2) + Statistician | `07_manuscript/manuscript.md` | No |

---

## Manual Handoff 1: Executing Database Searches

After Stage 4, you receive complete, copy-paste-ready search strings. Execute them and export results in one of these formats:

### PubMed/MEDLINE
- **Recommended export:** `Send to → Citation manager → Format: PubMed → Create file` → saves as `.nbib`
- **Alternative:** `Save → Format: PubMed → Create file` → saves as `.txt` (PubMed format)
- Place in: `04_database_search/abstracts/`
- **Tip:** PubMed search syntax uses `[MeSH Terms]`, `[tiab]` (title/abstract), and `[pt]` (publication type) field tags. The pipeline's search strings include all required tags.

### Ovid MEDLINE / Embase
- **Export:** `Export → Format: RIS` or `Format: Direct Export` → saves as `.ris`
- Place in: `04_database_search/abstracts/`
- **Tip:** Ovid uses `.mp.` (multi-purpose) and `/exp` (explosion) syntax; Embase uses Emtree. The pipeline generates database-specific strings for each.

### Deduplication
The pipeline handles deduplication programmatically from your exported files. However, if you prefer to manage records in a reference manager:
- **Rayyan:** Import all `.ris`/`.nbib` files → use "Duplicates" feature before exporting for screening
- **Zotero:** Import all files → Select All → Duplicates pane → Merge; export merged library as `.ris`
- **Covidence:** Upload `.ris` files directly — Covidence deduplicates automatically

If you deduplicate manually, export the deduplicated set as `.ris` or `.csv` and place in `04_database_search/abstracts/`. Note the pre- and post-deduplication counts for your PRISMA flow diagram.

---

## Manual Handoff 2: Retrieving Full-Text PDFs

After Stage 5, `05_screening/screened_results.csv` lists all included articles with authors, year, title, DOI, and PubMed ID.

**Step 1: Download auto-discovered OA PDFs (done for you)**
The pipeline automatically queries the Unpaywall API for every included study with a DOI and writes results to `05_screening/oa_pdf_links.csv`. Typically 30–60% of articles are freely available. Download these first — no institutional access needed.

**Step 2: Retrieve remaining paywalled articles (by priority):**
1. **PubMed Central** — `https://www.ncbi.nlm.nih.gov/pmc/` — free, high coverage for NIH-funded research
2. **Institutional library access** — authenticate via your institution's proxy/VPN; most library portals allow DOI-based search
3. **Interlibrary loan (ILL)** — for articles unavailable through the above; typically 24–72 hours
4. **Author contact** — email corresponding author directly; most respond within days

**Naming convention:** Use the pipeline's expected format: `[FirstAuthor]_[Year]_[keyword].pdf` (e.g., `Smith_2022_dupilumab_AD.pdf`)

**Place all PDFs in:** `06_data_extraction/full_texts/`

**Unavailable articles:** Tell the pipeline which PMIDs you couldn't retrieve. These are recorded as "full text not retrieved" with a documented reason — standard PRISMA practice, not a methodological failure.

---

## Review Type Decision Guide

| Question | Your situation → Recommended type |
|----------|----------------------------------|
| Is your question answerable with a focused PICO and existing RCT/cohort evidence? | **Systematic review** (± meta-analysis if data are poolable) |
| Do you want to statistically pool effect estimates across studies? | **Meta-analysis** (embedded in systematic review or standalone) |
| Is your question broad, conceptual, or mapping-focused (PCC framework)? | **Scoping review** (PRISMA-ScR; does not assess quality or synthesize quantitatively) |
| Do you need a rapid answer (4–8 weeks) with limited resource? | **Rapid review** (streamlined SR; explicitly document methodological shortcuts) |
| Are you collecting primary data (surveys, chart review, prospective cohort)? | **Original research** (STROBE for observational; CONSORT if RCT) |
| Are you reporting a single patient with teaching value? | **Case report** (CARE guideline; `case_report`) |
| Are you reporting 3–20 patients sharing a diagnosis, exposure, or outcome? | **Case series** (CARE guideline + cross-case comparison; `case_series`) |
| Is your question about patient experience, perspectives, or acceptability? | **Qualitative synthesis** (ENTREQ; synthesises qualitative primary research — thematic synthesis, meta-ethnography, framework synthesis) |
| Is your question "how accurately does this test identify the condition?" (sensitivity, specificity, AUC)? | **Diagnostic test accuracy review** (STARD; QUADAS-2 appraisal; bivariate/HSROC synthesis; SROC curves) |

**When to choose SR vs. scoping:** If your primary goal is answering "does X work?" with a quantitative estimate → systematic review. If your goal is "what is known about X?" or "what research exists?" → scoping review. Journals increasingly require pre-registration (PROSPERO) for systematic reviews; scoping reviews are more flexible.

**When to choose qualitative synthesis:** If your research question starts with "what are patients' experiences of...", "what are the barriers to...", or "how do patients perceive...", you need a qualitative synthesis rather than a systematic review. These reviews synthesise interview studies, focus groups, and ethnographic observations — not RCTs or cohort studies. Use the PICo framework (Population, phenomenon of Interest, Context). ENTREQ is the reporting guideline; CERQual (the qualitative equivalent of GRADE) rates confidence in synthesised findings. Common in patient-centred research, health services, and topics where lived experience matters as much as clinical efficacy.

**Qualitative synthesis methods:**
- **Thematic synthesis** (Thomas & Harden) — most common; generates descriptive and then analytical themes; works well for heterogeneous qualitative evidence
- **Meta-ethnography** (Noblit & Hare) — translates concepts across interpretive studies; better for smaller evidence bases with rich theoretical content
- **Framework synthesis** — applies a pre-specified conceptual framework; useful when theory-driven or policy-oriented
- **Meta-aggregation** (JBI) — aggregates findings into categories; used when evidence is more descriptive

**Network meta-analysis:** If you have multiple interventions to compare indirectly (multiple treatments vs. common comparator), inform the pipeline at setup. The Statistician will adapt to NMA methods (using `netmeta` in R or equivalent).

---

## Tips for Better Pipeline Outputs

**Before starting:**
- Have your PICO or PEO components defined — the pipeline refines them but starts faster with specifics
- Know your target journal — word limits and reporting requirements shape the manuscript from Stage 1
- Specify your databases — if you have Ovid access, say so; the pipeline will generate Ovid-specific syntax

**At project setup, specify if you want:**
- Pre-specified subgroup analyses (e.g., "by age group", "by disease severity") — informs the extraction template
- Sensitivity analyses (e.g., "excluding high risk-of-bias studies") — planned at Stage 3, executed at Stage 6
- GRADE certainty assessment — standard for systematic reviews; the Statistician generates SoF tables
- Network meta-analysis — requires different extraction template design

**PROSPERO registration:** For systematic reviews intended for high-impact journals, register your protocol on PROSPERO before Stage 4 (database searching). The pipeline produces a registerable protocol after Stage 3. Ask the pipeline to "format the protocol for PROSPERO submission" after Stage 3 completes.

---

## Starting the Pipeline

Tell Claude what you want to do using methodological terms:

> "Run the pipeline on [topic] as a PRISMA systematic review targeting [journal]"

> "Set up a meta-analysis on [intervention] for [population], PICO: P=[...] I=[...] C=[...] O=[...]"

> "I need to do a scoping review on [topic] — help me structure the protocol"

> "Help me run a rapid review on [topic] for a [journal] submission deadline in [month]"

> "Start a systematic review protocol for PROSPERO registration on [topic]"

> "I want to do a qualitative synthesis on [topic] — patients' experiences of [condition/treatment]"

> "Run a thematic synthesis of qualitative studies on [topic]"

Claude will confirm your project parameters and create the project directory structure automatically.

---

## Frequently Asked Questions

**Can I resume a project I started in a previous session?**
Yes. Open Claude Code in your project directory and say "resume the pipeline" or "continue my systematic review." The pipeline reads `project.yaml` to determine where to pick up.

**Can I override the AI's decisions at any stage?**
Yes. After any stage completes, you can review the output and instruct Claude to revise specific elements before proceeding. The pipeline also has built-in review cycles where agents self-correct; you can inject guidance at escalation points.

**How accurate is triple-independent data extraction?**
Triple-independent extraction with consensus reduces random extraction error. However, for primary endpoints and safety outcomes in your key studies, verify extracted values against source tables — especially when these drive your pooled estimates. The pipeline flags extraction discrepancies in the consensus report.

**Does the AI handle statistical heterogeneity decisions?**
The Statistician reports all heterogeneity metrics (I², Q, tau², prediction interval) and applies pre-specified decision rules (e.g., I² > 75% → consider not pooling). Final decisions about whether to pool, use narrative synthesis, or conduct subgroup analyses remain yours as PI; the pipeline provides the quantitative basis for that judgment.
