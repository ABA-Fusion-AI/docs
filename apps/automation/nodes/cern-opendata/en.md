---
node_id: "cern-opendata"
title: "CERN Open Data"
description: "Search and retrieve scientific records and datasets from the CERN Open Data Portal API. "
category: "Web Search & Information"
subcategory: "Scientific Search"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - cern
  - open-data
  - scientific-search
  - research
  - particle-physics
  - datasets
related_nodes:
  - open-data-search
  - science-direct-search
  - scientific-web-of-science
---

<!-- SECTION: header -->
# CERN Open Data

> **Category:** Web Search & Information | **Type:** Action Node

Search records and datasets from the CERN Open Data Portal API to retrieve scientific metadata for research, analysis, and discovery workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **CERN Open Data** node connects to the CERN Open Data Portal API and lets workflows search records, dataset metadata, and scientific resources published by CERN. It is useful for research pipelines, scientific discovery automation, and integrations that need accessible open data from high-energy physics and related domains.

### Key Features

- **Scientific Dataset Search:** Query CERN Open Data records and metadata
- **Research Discovery:** Find relevant datasets and publication-related resources
- **Structured Responses:** Return machine-readable JSON for downstream processing
- **Workflow Ready:** Combine with filtering, enrichment, and storage steps
- **Open Research Integration:** Support public scientific data workflows

### Typical Use Cases

- Discover datasets from the CERN Open Data Portal
- Enrich scientific workflows with official CERN records
- Build research dashboards and open-data catalogs
- Collect metadata for dataset indexing and analysis
- Support academic or experimental research automation

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | ✅ Yes | — | Search text or keyword used to find CERN Open Data records |
| `limit` | `number` | ❌ No | `10` | Maximum number of results to retrieve |
| `page` | `number` | ❌ No | `1` | Result page number for paginated responses |
| `sort` | `string` | ❌ No | — | Optional sorting field supported by the API |
| `filters` | `object` | ❌ No | — | Additional filters for narrowing the search |

### Example

```text
query: "CMS"
limit: 10
page: 1
filters:
  type: "dataset"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Keyword search or a structured object with request parameters |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Response containing matching CERN Open Data records |
| `error` | `object` | Error details when the request is invalid or the API fails |

### Success Output Example

```json
{
  "hits": {
    "hits": [
      {
        "id": "cms-open-data",
        "title": "CMS Open Data",
        "type": "dataset",
        "description": "Open data from the CMS experiment"
      }
    ],
    "total": 1
  }
}
```

### Error Output Example

```json
{
  "success": false,
  "error": "The CERN Open Data request was invalid or no results were returned.",
  "query": "CMS"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Search CMS Open Data Records

```text
query: "CMS"
limit: 10
page: 1
```

**Result:**

```json
{
  "hits": {
    "hits": [
      {
        "id": "cms-open-data",
        "title": "CMS Open Data",
        "type": "dataset"
      }
    ]
  }
}
```

### Example: Research Workflow

Use the node to discover scientific resources from CERN, then pass the result to filtering or enrichment steps for further research processing.

<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->
