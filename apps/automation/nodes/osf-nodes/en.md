---
node_id: "osf-nodes"
title: "OSF Nodes"
description: "Search nodes from Open Science Framework API."
category: "research"
subcategory: "academic-search"
version: "1.0.0"
language: "en"
last_updated: "2026-08-17"
author: "Fusion Team"
tags:
  - osf
  - open-science-framework
  - research
  - academic-search
  - science
  - open-science
  - nodes
  - datasets
  - projects
related_nodes:
  - science-direct-search
  - scopus-search
  - google-scholar
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# OSF Nodes

> **Category:** Research | **Type:** Action Node

Search and discover research projects, studies, components, and datasets from the [Open Science Framework (OSF)](https://osf.io/) API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **OSF Nodes** node connects to the Open Science Framework (OSF) API v2 to query and retrieve public research projects, components, and datasets. In OSF, a **node** is the primary organizational entity representing a research study, project, laboratory workspace, or data repository.

This node enables researchers, academic data pipelines, and automation workflows to programmatically search open science projects by title or keywords, paginate through search results, and access structured project metadata, contributor links, and web URLs.

### Key Features

- **Open Science Discovery:** Search thousands of open-access scientific projects, pre-registrations, and shared datasets hosted on the Open Science Framework.
- **Title & Keyword Filtering:** Query public nodes by title substring or topic keywords.
- **Flexible Pagination:** Control search pagination using `page` and `page_size` parameters.
- **Rich Metadata Extraction:** Retrieves project titles, abstracts, descriptions, creation/modification dates, license info, and category classifications.
- **No API Key Required for Public Data:** Queries public OSF repositories directly without mandatory credentials.

### Use Cases

- **Academic Literature & Dataset Discovery:** Automatically find shared scientific data, experimental protocols, and project materials.
- **Meta-Research & Bibliometrics:** Track research activity, open-science compliance, and data-sharing trends across institutions.
- **Automated Research Watch & Alerts:** Monitor new project registrations and studies in specific disciplines (e.g., climate science, psychology, neuroscience).
- **RAG & Scientific Knowledge Bases:** Ingest open research documentation and metadata into AI pipelines or enterprise search indexes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `title` | `string` | ❌ No | *(empty)* | Filter nodes by title or search keyword (e.g., `climate`, `machine learning`). |
| `page` | `number` | ❌ No | `1` | Page number for paginated results (1-indexed). |
| `page_size` | `number` | ❌ No | `10` | Number of results to return per page (max `50` or `100` depending on API limits). |

---

### Parameter Details

#### `title`
Specifies a search term to match against project titles in the Open Science Framework.
- Type: `string`
- Optional. When omitted, returns recent public nodes.
- Example: `"climate change"`, `"COVID-19"`, `"open neuroscience"`

#### `page`
The page number to retrieve when navigating through multiple pages of results.
- Type: `number`
- Default: `1`
- Example: `1`, `2`, `5`

#### `page_size`
The number of node objects returned in a single API request.
- Type: `number`
- Default: `10`
- Example: `10`, `25`, `50`

---

### API Endpoint

The node queries the official Open Science Framework API v2 endpoint:

```text
https://api.osf.io/v2/nodes/
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming trigger or workflow payload to start node execution. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` \| `array` | Emitted on successful search. Contains the API response data and list of OSF nodes. |
| `error` | `Error` | Emitted when network request, query parameter validation, or API response parsing fails. |

---

### Output Data Structure

The `success` output provides structured JSON representing the OSF JSON:API response format:

```json
{
  "data": [
    {
      "id": "8239n",
      "type": "nodes",
      "attributes": {
        "title": "Global Climate Simulation Dataset and Reproducibility Archive",
        "description": "Comprehensive simulation models and open datasets for global climate variability research.",
        "category": "project",
        "date_created": "2026-03-15T14:20:00.000Z",
        "date_modified": "2026-08-10T09:45:00.000Z",
        "public": true,
        "tags": [
          "climate",
          "meteorology",
          "open-data",
          "simulation"
        ],
        "current_user_can_comment": false,
        "current_user_permissions": [
          "read"
        ],
        "fork": false,
        "registration": false,
        "collection": false
      },
      "relationships": {
        "contributors": {
          "links": {
            "related": {
              "href": "https://api.osf.io/v2/nodes/8239n/contributors/"
            }
          }
        },
        "files": {
          "links": {
            "related": {
              "href": "https://api.osf.io/v2/nodes/8239n/files/"
            }
          }
        }
      },
      "links": {
        "self": "https://api.osf.io/v2/nodes/8239n/",
        "html": "https://osf.io/8239n/"
      }
    }
  ],
  "links": {
    "first": "https://api.osf.io/v2/nodes/?filter%5Btitle%5D=climate&page=1",
    "last": "https://api.osf.io/v2/nodes/?filter%5Btitle%5D=climate&page=10",
    "prev": null,
    "next": "https://api.osf.io/v2/nodes/?filter%5Btitle%5D=climate&page=2"
  },
  "meta": {
    "total": 98,
    "per_page": 10
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `data` | `array` | List of matching node objects returned by the OSF API |
| `data[].id` | `string` | Unique 5-character alphanumeric identifier of the OSF node (e.g. `8239n`) |
| `data[].type` | `string` | Object type (`"nodes"`) |
| `data[].attributes.title` | `string` | Full title of the project or component |
| `data[].attributes.description` | `string` | Detailed abstract or description of the research project |
| `data[].attributes.category` | `string` | Node category (e.g. `project`, `data`, `hypothesis`, `methods`, `analysis`) |
| `data[].attributes.date_created` | `string` | ISO 8601 creation timestamp |
| `data[].attributes.date_modified` | `string` | ISO 8601 last update timestamp |
| `data[].attributes.public` | `boolean` | Indicates whether the node is publicly accessible |
| `data[].attributes.tags` | `string[]` | User-defined keywords and discipline tags |
| `data[].links.html` | `string` | Direct web URL to the project on `osf.io` |
| `meta.total` | `number` | Total number of matching records available across all pages |
| `links.next` | `string` \| `null` | URL to retrieve the next page of results |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Search Projects by Keyword

Search for open projects related to climate science with 50 results per page.

**Parameter Configuration:**

```text
Title: climate
Page: 1
Page_size: 50
```

---

### Example 2: Paginated Scientific Harvester

Query consecutive pages in an automated data pipeline.

**Workflow Pattern:**

```text
Manual Trigger
  → OSF Nodes (title: "machine learning", page: 1, page_size: 25)
  → Function (extract project IDs and HTML links)
  → Database Insert (store research records)
```

---

### Example 3: Academic Watch Alerting

Run a weekly search for newly registered open-science projects and notify a research team.

**Workflow Pattern:**

```text
Cron Trigger (Weekly on Monday)
  → OSF Nodes (title: "genomics", page_size: 10)
  → Filter Array (filter where date_created is within last 7 days)
  → Email / Slack / Discord Send
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search nodes from Open Science Framework API
```

### Common Patterns

- **Literature & Data Harvester:** Cron Trigger → OSF Nodes → Function → Vector Database / Elasticsearch.
- **Multi-Source Scientific Aggregation:** OSF Nodes + ScienceDirect + Scopus → Merge → Deduplicate → Export CSV / JSON.
- **Open Science Monitor:** OSF Nodes → Function (check tags/license) → Alert if new project matches criteria.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### No results found / Empty `data` array

**Cause:** The search query in the `title` parameter did not match any public OSF projects.

**Solution:** Broaden the search keyword or verify the spelling. Note that search matches title substrings.

#### Rate Limiting (HTTP 429)

**Cause:** The OSF API applies rate limits to unauthenticated public requests (typically 100 requests per minute).

**Solution:** Introduce delays or debouncing between batch pagination requests using the **Delay** node.

#### Pagination Bounds

**Cause:** The requested `page` exceeds the total number of available pages (`meta.total / per_page`).

**Solution:** Check `meta.total` or verify if `links.next` is null before requesting subsequent pages.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Network error` | DNS resolution failure or API unreachable | Check outbound internet connection to `api.osf.io` |
| `HTTP 400 Bad Request` | Invalid query parameter or format | Verify `page` and `page_size` are positive integers |
| `HTTP 429 Too Many Requests` | Exceeded public API rate limit | Add a delay between requests |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [ScienceDirect Search](../science-direct-search/en.md) — Search Elsevier full-text research articles
- [Scopus Search](../scopus-search/en.md) — Search abstract and citation database for peer-reviewed literature
- [Google Scholar](../google-scholar/en.md) — Generate Google Scholar search URLs for academic queries
- [HTTP Request](../http-request/en.md) — Make custom API calls to OSF endpoints (e.g., users, files, registrations)
- [Filter Array](../filter-array/en.md) — Filter node records by tags, dates, or categories
- [Function](../function/en.md) — Transform OSF JSON:API responses into custom data models
- [Log](../log/en.md) — Inspect node responses in the workflow console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial release |

<!-- /SECTION: changelog -->
