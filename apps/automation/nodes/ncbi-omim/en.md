---
node_id: "ncbi-omim"
title: "NCBI OMIM"
description: "Search and retrieve medical genetics records from the NCBI OMIM database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - ncbi
  - omim
  - genetics
  - genomics
  - medical-genetics
  - bioinformatics
  - life-sciences
related_nodes:
  - ncbi-gene
  - ncbi-clinvar
  - ncbi-medgen
  - ncbi-snp
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI OMIM

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve medical genetics records from the Online Mendelian Inheritance in Man (OMIM) database through NCBI.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI OMIM** node provides workflow access to records about human genes and inherited disorders. Use it to search OMIM records with a text or field-qualified query, or retrieve a specific record by identifier.

### Key Features

- **OMIM Search:** Find records by gene symbol, disease name, phenotype, accession, or OMIM search expression
- **Record Lookup:** Retrieve an OMIM record by identifier
- **Medical Genetics Metadata:** Access record summaries, titles, identifiers, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Find gene-disease and phenotype records for research workflows
- Retrieve OMIM metadata for variant and clinical-genetics analysis
- Enrich gene, ClinVar, MedGen, or literature records
- Build medical-genetics catalogs and research reports
- Connect inherited-disorder information to downstream bioinformatics workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find OMIM records; required for `search` |
| `id` | `string` | Conditional | — | OMIM or NCBI record identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a gene, disorder, phenotype, accession, or field-qualified query:

```json
{
  "operation": "search",
  "query": "BRCA1[Title]"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the OMIM record identifier:

```json
{
  "operation": "getById",
  "id": "113705"
}
```

### API and Authentication

The workflow example does not contain a real API key. Its `apiKey` fields are empty, and the workflow-level `secrets` object is empty. NCBI public services can generally be used without an API key, subject to applicable rate limits; an NCBI API key can provide enhanced access for supported services.

Store the key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.ncbiApiKey}}"
}
```

Do not commit an actual NCBI API key in workflow files. OMIM content and service access may also be subject to OMIM and NCBI terms of use.

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
| `success` | `object` or `array` | OMIM record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "113705",
      "title": "BREAST CANCER 1; BRCA1",
      "accession": "113705",
      "source": "NCBI OMIM"
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
  "error": "NCBI OMIM request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search OMIM Records

```json
{
  "operation": "search",
  "query": "BRCA1[Title]"
}
```

### Retrieve an OMIM Record by ID

```json
{
  "operation": "getById",
  "id": "113705"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "BRCA1[Title]"
}
```

### Dynamic OMIM Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "familial breast cancer"
}
```

Keep the API key in Fusion’s secret system even when the query comes from incoming data.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: test ncbi omim-2026-09-08T11_03_36.800Z.json
title: Search and retrieve NCBI OMIM records
```

### Common Patterns

- **OMIM Search:** Manual Trigger → NCBI OMIM → Log
- **Record Lookup:** Manual Trigger → NCBI OMIM → Log
- **Genetics Enrichment:** NCBI OMIM → NCBI Gene or ClinVar → Database
- **Literature Research:** NCBI OMIM → PubMed Search → Report
- **Dynamic Search:** Input Data → NCBI OMIM → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a gene symbol, disorder, phenotype, accession, or supported NCBI search expression.

#### OMIM ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid OMIM or NCBI record identifier.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency, use batching where appropriate, or configure an NCBI API key through Fusion’s secret system.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| Missing query | `search` was selected without `query` | Provide an OMIM search query |
| Missing ID | `getById` was selected without `id` | Provide an OMIM record identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI Gene](./ncbi-gene.md) - Retrieve gene records
- [NCBI ClinVar](./ncbi-clinvar.md) - Work with clinical variant records
- [NCBI MedGen](./ncbi-medgen.md) - Retrieve medical-genetics concepts
- [NCBI SNP](./ncbi-snp.md) - Retrieve variant records
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation and workflow examples for OMIM search and lookup |

<!-- /SECTION: changelog -->
