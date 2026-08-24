---
node_id: "bio-rxiv"
title: "bioRxiv/medRxiv"
description: "Search and retrieve preprints from the bioRxiv and medRxiv servers."
category: "healthcare-life-sciences"
subcategory: "other-medical-sources"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - healthcare
  - life-sciences
  - biomedical
  - preprints
  - biorxiv
  - medrxiv
  - research
related_nodes:
  - pmc
  - google-scholar
  - clinical-trials-search
---

<!-- SECTION: header -->
# bioRxiv/medRxiv

> **Category:** Healthcare & Life Sciences | **Subcategory:** Other Medical Sources | **Type:** Action Node

Search and retrieve scientific preprints from the bioRxiv and medRxiv servers.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **bioRxiv/medRxiv** node searches public preprint repositories for early research findings in biology, life sciences, medicine, and related fields.

Use the node to find relevant preprints by keyword and pass the returned records to downstream workflow nodes for filtering, logging, summarization, or further research.

### Key Features

- **Two Preprint Servers:** Search either bioRxiv or medRxiv.
- **Keyword Search:** Find preprints using a search query.
- **Research Discovery:** Retrieve recent biology, life sciences, and medical research.
- **Structured Results:** Return the server response as workflow data.
- **No Credentials Required:** The public preprint servers can be queried without an API key.
- **Workflow Ready:** Connect results to log, function, AI, or data-processing nodes.

### Use Cases

- Discover biology and life sciences preprints
- Find medical research before journal publication
- Build literature review workflows
- Send preprints to an AI summarization node
- Monitor research topics across bioRxiv and medRxiv
- Enrich research and healthcare automation workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation to perform. The supported operation is `search`. |
| `server` | `enum` | Yes | `biorxiv` | Preprint server to query: `biorxiv` or `medrxiv`. |
| `query` | `string` | Yes | — | Keyword or phrase used to search the selected preprint server. |

### Server Values

| Value | Server | Description |
|-------|--------|-------------|
| `biorxiv` | bioRxiv | Biology and life sciences preprints. |
| `medrxiv` | medRxiv | Medical and health sciences preprints. |

### Default Configuration

```json
{
  "operation": "search",
  "server": "biorxiv",
  "query": "neuroscience"
}
```

### Query Guidance

Use a focused keyword or phrase such as a disease, organism, method, or research topic. Queries are sent to the selected public preprint server, so result availability depends on the server's current index.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | unknown | Incoming workflow data. The configured operation, server, and query determine the search. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | object or array | Search results returned by the selected preprint server. |
| `error` | object | Error details when validation, network, or server processing fails. |

### Successful Response

The successful response contains the records returned for the configured search. A record can include preprint metadata such as its title, authors, abstract, DOI, publication date, server identifier, and source URL. The exact fields depend on the selected server and the current API response.

Example shape:

```json
{
  "success": true,
  "server": "biorxiv",
  "query": "neuroscience",
  "results": [
    {
      "title": "Example neuroscience preprint",
      "authors": ["Example Author"],
      "abstract": "Example abstract text.",
      "doi": "10.1101/2026.01.01.123456",
      "url": "https://www.biorxiv.org/"
    }
  ]
}
```

### Error Response

```json
{
  "success": false,
  "error": "Unable to retrieve preprints from the selected server."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Search bioRxiv

Search biology and life sciences preprints for neuroscience research.

```json
{
  "operation": "search",
  "server": "biorxiv",
  "query": "neuroscience"
}
```

### Example: Search medRxiv

Search medical preprints for research about long COVID.

```json
{
  "operation": "search",
  "server": "medrxiv",
  "query": "long COVID"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search bioRxiv preprints and inspect the results
```

### Common Patterns

- **Research Search:** Manual Trigger → bioRxiv/medRxiv → Log
- **AI Review:** Manual Trigger → bioRxiv/medRxiv → AI Chat
- **Result Filtering:** bioRxiv/medRxiv → Function → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### No results returned

**Cause:** The query is too broad, too specific, or does not match records in the selected server's index.

**Solution:** Try a shorter query or a related keyword, and verify that the correct server is selected.

#### Invalid server

**Cause:** The `server` value is not `biorxiv` or `medrxiv`.

**Solution:** Select one of the supported server values.

#### Search request failed

**Cause:** The public server is unavailable, rate-limited, or returned an unexpected response.

**Solution:** Check the server status and try the workflow again later.

#### Missing query

**Cause:** No search term was supplied.

**Solution:** Configure a non-empty `query` value before running the node.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [PMC](../pmc/en.md) - Search biomedical articles and abstracts
- [Google Scholar](../google-scholar/en.md) - Search scholarly publications
- [Clinical Trials Search](../clinical-trials-search/en.md) - Search registered clinical studies

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation |

<!-- /SECTION: changelog -->
