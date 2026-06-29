---
node_id: "science-direct-search"
title: "ScienceDirect Search"
description: "Search ScienceDirect articles using the Elsevier API and return normalized scholarly metadata"
category: "research"
subcategory: "academic-search"
version: "1.0.0"
language: "en"
last_updated: "2026-06-29"
author: "Fusion Team"
tags:
  - sciencedirect
  - elsevier
  - scientific-articles
  - academic-search
  - research
related_nodes:
  - scopus-search
  - web-of-science-search
  - google-scholar-search-url
---

<!-- SECTION: header -->
# ScienceDirect Search

> **Category:** Research | **Type:** Action Node

Search ScienceDirect through the Elsevier API and return normalized article metadata including title, abstract, date, authors, publisher, journal, identifiers, and links when available.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **ScienceDirect Search** node is built for Elsevier full-text and journal article discovery. It uses the same normalized output shape as the Scopus node so results from multiple scholarly sources can be merged, compared, or exported consistently.

### Key Features

- Search ScienceDirect with an Elsevier API key
- Optional institution token support
- Returns normalized article metadata
- Includes writer aliases, structured authors, publisher, journal, identifiers, and abstract when returned
- Supports richer metadata with `view: "COMPLETE"`

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | `string` | Yes | ScienceDirect query or keyword expression |
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

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Search response with normalized article results |
| `error` | `Error` | API, network, validation, or credentials error |

### Important Result Fields

Each article result can include:

- `title`
- `date`
- `abstract`
- `authors`
- `writerNames`
- `writers`
- `publisher`
- `publishedBy`
- `journal`
- `publicationDate`
- `year`
- `documentType`
- `language`
- `keywords`
- `doi`
- `issn`
- `isbn`
- `volume`
- `issue`
- `pages`
- `url`
- `links`
- `identifiers`

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Example

```json
{
  "query": "\"artificial intelligence\" AND \"mining safety\"",
  "apiKey": "{{secrets.elsevier_api_key}}",
  "institutionToken": "{{secrets.elsevier_institution_token}}",
  "count": 10,
  "start": 0,
  "view": "COMPLETE"
}
```

<!-- /SECTION: examples -->

