---
node_id: "pub-med-search"
title: "PubMed Search"
description: "Search PubMed and retrieve full biomedical article data with abstracts from NCBI E-utilities."
category: "Healthcare & Life Sciences"
subcategory: "PubMed"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - pubmed
  - ncbi
  - biomedical
  - literature
  - research
  - e-utilities
  - health-sciences
related_nodes:
  - clinical-trials-search
  - science-direct-search
  - scientific-web-of-science
  - scopus-search
---

<!-- SECTION: header -->
# PubMed Search

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Search PubMed and retrieve full biomedical article data with abstracts from the NCBI E-utilities API. The node returns normalized article records including identifiers, titles, abstracts, authors, publication dates, and journal information.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **PubMed Search** node queries the NCBI PubMed database through the E-utilities API and retrieves biomedical literature matching a search term. It uses a two-step process: first fetching article identifiers (esearch), then retrieving full article data including abstracts (esummary + efetch).

The node processes results into a consistent workflow-friendly structure, enabling seamless integration into research pipelines and literature review automation.

### Key Features

- **PubMed Article Search:** Search by keyword, MeSH terms, or advanced queries
- **Full Article Retrieval:** Get article metadata, abstracts, and citation information
- **Configurable Result Limits:** Request between 1 and 100+ articles
- **Structured Article Data:** Returns PMID, title, abstract, authors, journal, publication date, and DOI
- **E-utilities API:** Direct access to NCBI's powerful search and retrieval tools
- **Author Parsing:** Normalized author name and affiliation information
- **Retry Handling:** Retries temporary network errors and rate limits
- **Timeout Protection:** Uses appropriate request timeouts to prevent hanging
- **No Authentication Required:** Uses the public NCBI E-utilities API

### Returned Article Information

Each processed article can contain:

- PMID (PubMed Identifier)
- Article title
- Abstract text
- Author names and affiliations
- Journal name and ISSN
- Publication date
- Publication year
- Article language
- DOI (Digital Object Identifier)
- Keywords and MeSH terms

### Use Cases

- Search biomedical literature by topic or condition
- Build literature review datasets
- Retrieve article abstracts for analysis
- Create research dashboards and catalogs
- Enrich clinical workflows with relevant literature
- Monitor research trends in a specific field
- Automate citation and reference collection
- Support systematic reviews and meta-analyses

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `searchTerm` | `string` | ✅ Yes | — | Search query for PubMed. Can include keywords, MeSH terms, author names, or advanced search syntax. |
| `maxResults` | `number` | ❌ No | `8` | Maximum number of articles to retrieve. Typically 1 to 100+. |
| `apiKey` | `string` | ❌ No | — | Optional NCBI API key for higher rate limits and priority. |
| `sort` | `enum` | ❌ No | `relevance` | Sorting method for results. Options: `relevance`, `pub_date`, `author`. |
| `year_from` | `number` | ❌ No | — | Filter to articles published from this year onwards. |
| `year_to` | `number` | ❌ No | — | Filter to articles published up to this year. |
| `language` | `string` | ❌ No | `eng` | Language code filter (e.g., `eng` for English). |
| `retmax` | `number` | ❌ No | `maxResults` | Maximum results per fetch request (E-utilities parameter). |
| `retstart` | `number` | ❌ No | `0` | Starting index for result pagination. |

### Search Query Examples

| Query | Description |
|-------|-------------|
| `diabetes` | Simple keyword search |
| `BRCA1[GENE] AND cancer` | Gene-specific search with condition |
| `covid-19[MeSH Terms]` | MeSH term search |
| `Smith J[AUTH]` | Author-based search |
| `"artificial intelligence" AND healthcare` | Phrase and Boolean search |

### Default Values

| Parameter | Default |
|-----------|---------|
| `maxResults` | `8` |
| `sort` | `relevance` |
| `language` | `eng` |
| `retstart` | `0` |

### Request Behavior

The node:

- sends requests to the NCBI E-utilities API (esearch and efetch endpoints);
- requests XML or JSON output;
- uses appropriate request timeouts;
- retries temporary errors up to three times;
- applies exponential backoff between retries;
- respects NCBI rate limits;
- processes results into normalized article records;
- supports pagination for large result sets.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Search term string or object containing search parameters such as `searchTerm`, `maxResults`, `year_from`, or `year_to` |

The node primarily uses the configured `searchTerm` and `maxResults` values. Workflow input can override these parameters.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `searchTerm` | `string` | The search query used |
| `total_results` | `number` | Total number of matching articles in PubMed |
| `returned_results` | `number` | Number of articles returned by the node |
| `articles` | `array` | Processed article records |

### Article Structure

Each item in `articles` contains:

| Field | Type | Description |
|-------|------|-------------|
| `pmid` | `string` | PubMed Identifier (unique article ID) |
| `title` | `string` | Article title |
| `abstract` | `string` | Article abstract (when available) |
| `authors` | `array` | Author list with names and affiliations |
| `journal` | `string` | Journal name |
| `journal_issn` | `string` | Journal ISSN |
| `pub_date` | `string` | Publication date |
| `pub_year` | `number` | Publication year |
| `doi` | `string` | Digital Object Identifier |
| `language` | `string` | Article language code |
| `mesh_terms` | `array` | MeSH indexing terms |
| `keywords` | `array` | Article keywords |
| `url` | `string` | Direct PubMed Central or PubMed URL |

### Author Structure

Each author in the `authors` array contains:

| Field | Type | Description |
|-------|------|-------------|
| `name` | `string` | Author full name |
| `initials` | `string` | Author initials |
| `affiliation` | `string` | Institutional affiliation |

### Successful Response Example

```json
{
  "searchTerm": "diabetes",
  "total_results": 1250000,
  "returned_results": 8,
  "articles": [
    {
      "pmid": "37234567",
      "title": "Novel Treatment Approaches for Type 2 Diabetes",
      "abstract": "This study investigates innovative therapeutic strategies...",
      "authors": [
        {
          "name": "Smith, John",
          "initials": "J",
          "affiliation": "Department of Medicine, University Hospital"
        }
      ],
      "journal": "Journal of Clinical Endocrinology",
      "journal_issn": "0021-972X",
      "pub_date": "2026-08-15",
      "pub_year": 2026,
      "doi": "10.1210/clinem.12345",
      "language": "eng",
      "mesh_terms": ["Diabetes Mellitus, Type 2", "Therapeutics"],
      "keywords": ["diabetes", "treatment", "novel therapy"],
      "url": "https://pubmed.ncbi.nlm.nih.gov/37234567/"
    }
  ]
}
```

### Error Response Example

```json
{
  "success": false,
  "error": "PubMed search request failed or returned no results",
  "searchTerm": "diabetes"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Basic Keyword Search

```text
searchTerm: "diabetes"
maxResults: 8
```

**Result:**

```json
{
  "searchTerm": "diabetes",
  "total_results": 1250000,
  "returned_results": 8,
  "articles": [
    {
      "pmid": "37234567",
      "title": "Novel Treatment Approaches for Type 2 Diabetes",
      "abstract": "This study investigates...",
      "pub_year": 2026
    }
  ]
}
```

### Example: Advanced Search with Filters

```text
searchTerm: "BRCA1[GENE] AND cancer"
maxResults: 15
year_from: 2020
sort: "pub_date"
```

### Example: Literature Review Workflow

Use the node to systematically search for recent articles on a research topic, retrieve abstracts and metadata, then pass results to analysis, storage, or dashboard steps for literature review automation.

```text
searchTerm: "artificial intelligence healthcare"
maxResults: 20
year_from: 2024
```

<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store NCBI API credentials in Fusion's credential system. Do not place your API key directly in workflow parameters or exported examples. The node works with the public NCBI E-utilities API, but using an API key increases your rate limit quota.
<!-- /SECTION: security -->
