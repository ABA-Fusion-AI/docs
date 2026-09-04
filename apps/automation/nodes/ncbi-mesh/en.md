---
node_id: "ncbi-mesh"
title: "NCBI MeSH"
description: "Search and retrieve biomedical subject-heading records from the NCBI MeSH database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - ncbi
  - mesh
  - medical-subject-headings
  - biomedical-literature
  - clinical-terminology
  - bioinformatics
  - life-sciences
related_nodes:
  - pub-med-search
  - ncbi-medgen
  - ncbi-clinvar
  - ncbi-gtr
  - pmc
---

<!-- SECTION: header -->
# NCBI MeSH

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve Medical Subject Headings (MeSH) records from the National Center for Biotechnology Information (NCBI) MeSH database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI MeSH** node provides workflow access to the controlled vocabulary used to index biomedical literature. Use it to search subject headings by topic, disease, gene, or research term, or retrieve a specific MeSH record by identifier.

### Key Features

- **Subject-Heading Search:** Find MeSH terms using a topic, disease, gene, or NCBI search expression
- **Record Lookup:** Retrieve a specific MeSH record by NCBI identifier
- **Terminology Metadata:** Access preferred terms, entry terms, definitions, tree information, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Build consistent biomedical literature searches
- Normalize medical terminology across research datasets
- Discover related subject headings and hierarchical MeSH concepts
- Connect controlled vocabulary with PubMed, PMC, MedGen, ClinVar, or GTR workflows
- Improve literature annotation and knowledge-management pipelines

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find MeSH records; required for `search` |
| `id` | `string` | Conditional | — | NCBI MeSH identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a topic, disease, gene, or MeSH search expression:

```json
{
  "operation": "search",
  "query": "Breast Neoplasms[MeSH Terms]"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI MeSH identifier:

```json
{
  "operation": "getById",
  "id": "68001943"
}
```

### API and Authentication

NCBI public services can generally be used without an API key, subject to rate limits. An NCBI API key can provide enhanced access for supported services.

Store the key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.ncbiApiKey}}"
}
```

The example workflow contains empty API-key values, not a real key. Do not commit an actual NCBI API key in workflow files.

### Request Limits

Limits depend on the NCBI service and whether an API key is supplied. Respect NCBI usage policies and avoid high-frequency requests without an appropriate request strategy.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional dynamic input containing `operation`, `query`, `id`, and `apiKey` overrides |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | MeSH record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "68001943",
      "term": "Breast Neoplasms",
      "source": "NCBI MeSH"
    }
  ]
}
```

The exact fields depend on the selected operation and the response returned by NCBI.

### Error Output

Invalid operations, missing query or ID values, rate limits, network failures, and NCBI API errors are routed to the error output.

```json
{
  "success": false,
  "error": "NCBI MeSH request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search MeSH Terms

```json
{
  "operation": "search",
  "query": "Breast Neoplasms[MeSH Terms]"
}
```

### Retrieve a MeSH Record by ID

```json
{
  "operation": "getById",
  "id": "68001943"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "Breast Neoplasms[MeSH Terms]"
}
```

### Dynamic MeSH Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "genetic diseases"
}
```

Keep the API key in Fusion’s secret system even when the query comes from incoming data.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Search and retrieve NCBI MeSH records
```

### Common Patterns

- **Term Search:** Manual Trigger → NCBI MeSH → Log
- **Term Lookup:** Manual Trigger → NCBI MeSH → Log
- **Literature Search:** NCBI MeSH → PubMed Search → Database
- **Clinical Terminology:** NCBI MeSH → NCBI MedGen → Function
- **Dynamic Search:** Input Data → NCBI MeSH → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a topic, disease, gene, keyword, or supported MeSH search expression.

#### MeSH ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI MeSH identifier.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency, use batching where appropriate, or configure an NCBI API key through Fusion’s secret system.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|------|----------|
| Missing query | `search` was selected without `query` | Provide a search query |
| Missing ID | `getById` was selected without `id` | Provide a MeSH identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [PubMed Search](./pub-med-search.md) - Search biomedical literature
- [NCBI MedGen](./ncbi-medgen.md) - Search medical concepts and terminology
- [NCBI ClinVar](./ncbi-clinvar.md) - Search clinical variant records
- [NCBI GTR](./ncbi-gtr.md) - Search genetic test records
- [PMC](./pmc.md) - Retrieve PubMed Central content

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial documentation and workflow examples for MeSH search and lookup |

<!-- /SECTION: changelog -->
