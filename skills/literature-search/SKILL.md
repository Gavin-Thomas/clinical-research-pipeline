---
name: literature-search
description: |
  Use when the user wants to conduct a literature search, survey existing research on a topic,
  identify gaps in the literature, or start the first stage of a systematic/scoping review.
  Trigger on phrases like "literature search", "what's been published on", "survey the literature",
  "find research gaps", "landscape review", or "what do we know about".
---

# Literature Search (Stage 1)

Conduct a preliminary landscape survey of existing literature on a topic using AI agent teams.

## Prerequisites

- A `project.yaml` file must exist in the current project directory (created by `/research-pipeline` or manually)
- Read `project.yaml` to get `topic`, `project_type`, and `databases`

If no `project.yaml` exists, ask the user for the topic and create a minimal one.

## Process

### Step 1: Dispatch Librarian Agent

Dispatch an Agent subagent with the Librarian role. Read `references/agent-roles.md` for the Librarian system prompt.

**Librarian agent prompt:**

```
[Insert Librarian system prompt from agent-roles.md]

TASK: Conduct a preliminary landscape survey on the following topic:

Topic: [from project.yaml or user input]
Project type: [from project.yaml]

Using WebSearch and WebFetch, search for:
1. Recent systematic reviews and meta-analyses on this topic (last 5 years)
2. Key landmark studies
3. Current trends and emerging subtopics
4. Known gaps identified in existing review articles
5. Volume of literature (approximate number of studies)

Search these sources:
- PubMed: site:pubmed.ncbi.nlm.nih.gov [topic keywords]
- Google Scholar: [topic keywords] systematic review OR meta-analysis
- Preprint servers: site:medrxiv.org OR site:biorxiv.org [topic keywords]

IMPORTANT: These are general web searches, not direct database API queries. Results provide a preliminary overview that supplements the PI's domain knowledge.

OUTPUT: Write a structured landscape report in markdown with these sections:
1. Search Summary (sources searched, date, approximate results)
2. Existing Reviews (list with citations)
3. Key Studies (landmark papers)
4. Current Trends
5. Identified Gaps
6. Volume Assessment
7. Preliminary Bibliography

**SELF-CHECK (required before writing output to disk):**

Before producing output, verify each item and correct any issues inline:

Coverage:
- [ ] At least 3 distinct databases/sources searched (PubMed, Google Scholar, preprint server)
- [ ] At least 5 existing reviews or meta-analyses identified, if literature volume supports it
- [ ] Search terms included synonyms and MeSH-equivalent concepts (not just the verbatim topic phrase)

Quality of Identified Gaps:
- [ ] Each gap is specific and falsifiable (not generic like "more research is needed")
- [ ] Each gap is verified against the listed existing reviews — do not flag a gap already addressed by a cited review
- [ ] At least one gap is actionable for the stated project type

Citations:
- [ ] Every paper in the Preliminary Bibliography is referenced in the report body
- [ ] All citations include author, year, title, and journal/source
- [ ] No fabricated citations — if unsure of a citation, mark `[VERIFY]` rather than guessing

If any check fails, correct before finalizing output.
```

### Step 2: Dispatch PI Agent

After the Librarian's output is received, dispatch a PI Agent subagent.

**PI agent prompt:**

```
[Insert PI system prompt from agent-roles.md]

TASK: Review the following landscape survey and provide strategic analysis.

LANDSCAPE SURVEY:
[Insert Librarian's output]

Analyze this landscape and produce:
1. GAP ANALYSIS: Which areas are understudied, outdated, or have conflicting evidence?
2. SATURATION ASSESSMENT: Which areas are well-covered and unlikely to yield novel contributions?
3. OPPORTUNITY RANKING: Rank the top 3-5 research opportunities by novelty, feasibility, and clinical impact
4. TREND ASSESSMENT: What direction is the field moving? Where will it be in 2-3 years?

**SELF-CHECK (required before writing output to disk):**

Before producing your Strategic Analysis, verify each item:

Gap Analysis Quality:
- [ ] Each gap is substantiated by a specific limitation noted in a cited review (not inferred generally)
- [ ] The gap analysis distinguishes between: (a) truly unstudied areas, (b) areas studied with low-quality evidence, (c) areas with conflicting results
- [ ] Saturation assessment does not flag well-covered areas as opportunities

Opportunity Ranking:
- [ ] Each ranked opportunity is distinct from the others (not variations of the same question)
- [ ] Feasibility is assessed relative to the likely project type (e.g., a case report cannot address a systematic-review-scale gap)
- [ ] Clinical impact is grounded in a real unmet need stated in the landscape survey, not hypothetical

Logical Consistency:
- [ ] No opportunity is ranked highly if the saturation assessment classifies that area as well-covered
- [ ] Trend assessment is forward-looking (what will the field need in 2-3 years) not just a restatement of current state

If any check fails, correct before finalizing output.

OUTPUT: Append your analysis to the landscape report under a "## Strategic Analysis" section.
```

### Step 3: Write Output

Combine both agents' outputs into `01_literature_search/landscape_report.md`.

### Step 4: Quick Review

Dispatch a single Independent Reviewer agent per `references/reviewer-protocol.md` Quick Review process.

**Review criteria for this stage:**
- Was the search reasonably comprehensive given web search limitations?
- Are the identified gaps genuine (not already addressed by cited reviews)?
- Is the strategic analysis clinically grounded?
- Are there obvious major studies or reviews that were missed?

**Retry logic (implement inline — do not rely solely on reviewer-protocol.md reference):**

Initialize `review_iteration = 0`.

After the reviewer returns a verdict:
- **APPROVE**: Proceed to Step 5.
- **REVISE** or **REJECT**: Re-dispatch the Librarian agent (and PI agent if the strategic analysis was flagged) with the reviewer's findings table appended to their original prompt as a "REVISION REQUIRED" block. The agents must address each finding explicitly before re-writing their sections to disk. Then re-dispatch the same reviewer with the revised output and `review_iteration += 1`. Include the prior review verdict and a summary of changes made in the reviewer's context.
- If `review_iteration >= 2` and verdict is still REVISE or REJECT: escalate per the Escalation Protocol in `references/reviewer-protocol.md` — print the warning banner, all reviewer findings, and the agent's revision notes, then pause for human PI input.

### Step 5: Update project.yaml

Set `stages.literature_search: completed`.

## Adaptation by Project Type

All project types use this stage identically — it always produces a landscape report.
