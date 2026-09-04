---
node_id: "ncbi-medgen"
title: "NCBI MedGen"
description: "Search and retrieve medical concept records from the NCBI MedGen database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - ncbi
  - medgen
  - medical-concepts
  - clinical-terminology
  - genetics
  - genomics
  - bioinformatics
  - life-sciences
related_nodes:
  - ncbi-clinvar
  - ncbi-gtr
  - ncbi-gene
  - ncbi-dbvar
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI MedGen

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve medical concept records from the National Center for Biotechnology Information (NCBI) MedGen database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI MedGen** node provides workflow access to standardized medical concepts and related clinical terminology. Use it to search concepts by disease, condition, gene, symptom, or research term, or retrieve a specific concept by identifier.

### Key Features

- **Concept Search:** Find medical concepts using a disease, condition, gene, symptom, or NCBI search expression
- **Concept Lookup:** Retrieve a specific MedGen concept by NCBI identifier
- **Clinical Metadata:** Access concept names, definitions, synonyms, identifiers, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Normalize disease and condition concepts in research workflows
- Find clinical concepts associated with genes or inherited disorders
- Enrich datasets with standardized medical terminology and synonyms
- Connect medical concepts with ClinVar, GTR, Gene, dbVar, or publication records
- Support clinical-research search and annotation workflows

> MedGen records are reference information and should not be used as a standalone clinical diagnosis.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find MedGen concepts; required for `search` |
| `id` | `string` | Conditional | — | NCBI MedGen identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as a disease, condition, gene, or research term:

```json
{
  "operation": "search",
  "query": "BRCA1"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI MedGen identifier:

```json
{
  "operation": "getById",
  "id": "1850577"
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
| `success` | `object` or `array` | MedGen concept record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "1850577",
      "concept": "Medical condition",
      "source": "NCBI MedGen"
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
  "error": "NCBI MedGen request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search MedGen Concepts

```json
{
  "operation": "search",
  "query": "BRCA1"
}
```

### Retrieve a MedGen Concept by ID

```json
{
  "operation": "getById",
  "id": "1850577"
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

### Dynamic MedGen Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "cystic fibrosis"
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
title: Search and retrieve NCBI MedGen concepts
```

### Common Patterns

- **Concept Search:** Manual Trigger → NCBI MedGen → Log
- **Concept Lookup:** Manual Trigger → NCBI MedGen → Log
- **Variant Annotation:** NCBI MedGen → NCBI ClinVar → Database
- **Genetic-Test Research:** NCBI MedGen → NCBI GTR → Function
- **Dynamic Search:** Input Data → NCBI MedGen → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a disease, condition, gene, symptom, keyword, or supported NCBI search expression.

#### MedGen ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI MedGen identifier.

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
| Missing ID | `getById` was selected without `id` | Provide a MedGen identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI ClinVar](./ncbi-clinvar.md) - Search clinical variant records
- [NCBI GTR](./ncbi-gtr.md) - Search genetic test records
- [NCBI Gene](./ncbi-gene.md) - Retrieve gene records and annotations
- [NCBI dbVar](./ncbi-dbvar.md) - Search structural variation records
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial documentation and workflow examples for MedGen search and lookup |

<!-- /SECTION: changelog -->
