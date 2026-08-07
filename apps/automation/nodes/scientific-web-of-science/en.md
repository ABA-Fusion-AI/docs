---
node_id: "scientific-web-of-science"
title: "Web of Science Search"
description: "Search Web of Science through Clarivate API credentials and return normalized article metadata"
category: "research"
subcategory: "academic-search"
version: "1.0.0"
language: "en"
last_updated: "2026-06-29"
author: "Fusion Team"
tags:
  - web-of-science
  - clarivate
  - scientific-articles
  - academic-search
  - research
related_nodes:
  - scopus-search
  - science-direct-search
  - google-scholar-search-url
---

<!-- SECTION: header -->
# Web of Science Search

> **Category:** Research | **Type:** Action Node

Search Web of Science records through Clarivate API credentials and return normalized scientific article metadata.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Web of Science Search** node supports both classic and starter API response modes. It normalizes records into the same article shape used by the Scopus and ScienceDirect nodes, including title, date, abstract, authors, publisher, journal, keywords, identifiers, citations, and source links when available.

### Key Features

- Search Web of Science Core Collection records
- Supports `classic` and `starter` API modes
- Uses Clarivate API key authentication
- Normalized metadata output for cross-source comparison
- Captures Web of Science UID in `identifiers.wosUid`

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | `string` | Yes | Web of Science query string |
| `apiKey` | `string` | Yes | Clarivate Web of Science API key |
| `apiMode` | `classic \| starter` | No | API response mode. Defaults to `classic` |
| `databaseId` | `string` | No | Web of Science database id. Defaults to `WOS` |
| `count` | `number` | No | Number of records to return, from 1 to 100 |
| `firstRecord` | `number` | No | One-based first record index |
| `baseUrl` | `string` | No | Override API base URL for proxy or institution-specific routing |
| `timeoutMs` | `number` | No | Request timeout in milliseconds |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Search response with normalized Web of Science records |
| `error` | `Error` | API, network, validation, or credentials error |

### Output Fields

Each article result can include `title`, `date`, `abstract`, `authors`, `writerNames`, `publisher`, `journal`, `publicationDate`, `year`, `documentType`, `language`, `keywords`, `doi`, `issn`, `citationCount`, `url`, `links`, and `identifiers.wosUid`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Classic API

```json
{
  "query": "TS=(\"artificial intelligence\" AND \"mining safety\")",
  "apiKey": "{{secrets.web_of_science_api_key}}",
  "apiMode": "classic",
  "databaseId": "WOS",
  "count": 10,
  "firstRecord": 1
}
```

### Starter API

```json
{
  "query": "\"artificial intelligence\" \"mining safety\"",
  "apiKey": "{{secrets.web_of_science_api_key}}",
  "apiMode": "starter",
  "databaseId": "WOS",
  "count": 10,
  "firstRecord": 1
}
```

<!-- /SECTION: examples -->

