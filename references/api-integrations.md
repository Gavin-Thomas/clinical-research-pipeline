# API Integrations Reference

This file documents the external APIs the pipeline uses for programmatic literature search, abstract retrieval, citation analysis, and open-access PDF discovery. All APIs are free and do not require authentication (though some benefit from an API key for higher rate limits).

---

## PubMed E-utilities API

**Base URL:** `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/`

**Used in:** Stages 1, 4, 5

**Purpose:** Programmatic PubMed search and abstract download. Replaces manual copy-paste searching.

### Key Endpoints

**1. ESearch — Search and get PMIDs**

```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi
  ?db=pubmed
  &term={URL-encoded search query}
  &retmax=500
  &retmode=json
  &sort=relevance
  &datetype=pdat
  &mindate={YYYY/MM/DD}
  &maxdate={YYYY/MM/DD}
```

Returns: JSON with `esearchresult.idlist` (array of PMIDs) and `esearchresult.count` (total hits).

Use `retmax=0` first to get the count without downloading IDs (for yield estimation in the autonomous refinement loop).

**2. EFetch — Download abstracts by PMID**

```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi
  ?db=pubmed
  &id={comma-separated PMIDs}
  &rettype=abstract
  &retmode=xml
```

Returns: PubMed XML with full abstract text, MeSH terms, authors, DOI, journal, year.

Batch limit: 200 PMIDs per request. For larger sets, paginate with `retstart`.

**3. ELink — Find related articles / cited-by**

```
GET https://eutils.ncbi.nlm.nih.gov/entrez/eutils/elink.fcgi
  ?dbfrom=pubmed
  &db=pubmed
  &id={PMID}
  &cmd=neighbor_score
  &retmode=json
```

Returns: Related PMIDs ranked by relevance score. Useful for snowball searching in Stage 1.

### Rate Limits

- Without API key: 3 requests/second
- With API key: 10 requests/second
- Get a free key at: https://www.ncbi.nlm.nih.gov/account/settings/ → API Key
- Pass key as `&api_key={key}` parameter

### Pipeline Usage Pattern

```python
# Stage 4: Test search yield before manual handoff
import urllib.parse

query = '"artificial intelligence"[MeSH] AND "dermatology"[MeSH]'
encoded = urllib.parse.quote(query)
count_url = f"https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term={encoded}&retmax=0&retmode=json"
# WebFetch this URL → parse JSON → read esearchresult.count

# Stage 5: Programmatic abstract download (supplements manual export)
fetch_url = f"https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id={pmid_list}&rettype=abstract&retmode=xml"
# WebFetch → parse XML → extract title, abstract, authors, DOI, MeSH terms
```

---

## Semantic Scholar API

**Base URL:** `https://api.semanticscholar.org/graph/v1/`

**Used in:** Stage 1

**Purpose:** Embedding-based semantic search, citation graph traversal, bulk metadata retrieval. Complements PubMed by finding conceptually related papers that keyword search misses.

### Key Endpoints

**1. Paper Search — Semantic keyword search**

```
GET https://api.semanticscholar.org/graph/v1/paper/search
  ?query={natural language query}
  &limit=100
  &offset=0
  &fields=paperId,title,abstract,year,citationCount,authors,externalIds,tldr
  &year={YYYY}-{YYYY}
  &fieldsOfStudy=Medicine
```

Returns: Papers ranked by semantic relevance (not just keyword match). The `tldr` field provides an AI-generated one-line summary.

**2. Paper Details — Get full metadata by ID**

```
GET https://api.semanticscholar.org/graph/v1/paper/{paperId}
  ?fields=paperId,title,abstract,year,citationCount,referenceCount,authors,externalIds,citations,references,tldr
```

Accepts: Semantic Scholar ID, DOI (`DOI:10.xxxx/xxxx`), PMID (`PMID:12345678`), or ArXiv ID.

**3. Paper Citations — Who cited this paper?**

```
GET https://api.semanticscholar.org/graph/v1/paper/{paperId}/citations
  ?fields=paperId,title,abstract,year,citationCount,authors
  &limit=100
```

**4. Paper References — What does this paper cite?**

```
GET https://api.semanticscholar.org/graph/v1/paper/{paperId}/references
  ?fields=paperId,title,abstract,year,citationCount,authors
  &limit=100
```

**5. Bulk Paper Lookup — Multiple papers at once**

```
POST https://api.semanticscholar.org/graph/v1/paper/batch
  ?fields=paperId,title,abstract,year,citationCount,authors,externalIds
Body: {"ids": ["DOI:10.xxxx/xxxx", "PMID:12345678", ...]}
```

Batch limit: 500 papers per request.

### Rate Limits

- Without API key: 100 requests/5 minutes
- With API key: 1 request/second sustained
- Get a free key at: https://www.semanticscholar.org/product/api#api-key

### Pipeline Usage Pattern

```
# Stage 1: Find semantically related papers the Librarian's WebSearch might miss
query = "AI dermatology diagnostic accuracy melanoma detection"
url = f"https://api.semanticscholar.org/graph/v1/paper/search?query={query}&limit=50&fields=paperId,title,abstract,year,citationCount,authors,externalIds,tldr&fieldsOfStudy=Medicine"

# Stage 1: Citation snowballing — find papers citing a key review
url = f"https://api.semanticscholar.org/graph/v1/paper/DOI:10.1001/jamadermatol.2023.0001/citations?fields=title,abstract,year,citationCount&limit=100"
```

---

## OpenAlex API

**Base URL:** `https://api.openalex.org/`

**Used in:** Stages 1, 4

**Purpose:** Open metadata for 250M+ works. Best for: volume estimation, concept mapping, institutional filtering, and discovering grey literature not indexed in PubMed.

### Key Endpoints

**1. Works Search — Filter and search works**

```
GET https://api.openalex.org/works
  ?search={query}
  &filter=publication_year:2020-2026,type:article,concepts.id:C71924100
  &sort=cited_by_count:desc
  &per_page=50
  &select=id,doi,title,publication_year,cited_by_count,authorships,concepts,open_access
```

**2. Concepts — Map topic to OpenAlex concept IDs**

```
GET https://api.openalex.org/concepts
  ?search=dermatology
```

Returns concept IDs (e.g., `C71924100` for Dermatology) that can filter works searches.

**3. Works Count — Quick volume estimation**

```
GET https://api.openalex.org/works
  ?filter=default.search:{query},publication_year:2020-2026
  &per_page=1
```

Read `meta.count` from the response for total matching works.

### Rate Limits

- Unauthenticated: 10 requests/second, max 100,000/day
- With polite pool (add `mailto=your@email.com`): same rate, prioritized
- No API key needed

### Pipeline Usage Pattern

```
# Stage 1: Quick volume check — how much literature exists?
url = "https://api.openalex.org/works?filter=default.search:AI%20dermatology%20diagnosis,publication_year:2020-2026&per_page=1"
# Read meta.count → report "Approximately N works identified in OpenAlex"

# Stage 4: Validate search yield against OpenAlex count
# If PubMed yields << OpenAlex count, the search may be too restrictive
```

---

## Unpaywall API

**Base URL:** `https://api.unpaywall.org/v2/`

**Used in:** Stage 5 (post-screening PDF retrieval)

**Purpose:** Finds legal open-access PDF URLs for papers by DOI. Checks publisher websites, institutional repositories, preprint servers, and PubMed Central.

### Endpoint

```
GET https://api.unpaywall.org/v2/{DOI}?email={your_email}
```

Returns JSON with:
- `is_oa`: boolean — is an OA version available?
- `best_oa_location.url_for_pdf`: direct PDF URL (if available)
- `best_oa_location.url_for_landing_page`: landing page URL
- `best_oa_location.host_type`: "publisher" | "repository"
- `oa_locations[]`: all available OA locations ranked by quality

### Rate Limits

- 100,000 requests/day
- Requires email address (no API key)
- Polite: add 100ms delay between requests

### Pipeline Usage Pattern

```
# After Stage 5 screening, for each included study with a DOI:
doi = "10.1001/jamadermatol.2023.0001"
url = f"https://api.unpaywall.org/v2/{doi}?email=researcher@university.edu"
# WebFetch → check is_oa → if true, extract best_oa_location.url_for_pdf
# Save PDF URL to 05_screening/oa_pdf_links.csv for user convenience
```

---

## Novelty Check Pattern (Stage 2)

Used in Step 3a of the Research Question skill to verify the proposed question hasn't already been answered by an existing or in-progress review. Searches 5 sources in parallel.

```
1. PubMed E-utilities:
   - Build query: {PICO terms} AND (systematic review[pt] OR meta-analysis[pt])
   - Filter: last 3 years (datetype=pdat)
   - WebFetch ESearch (retmax=20) → WebFetch EFetch for titles/abstracts

2. Semantic Scholar:
   - Natural language query from research question
   - Filter to Medicine, last 3 years
   - WebFetch paper/search endpoint → filter to review-type papers

3. PROSPERO:
   - WebSearch: site:crd.york.ac.uk/prospero {PICO terms}
   - Check for "Ongoing" registrations — these indicate someone is already doing this review

4. Cochrane Library:
   - WebSearch: site:cochranelibrary.com {PICO terms} review
   - Cochrane protocols are near-certain to be published

5. WebSearch (catch-all):
   - General search for preprints and non-indexed reviews

Score each result 0–4 based on PICO component overlap.
BLOCK if score ≥3.5 or if ongoing PROSPERO/Cochrane with ≥2.5.
Write results to 02_research_question/novelty_check.md.
```

---

## API Integration Patterns

### Autonomous Search Refinement Loop (autoresearch-inspired)

Used in Stage 4 to automatically test and refine search strategies before asking the user to execute them manually.

```
LOOP (max 3 iterations):
  1. TRANSLATE: Convert the Boolean search strategy into a PubMed E-utilities query
  2. TEST: WebFetch the ESearch URL with retmax=0 to get the result count
  3. EVALUATE:
     - If count == 0: TOO RESTRICTIVE → loosen (remove one concept block, broaden date range, add synonyms)
     - If count > 500: TOO BROAD → tighten (add study design filter, narrow population, restrict date)
     - If 10 ≤ count ≤ 500: ACCEPTABLE → stop loop, proceed
     - If 1 ≤ count < 10: POSSIBLY TOO NARROW → warn but proceed (small evidence base)
  4. ADJUST: Modify the search strategy based on evaluation
  5. LOG: Record iteration number, query, count, and adjustment made
```

### Citation Snowballing (Stage 1)

After the Librarian identifies key landmark papers:

```
1. Look up each landmark paper on Semantic Scholar by DOI
2. Fetch its citations (papers citing it) and references (papers it cites)
3. Filter by year range and fieldsOfStudy=Medicine
4. Identify any highly-cited papers not found in the initial WebSearch
5. Add to the landscape report under "## Citation Snowball Findings"
```

### OA PDF Auto-Discovery (Stage 5 post-screening)

After screening completes, automatically check Unpaywall for every included study:

```
1. Read 05_screening/screened_results.csv — filter to INCLUDE decisions
2. For each row with a DOI, WebFetch the Unpaywall API
3. Compile results into 05_screening/oa_pdf_links.csv:
   record_id, doi, is_oa, pdf_url, host_type, landing_page_url
4. Print summary: "Found OA PDFs for N/M included studies"
5. User only needs to manually retrieve the remaining paywalled PDFs
```
