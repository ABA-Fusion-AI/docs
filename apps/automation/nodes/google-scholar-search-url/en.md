---
node_id: "google-scholar-search-url"
title: "Google Scholar Search URL"
description: "Generate a Google Scholar search URL for manual or browser-authenticated academic research workflows"
category: "research"
subcategory: "academic-search"
version: "1.0.0"
language: "en"
last_updated: "2026-06-29"
author: "Fusion Team"
tags:
  - google-scholar
  - scientific-articles
  - academic-search
  - research
related_nodes:
  - scopus-search
  - science-direct-search
  - web-of-science-search
---

<!-- SECTION: header -->
# Google Scholar Search URL

> **Category:** Research | **Type:** Action Node

Build a Google Scholar search URL from workflow parameters. This node intentionally does not scrape Google Scholar results because Google Scholar does not provide a stable official public search API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Google Scholar Search URL** node is useful for manual review, browser-assisted workflows, or handoff to a human researcher. It returns a URL and note explaining that result scraping is not performed.

### Key Features

- Generates a Google Scholar URL from a query
- Supports year range filters
- Supports language selection
- Can include or exclude patents/citations in the generated search URL
- Requires no API key

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | `string` | Yes | Search query |
| `startYear` | `number` | No | Minimum publication year |
| `endYear` | `number` | No | Maximum publication year |
| `language` | `string` | No | Scholar UI language. Defaults to `en` |
| `start` | `number` | No | Result offset |
| `includePatents` | `boolean` | No | Include patent-related Scholar results |
| `includeCitations` | `boolean` | No | Include citation-only entries |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | URL generation result |
| `error` | `Error` | Validation error |

### Output Shape

```json
{
  "source": "google-scholar",
  "query": "artificial intelligence in mining safety",
  "mode": "url",
  "url": "https://scholar.google.com/scholar?q=...",
  "note": "Google Scholar does not provide a stable official public search API. This node generates the search URL instead of scraping results."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Example

```json
{
  "query": "artificial intelligence in mining safety",
  "startYear": 2020,
  "endYear": 2026,
  "language": "en",
  "start": 0,
  "includePatents": false,
  "includeCitations": false
}
```

<!-- /SECTION: examples -->

