---
node_id: "ncbi-geoprofiles"
title: "NCBI GEO Profiles"
description: "Search and retrieve gene expression profiles from the NCBI GEO Profiles database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-03"
author: "Fusion Team"
tags:
  - ncbi
  - geo-profiles
  - gene-expression
  - genomics
  - bioinformatics
  - life-sciences
related_nodes:
  - ncbi-bioproject
  - ncbi-biosample
  - ncbi-gene
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI GEO Profiles

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve gene expression profile data from the National Center for Biotechnology Information (NCBI) GEO Profiles database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI GEO Profiles** node provides workflow access to gene expression profiles derived from the NCBI Gene Expression Omnibus (GEO) resource. Use it to search profiles with a text query or retrieve a specific profile by identifier.

### Key Features

- **Profile Search:** Find gene expression profiles using a gene, disease, organism, study term, or NCBI search expression
- **Profile Lookup:** Retrieve a specific GEO Profile by NCBI identifier
- **Expression Data:** Access profile values and metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Explore gene expression patterns associated with diseases or experimental conditions
- Retrieve profiles for genomics and bioinformatics workflows
- Enrich research datasets with public expression-profile metadata
- Connect expression studies with BioProject, BioSample, Gene, or publication data
- Build gene-expression research and annotation workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find GEO Profiles; required for `search` |
| `id` | `string` | Conditional | — | NCBI GEO Profile identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as a gene, organism, disease, or research term:

```json
{
  "operation": "search",
  "query": "BRCA1"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the GEO Profile identifier:

```json
{
  "operation": "getById",
  "id": "132766449"
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

The example workflow contains only the placeholder `tap your API key here`, not a real key. Do not commit an actual NCBI API key in workflow files.

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
| `success` | `object` or `array` | GEO Profile record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "132766449",
      "gene": "BRCA1",
      "source": "NCBI GEO Profiles"
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
  "error": "NCBI GEO Profiles request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search GEO Profiles by Gene

```json
{
  "operation": "search",
  "query": "BRCA1"
}
```

### Retrieve a GEO Profile by ID

```json
{
  "operation": "getById",
  "id": "132766449"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "BRCA1"
}
```

### Dynamic GEO Profile Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "Homo sapiens gene expression"
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
title: Search and retrieve NCBI GEO Profiles
```

### Common Patterns

- **Profile Search:** Manual Trigger → NCBI GEO Profiles → Log
- **Profile Lookup:** Manual Trigger → NCBI GEO Profiles → Log
- **Expression Enrichment:** NCBI GEO Profiles → Database or Function
- **Dynamic Search:** Input Data → NCBI GEO Profiles → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a gene, organism, disease, keyword, or supported NCBI search expression.

#### GEO Profile ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI GEO Profile identifier.

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
| Missing ID | `getById` was selected without `id` | Provide a GEO Profile identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI BioProject](./ncbi-bioproject.md) - Work with study and project records
- [NCBI BioSample](./ncbi-biosample.md) - Work with biological sample records
- [NCBI Gene](./ncbi-gene.md) - Retrieve gene records and annotations
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-03 | Initial documentation and workflow examples for GEO Profiles search and lookup |

<!-- /SECTION: changelog -->
