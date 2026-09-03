---
node_id: "ncbi-clinvar"
title: "NCBI ClinVar"
description: "Search and retrieve clinical variant records from the NCBI ClinVar database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-03"
author: "Fusion Team"
tags:
  - ncbi
  - clinvar
  - clinical-variants
  - genetics
  - genomics
  - bioinformatics
  - life-sciences
related_nodes:
  - ncbi-bioproject
  - ncbi-biosample
  - ncbi-cdd
  - ncbi-assembly
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI ClinVar

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve clinical variant records from the National Center for Biotechnology Information (NCBI) ClinVar database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI ClinVar** node provides workflow access to records that describe the relationship between genetic variants and human health. Use it to search ClinVar records by gene, variant, condition, or research term, or retrieve a specific record by its NCBI identifier.

### Key Features

- **Variant Search:** Find ClinVar records using a gene, variant, condition, or NCBI search expression
- **Record Lookup:** Retrieve a specific ClinVar record by identifier
- **Clinical Metadata:** Access variant, condition, gene, review, and clinical-significance information returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Investigate clinical variants associated with a gene or disease
- Retrieve ClinVar records for genetic-interpretation workflows
- Enrich patient-safe research datasets with public variant metadata
- Connect variant analysis with BioSample, BioProject, Assembly, or publication data
- Build catalogs of variants and reported clinical significance

> ClinVar records are reference information and should not be treated as a standalone clinical diagnosis. Apply appropriate expert review and current clinical guidance.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find ClinVar records; required for `search` |
| `id` | `string` | Conditional | — | NCBI ClinVar identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as a gene, variant, condition, or search expression:

```json
{
  "operation": "search",
  "query": "BRCA1[gene]"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI ClinVar identifier:

```json
{
  "operation": "getById",
  "id": "4886868"
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
| `success` | `object` or `array` | ClinVar record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "4886868",
      "gene": "BRCA1",
      "clinicalSignificance": "As reported by ClinVar",
      "source": "NCBI ClinVar"
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
  "error": "NCBI ClinVar request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search ClinVar Records by Gene

```json
{
  "operation": "search",
  "query": "BRCA1[gene]"
}
```

### Retrieve a ClinVar Record by ID

```json
{
  "operation": "getById",
  "id": "4886868"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "BRCA1[gene] AND pathogenic"
}
```

### Dynamic ClinVar Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "cystic fibrosis[condition]"
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
title: Search and retrieve NCBI ClinVar records
```

### Common Patterns

- **Variant Search:** Manual Trigger → NCBI ClinVar → Log
- **Variant Lookup:** Manual Trigger → NCBI ClinVar → Log
- **Gene Enrichment:** Input Data → NCBI ClinVar → Database
- **Research Annotation:** NCBI BioSample → NCBI ClinVar → Function
- **Dynamic Search:** Input Data → NCBI ClinVar → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a gene, variant, condition, keyword, or supported NCBI search expression.

#### ClinVar ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI ClinVar identifier.

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
| Missing ID | `getById` was selected without `id` | Provide a ClinVar identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI BioProject](./ncbi-bioproject.md) - Work with study and project records
- [NCBI BioSample](./ncbi-biosample.md) - Work with biological sample records
- [NCBI Conserved Domains](./ncbi-cdd.md) - Retrieve conserved protein-domain records
- [NCBI Assembly](./ncbi-assembly.md) - Retrieve genome assembly records
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-03 | Initial documentation and workflow examples for ClinVar search and lookup |

<!-- /SECTION: changelog -->
