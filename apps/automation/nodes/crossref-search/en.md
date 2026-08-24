---
node_id: "crossref-search"
title: "Crossref Search"
description: "Search Crossref for scholarly works, publications, and research papers."
category: "web-search-information"
subcategory: "academic-search"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - crossref
  - academic-search
  - scholarly-works
  - publications
  - research
  - doi
related_nodes:
  - google-scholar-search-url
  - semantic-scholar
  - openalex
---

<!-- SECTION: header -->
# Crossref Search

> **Category:** Web Search & Information | **Subcategory:** Academic Search | **Type:** Action Node

Search Crossref for scholarly works, publications, and research papers by keyword, with optional publication and type filters.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Crossref Search** node queries the public Crossref scholarly metadata service and returns publications matching a search query.

Use it to discover academic works by title, DOI, author, or topic, then pass the results to downstream nodes for logging, filtering, citation analysis, or summarization.

### Key Features

- **Scholarly Search:** Find academic publications and research papers.
- **Flexible Queries:** Search by topic, title, DOI, author, or other scholarly metadata.
- **Crossref Filters:** Narrow results by publication date, work type, and other Crossref filter expressions.
- **Structured Metadata:** Return Crossref work records with bibliographic information and identifiers.
- **No Credentials Required:** Uses the public Crossref API.
- **Workflow Ready:** Connect results to log, function, AI, or data-processing nodes.

### Use Cases

- Discover research papers for a literature review
- Find publications related to a research topic
- Search for works by a known author or DOI
- Filter journal articles by publication date
- Build citation and scholarly metadata workflows
- Send publication metadata to an AI summarization node

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | Yes | — | Search term or phrase. Can identify a topic, title, author, DOI, or other scholarly metadata. |
| `filter` | `string` | No | — | Optional comma-separated Crossref filters, such as `from-pub-date:2024-01-01,type:journal-article`. |

### Query Examples

Search by topic:

```json
{
  "query": "artificial intelligence"
}
```

Search with Crossref filters:

```json
{
  "query": "artificial intelligence",
  "filter": "from-pub-date:2024-01-01,type:journal-article"
}
```

### Filter Syntax

Crossref filters are supplied as a comma-separated string. Each filter uses the form `name:value`.

| Filter | Example | Description |
|--------|---------|-------------|
| `from-pub-date` | `from-pub-date:2024-01-01` | Include works published on or after the specified date. |
| `until-pub-date` | `until-pub-date:2026-08-24` | Include works published on or before the specified date. |
| `type` | `type:journal-article` | Limit results to a Crossref work type. |

The available filter names and values follow the Crossref REST API. Invalid filter expressions may cause the request to fail or be ignored by Crossref.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | unknown | Incoming workflow data. The configured `query` and optional `filter` determine the Crossref search. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | object or array | Crossref search response containing matching scholarly work metadata. |
| `error` | object | Error details when validation, network, or Crossref processing fails. |

### Successful Response

Crossref responses commonly include a result container with matching work records. A work record can contain metadata such as its title, authors, DOI, publication date, publisher, container title, URL, and type. The exact fields depend on the publication record.

Example shape:

```json
{
  "message-type": "work-list",
  "message": {
    "total-results": 1,
    "items": [
      {
        "title": ["Example research publication"],
        "author": [
          {
            "given": "Example",
            "family": "Author"
          }
        ],
        "DOI": "10.1234/example.2026.001",
        "type": "journal-article",
        "published": {
          "date-parts": [[2026, 1, 1]]
        },
        "URL": "https://doi.org/10.1234/example.2026.001"
      }
    ]
  }
}
```

### Error Output

Errors are routed through the `error` output. They can result from a missing query, an invalid filter, a network failure, rate limiting, or an unavailable Crossref service.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Search by Topic

```json
{
  "query": "artificial intelligence"
}
```

### Example: Search Journal Articles Since a Date

```json
{
  "query": "machine learning in healthcare",
  "filter": "from-pub-date:2024-01-01,type:journal-article"
}
```

### Example: Search by Author or DOI

```json
{
  "query": "Ada Lovelace"
}
```

```json
{
  "query": "10.1038/s41586-020-2649-2"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search Crossref publications and inspect the results
```

### Common Patterns

- **Publication Search:** Manual Trigger → Crossref Search → Log
- **Filtered Research:** Manual Trigger → Crossref Search → Function
- **AI Literature Review:** Crossref Search → AI Chat → Log
- **Citation Enrichment:** Crossref Search → Function → Database

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Missing query

**Cause:** The `query` parameter is empty or was not provided.

**Solution:** Configure a non-empty search term or phrase.

#### No results returned

**Cause:** The query is too specific or the selected filters exclude matching works.

**Solution:** Try a broader query and remove filters one at a time.

#### Invalid filter

**Cause:** A filter name or value does not follow Crossref REST API syntax.

**Solution:** Use comma-separated `name:value` expressions supported by Crossref, such as `type:journal-article`.

#### Request failed or was rate-limited

**Cause:** Crossref may be temporarily unavailable or may limit requests.

**Solution:** Retry later and avoid sending unnecessary repeated requests in a tight loop.

#### Incomplete publication metadata

**Cause:** Crossref metadata is supplied by publishers and may be incomplete for some works.

**Solution:** Use the DOI or URL to verify the publication with the publisher or another scholarly source.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Google Scholar Search URL](../google-scholar-search-url/en.md) - Generate Google Scholar search URLs
- [Semantic Scholar](../semantic-scholar/en.md) - Search scholarly literature and citation data
- [OpenAlex](../openalex/en.md) - Search open scholarly metadata

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation |

<!-- /SECTION: changelog -->
