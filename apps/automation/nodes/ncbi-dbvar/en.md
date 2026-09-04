---
node_id: "ncbi-dbvar"
title: "NCBI dbVar"
description: "Search and retrieve structural variation records from the NCBI dbVar database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - ncbi
  - dbvar
  - structural-variation
  - genetics
  - genomics
  - bioinformatics
  - research
  - life-sciences
related_nodes:
  - ncbi-clinvar
  - ncbi-gene
  - ncbi-genome
  - ncbi-bioproject
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI dbVar

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve structural variation records from the National Center for Biotechnology Information (NCBI) dbVar database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI dbVar** node provides workflow access to genomic structural variation records and related metadata. Use it to search records by organism, variant, accession, gene, or research term, or retrieve a specific record by identifier.

### Key Features

- **Variation Search:** Find structural-variation records using an organism, gene, variant, accession, or NCBI search expression
- **Record Lookup:** Retrieve a specific dbVar record by NCBI identifier
- **Variation Metadata:** Access variant, organism, accession, study, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Explore structural variants associated with an organism or gene
- Retrieve variation metadata for genomics and bioinformatics workflows
- Enrich variant catalogs with public NCBI dbVar records
- Connect structural-variation data with ClinVar, Gene, Genome, or BioProject records
- Support comparative-genomics and variant-annotation research

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find dbVar records; required for `search` |
| `id` | `string` | Conditional | — | NCBI dbVar identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as an organism, gene, variant, accession, or research term:

```json
{
  "operation": "search",
  "query": "Homo sapiens[Organism]"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI dbVar identifier:

```json
{
  "operation": "getById",
  "id": "57746192"
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
| `success` | `object` or `array` | dbVar record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "57746192",
      "organism": "Homo sapiens",
      "source": "NCBI dbVar"
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
  "error": "NCBI dbVar request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search dbVar by Organism

```json
{
  "operation": "search",
  "query": "Homo sapiens[Organism]"
}
```

### Retrieve a dbVar Record by ID

```json
{
  "operation": "getById",
  "id": "57746192"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "Homo sapiens[Organism]"
}
```

### Dynamic dbVar Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "BRCA1 structural variation"
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
title: Search and retrieve NCBI dbVar records
```

### Common Patterns

- **Variation Search:** Manual Trigger → NCBI dbVar → Log
- **Record Lookup:** Manual Trigger → NCBI dbVar → Log
- **Variant Enrichment:** NCBI dbVar → NCBI ClinVar → Database
- **Genome Research:** NCBI Genome → NCBI dbVar → Function
- **Dynamic Search:** Input Data → NCBI dbVar → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide an organism, gene, variant, accession, keyword, or supported NCBI search expression.

#### dbVar ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI dbVar identifier.

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
| Missing ID | `getById` was selected without `id` | Provide a dbVar identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI ClinVar](./ncbi-clinvar.md) - Search clinical variant records
- [NCBI Gene](./ncbi-gene.md) - Retrieve gene records and annotations
- [NCBI Genome](./ncbi-genome.md) - Retrieve genome records
- [NCBI BioProject](./ncbi-bioproject.md) - Work with study and project records
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial documentation and workflow examples for dbVar search and lookup |

<!-- /SECTION: changelog -->
