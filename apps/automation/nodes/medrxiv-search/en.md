---
node_id: "medrxiv-search"
title: "MedRxiv/BioRxiv Search"
description: "Search recent pre-print articles on medRxiv and bioRxiv by keyword and get titles, authors, abstracts, DOIs and PDF links."
category: "Research / Literature"
subcategory: "Pre-prints"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - medrxiv
  - biorxiv
  - preprints
  - research
  - science
  - health
related_nodes:
  - bio-rxiv
  - pmc
  - rcsbpdb
  - scientific-arxiv
---

<!-- SECTION: overview -->
# MedRxiv/BioRxiv Search

> **Category:** Research / Literature &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Search **pre-print articles** published on **medRxiv** (health sciences) or **bioRxiv** (biology) during the last days, using the public API `https://api.medrxiv.org`. The node downloads the articles of the selected period, keeps those whose **title, abstract or authors** contain your query, and returns them with ready-to-use links (abstract page, full text, PDF).

No API key is required.

### Use Cases

- **Research watch:** Get every new pre-print on a disease, a drug or a gene, and send a weekly digest by email or Slack.
- **Literature review:** Collect recent abstracts on a topic and summarize them with an LLM node.
- **Author tracking:** Follow the new pre-prints of a researcher or a lab (the query also matches author names).
- **Dataset building:** Store titles, DOIs and abstracts in a database or a spreadsheet.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `server` | `enum` | No | `medrxiv` | Pre-print server: `medrxiv` (health sciences) or `biorxiv` (biology). |
| `query` | `string` | Yes | — | Keyword(s) to look for in the title, abstract and authors (max 500 characters). Case-insensitive. |
| `maxResults` | `number` | No | `10` | Maximum number of articles returned (1–100). |
| `intervalDays` | `number` | No | `180` | Search the articles published in the last *N* days (1–365). |

### How the Search Works

1. The node computes the period: from *today − `intervalDays`* to *today*.
2. It downloads **all** articles of the server for this period, 100 per request.
3. It keeps the articles whose title, abstract or authors **contain** the query (simple text match, not a relevance search).
4. It returns the first `maxResults` matches.

> **Performance:** bioRxiv publishes thousands of pre-prints per month. A long period means many API requests and a slow run (up to several minutes). Use a small `intervalDays` (e.g. `7`–`30`) for frequent workflows, especially with `biorxiv`.

> The query is matched as one phrase: `long covid` only finds articles containing exactly "long covid". Use short, specific terms.

### Authentication

No authentication is needed. The API is public. The node retries automatically (up to 3 times) on timeouts, rate limits (429) and server errors (5xx).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` / `object` | Triggers the node. A string input is used as the query when `query` is empty. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Search summary and matching articles. |
| `error` | `Error` | Emitted on empty query or API errors. |

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `server` | `string` | Server searched. |
| `query` | `string` | Query used. |
| `interval_days` | `number` | Period searched, in days. |
| `total_results` | `number` | Number of matching articles in the period. |
| `returned_results` | `number` | Number of articles returned (≤ `maxResults`). |
| `articles[]` | `array` | `title`, `authors`, `doi`, `abstract`, `abstract_url`, `pdf_url`, `full_text_url`, `publication_date`, `category`, `version`, `type`, `license`, `published_journal`, `has_jatsxml`. |

### Output Example

```json
{
  "server": "medrxiv",
  "query": "covid",
  "interval_days": 180,
  "total_results": 214,
  "returned_results": 1,
  "articles": [
    {
      "title": "Long-term outcomes after COVID-19 hospitalisation: a cohort study",
      "authors": "Smith, J.; Doe, A.; Martin, L.",
      "doi": "10.1101/2026.10.12.26312345",
      "abstract": "Background: ...",
      "abstract_url": "https://www.medrxiv.org/content/10.1101/2026.10.12.26312345",
      "pdf_url": "https://www.medrxiv.org/content/10.1101/2026.10.12.26312345.full.pdf",
      "full_text_url": "https://www.medrxiv.org/content/10.1101/2026.10.12.26312345.full",
      "publication_date": "2026-10-14",
      "category": "infectious diseases",
      "version": "1",
      "type": "new results",
      "license": "cc_by",
      "published_journal": "NA",
      "has_jatsxml": true
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search recent medRxiv pre-prints
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **MedRxiv/BioRxiv Search Node:** Downloads the pre-prints of the period and filters them on the query.
3. **Log Node:** Displays the matching articles.

### Example Configuration

```json
{
  "server": "biorxiv",
  "query": "CRISPR",
  "maxResults": 20,
  "intervalDays": 30
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Search query is required and cannot be empty`
- **Cause:** `query` is empty and the input is not a string.
- **Solution:** Fill in `query`, e.g. `{{ input.topic }}`.

#### `Search query cannot exceed 500 characters`
- **Cause:** The query is too long.
- **Solution:** Use a short keyword or phrase.

#### The run takes a long time
- **Cause:** `intervalDays` is large, so the node downloads many pages (especially on bioRxiv).
- **Solution:** Reduce `intervalDays`.

#### `total_results: 0`
- **Cause:** No article of the period contains the exact query.
- **Solution:** Use a shorter or more common term, or increase `intervalDays`.

#### `Error searching medrxiv: Rate Limited: Too many requests...`
- **Cause:** Too many calls to the API; the node already retried 3 times.
- **Solution:** Wait a few minutes and avoid running the workflow in tight loops.

#### `Error searching medrxiv: Request timeout after 30000ms` / `Server Error (5xx)`
- **Cause:** The medRxiv API is slow or unavailable.
- **Solution:** Retry later.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: parameter reference, search behavior, output schema, and workflow integration |

<!-- /SECTION: changelog -->
