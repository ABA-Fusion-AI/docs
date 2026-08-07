---
node_id: "scopus-search"
title: "Scopus Search"
description: "Search scientific articles in Scopus using the Elsevier API and return normalized article metadata"
category: "research"
subcategory: "academic-search"
version: "1.0.0"
language: "en"
last_updated: "2026-06-29"
author: "Fusion Team"
tags:
  - scopus
  - elsevier
  - scientific-articles
  - academic-search
  - research
related_nodes:
  - science-direct-search
  - scientific-web-of-science
  - google-scholar-search-url
---

<!-- SECTION: header -->
# Scopus Search

> **Category:** Research | **Type:** Action Node

Search Scopus through the Elsevier API and return normalized scientific article metadata for downstream analysis, deduplication, reporting, or LLM summarization.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Scopus Search** node queries the Elsevier Scopus Search API. Results are normalized into a consistent article shape with title, abstract, publication date, authors, publisher, journal, keywords, identifiers, citation count, and links when the API returns those fields.

### Key Features

- Search Scopus using an Elsevier API key
- Optional institution token support
- Normalized `results` array
- Author and writer aliases for easy workflow mapping
- Extracts article identifiers such as DOI, EID, PII, ISSN, and ISBN
- Supports richer metadata with `view: "COMPLETE"`

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | `string` | Yes | Scopus query expression or keyword search |
| `apiKey` | `string` | Yes | Elsevier API key |
| `institutionToken` | `string` | No | Elsevier institution token for subscribed access |
| `count` | `number` | No | Number of records to return, from 1 to 100 |
| `start` | `number` | No | Zero-based result offset |
| `sort` | `string` | No | Elsevier sort expression |
| `date` | `string` | No | Date filter accepted by the Elsevier API |
| `view` | `string` | No | Elsevier response view. Use `COMPLETE` for richer metadata |
| `timeoutMs` | `number` | No | Request timeout in milliseconds |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Trigger or upstream data. Parameters can use workflow expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Search response with normalized article results |
| `error` | `Error` | API, network, validation, or credentials error |

### Output Shape

```json
{
  "source": "scopus",
  "query": "artificial intelligence in mining safety",
  "totalResults": 100,
  "count": 5,
  "start": 0,
  "results": [
    {
      "source": "scopus",
      "title": "Article title",
      "date": "2026-01-15",
      "abstract": "Article abstract when returned by Scopus.",
      "authors": [
        {
          "name": "Jane Smith",
          "givenName": "Jane",
          "familyName": "Smith",
          "id": "12345678900",
          "affiliation": "Example University"
        }
      ],
      "writerNames": ["Jane Smith"],
      "writers": ["Jane Smith"],
      "publisher": "Elsevier",
      "publishedBy": "Elsevier",
      "journal": "Journal Name",
      "publicationDate": "2026-01-15",
      "year": 2026,
      "documentType": "Article",
      "keywords": ["AI", "mining safety"],
      "doi": "10.0000/example",
      "issn": "0000-0000",
      "citationCount": 12,
      "url": "https://api.elsevier.com/...",
      "identifiers": {
        "doi": "10.0000/example",
        "eid": "2-s2.0-...",
        "scopusId": "SCOPUS_ID:..."
      }
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Example

```json
{
  "query": "TITLE-ABS-KEY(\"artificial intelligence\" AND \"mining safety\")",
  "apiKey": "{{secrets.elsevier_api_key}}",
  "institutionToken": "{{secrets.elsevier_institution_token}}",
  "count": 10,
  "start": 0,
  "view": "COMPLETE"
}
```

<!-- /SECTION: examples -->

