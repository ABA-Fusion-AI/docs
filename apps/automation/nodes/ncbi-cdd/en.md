---
node_id: "ncbi-cdd"
title: "NCBI Conserved Domains"
description: "Search and retrieve conserved domain records from the NCBI Conserved Domain Database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-03"
author: "Fusion Team"
tags:
  - ncbi
  - conserved-domains
  - cdd
  - proteins
  - bioinformatics
  - research
  - life-sciences
related_nodes:
  - ncbi-bioproject
  - ncbi-biosample
  - ncbi-assembly
  - uni-prot-kb
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI Conserved Domains

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve conserved domain records from the National Center for Biotechnology Information (NCBI) Conserved Domain Database (CDD).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI Conserved Domains** node provides workflow access to conserved protein-domain records maintained by NCBI. Use it to search for domains with a text query or retrieve a specific conserved-domain record by its NCBI identifier.

### Key Features

- **Domain Search:** Find conserved domains using a keyword, protein name, gene, or research term
- **Domain Lookup:** Retrieve a specific conserved-domain record by NCBI identifier
- **Protein Annotation Support:** Use domain records to support protein-function and sequence-analysis workflows
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Investigate conserved regions associated with a protein or gene
- Retrieve domain metadata for bioinformatics and annotation workflows
- Enrich protein records with domain-related information
- Connect domain research with BioProject, BioSample, Assembly, or UniProt data
- Build internal catalogs of protein families and conserved functional regions

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text used to search conserved-domain records; required for `search` |
| `id` | `string` | Conditional | — | NCBI conserved-domain identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as a protein name, gene, or research term:

```json
{
  "operation": "search",
  "query": "BRCA1"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI conserved-domain identifier:

```json
{
  "operation": "getById",
  "id": "483963"
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

The example workflow contains only the placeholder `tap your NCBI API key here`, not a real key. Do not commit an actual NCBI API key in workflow files.

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
| `success` | `object` or `array` | Conserved-domain record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "483963",
      "name": "Conserved protein domain",
      "source": "NCBI Conserved Domain Database"
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
  "error": "NCBI Conserved Domains request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search Conserved Domains

```json
{
  "operation": "search",
  "query": "BRCA1"
}
```

### Retrieve a Conserved Domain by ID

```json
{
  "operation": "getById",
  "id": "483963"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "kinase domain"
}
```

### Dynamic Domain Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "transcription factor"
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
title: Search and retrieve NCBI conserved-domain records
```

### Common Patterns

- **Domain Search:** Manual Trigger → NCBI Conserved Domains → Log
- **Domain Lookup:** Manual Trigger → NCBI Conserved Domains → Log
- **Protein Enrichment:** UniProtKB → NCBI Conserved Domains → Database
- **Research Annotation:** NCBI Assembly → NCBI Conserved Domains → Function
- **Dynamic Search:** Input Data → NCBI Conserved Domains → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a protein name, gene, domain term, or supported NCBI search expression.

#### Domain ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI conserved-domain identifier.

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
| Missing ID | `getById` was selected without `id` | Provide a conserved-domain identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI BioProject](./ncbi-bioproject.md) - Work with study and project records
- [NCBI BioSample](./ncbi-biosample.md) - Work with biological sample records
- [NCBI Assembly](./ncbi-assembly.md) - Retrieve genome assembly records
- [UniProtKB](./uni-prot-kb.md) - Retrieve protein and annotation data
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-03 | Initial documentation and workflow examples for conserved-domain search and lookup |

<!-- /SECTION: changelog -->
