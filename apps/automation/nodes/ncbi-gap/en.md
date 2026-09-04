---
node_id: "ncbi-gap"
title: "NCBI GAP"
description: "Search and retrieve study and phenotype information from the NCBI dbGaP database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - ncbi
  - gap
  - dbgap
  - genomics
  - genetics
  - bioinformatics
  - research
  - life-sciences
related_nodes:
  - ncbi-bioproject
  - ncbi-biosample
  - ncbi-clinvar
  - ncbi-genome
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI GAP

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve study and phenotype information from the National Center for Biotechnology Information (NCBI) database of Genotypes and Phenotypes (dbGaP).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI GAP** node provides workflow access to public study metadata and related records from dbGaP. Use it to search studies by disease, phenotype, gene, or research term, or retrieve a specific record by identifier.

### Key Features

- **Study Search:** Find dbGaP records using a disease, phenotype, gene, study term, or NCBI search expression
- **Record Lookup:** Retrieve a specific dbGaP record by NCBI identifier
- **Study Metadata:** Access study, phenotype, organism, accession, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Find genomic studies associated with diseases or phenotypes
- Retrieve study metadata for genetics and bioinformatics workflows
- Enrich research catalogs with public dbGaP study information
- Connect genotype and phenotype studies with BioProject, BioSample, Genome, or ClinVar data
- Support research discovery before requesting access to controlled datasets

> dbGaP may contain controlled-access data. This node provides NCBI records and metadata; it does not bypass dbGaP authorization or data-use requirements.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find dbGaP records; required for `search` |
| `id` | `string` | Conditional | — | NCBI dbGaP identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as a disease, phenotype, gene, or research term:

```json
{
  "operation": "search",
  "query": "Alzheimer"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI dbGaP identifier:

```json
{
  "operation": "getById",
  "id": "2012608"
}
```

### API and Authentication

NCBI public services can generally be used without an API key, subject to rate limits. An NCBI API key can provide enhanced access for supported services. An API key does not grant access to controlled dbGaP data.

Store the key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.ncbiApiKey}}"
}
```

The example workflow contains empty API-key values, not a real key. Do not commit an actual NCBI API key in workflow files.

### Access Requirements

Access to controlled dbGaP datasets requires the appropriate institutional approvals and NCBI authorization. Follow dbGaP data-use policies for any downstream data access.

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
| `success` | `object` or `array` | dbGaP record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "2012608",
      "query": "Alzheimer",
      "source": "NCBI dbGaP"
    }
  ]
}
```

The exact fields depend on the selected operation and the response returned by NCBI.

### Error Output

Invalid operations, missing query or ID values, rate limits, network failures, authorization issues, and NCBI API errors are routed to the error output.

```json
{
  "success": false,
  "error": "NCBI GAP request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search dbGaP Studies

```json
{
  "operation": "search",
  "query": "Alzheimer"
}
```

### Retrieve a dbGaP Record by ID

```json
{
  "operation": "getById",
  "id": "2012608"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "Alzheimer"
}
```

### Dynamic dbGaP Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "type 2 diabetes phenotype"
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
title: Search and retrieve NCBI dbGaP records
```

### Common Patterns

- **Study Search:** Manual Trigger → NCBI GAP → Log
- **Record Lookup:** Manual Trigger → NCBI GAP → Log
- **Research Enrichment:** NCBI GAP → Database or Function
- **Cross-Database Research:** NCBI GAP → NCBI ClinVar or NCBI BioProject → Log
- **Dynamic Search:** Input Data → NCBI GAP → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a disease, phenotype, gene, keyword, or supported NCBI search expression.

#### dbGaP ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI dbGaP identifier.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency, use batching where appropriate, or configure an NCBI API key through Fusion’s secret system.

#### Access is restricted

**Cause:** The requested dbGaP resource requires authorization or controlled-data access approval.

**Solution:** Verify that you are requesting public metadata and follow the dbGaP access process for controlled datasets.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|------|----------|
| Missing query | `search` was selected without `query` | Provide a search query |
| Missing ID | `getById` was selected without `id` | Provide a dbGaP identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Authorization or HTTP 401/403 | Resource requires authorization | Verify access permissions and dbGaP approvals |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI BioProject](./ncbi-bioproject.md) - Work with study and project records
- [NCBI BioSample](./ncbi-biosample.md) - Work with biological sample records
- [NCBI ClinVar](./ncbi-clinvar.md) - Search clinical variant records
- [NCBI Genome](./ncbi-genome.md) - Retrieve genome records
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial documentation and workflow examples for dbGaP search and lookup |

<!-- /SECTION: changelog -->
