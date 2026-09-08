---
node_id: "ncbi-protein"
title: "NCBI Protein"
description: "Search and retrieve protein records from the NCBI Protein database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - ncbi
  - protein
  - proteomics
  - genomics
  - bioinformatics
  - sequence
  - life-sciences
related_nodes:
  - ncbi-gene
  - ncbi-assembly
  - ncbi-bioproject
  - ncbi-sra
  - uni-prot-kb
---

<!-- SECTION: header -->
# NCBI Protein

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve protein sequence and annotation records from the National Center for Biotechnology Information (NCBI) Protein database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI Protein** node provides workflow access to protein records indexed by NCBI. Use it to search by accession, gene, organism, protein name, or another supported NCBI query, or retrieve a specific protein record by identifier.

### Key Features

- **Protein Search:** Find records with an accession, gene, organism, protein name, or NCBI search expression
- **Record Lookup:** Retrieve a protein record by NCBI identifier
- **Sequence Metadata:** Access protein identifiers, descriptions, organism information, and sequence-related data returned by NCBI
- **Pagination:** Request a specific result page for searches with multiple matches
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override operation, query, identifier, page, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Retrieve protein records for sequence-analysis workflows
- Find proteins by accession, gene, organism, or functional description
- Enrich BioProject, Gene, Assembly, or SRA records with protein information
- Build proteomics and bioinformatics catalogs
- Connect NCBI protein data with UniProtKB or downstream reporting workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find protein records; required for `search` |
| `page` | `number` | No | `1` | Result page to request for `search` |
| `id` | `string` | Conditional | — | NCBI Protein identifier or UID required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query. The example searches for a protein accession:

```json
{
  "operation": "search",
  "apiKey": "",
  "query": "P38398[Accession]",
  "page": 1
}
```

Queries can use protein accessions, gene symbols, organism names, protein names, or supported NCBI field expressions.

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI Protein identifier:

```json
{
  "operation": "getById",
  "apiKey": "",
  "id": "728984"
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
| `input` | `object` | Optional dynamic input containing `operation`, `query`, `page`, `id`, and `apiKey` overrides |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | Protein record or search response returned by NCBI |

### Search Output Example

```json
{
  "results": [
    {
      "uid": "728984",
      "accession": "P38398",
      "title": "DNA repair protein BRCA1",
      "organism": "Homo sapiens"
    }
  ],
  "page": 1
}
```

The exact fields depend on the selected operation and the response returned by NCBI.

### Error Output

Invalid operations, missing query or ID values, invalid pages, rate limits, network failures, and NCBI API errors are routed to the error output.

```json
{
  "success": false,
  "error": "NCBI Protein request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search by Protein Accession

```json
{
  "operation": "search",
  "query": "P38398[Accession]",
  "page": 1
}
```

### Retrieve a Protein by ID

```json
{
  "operation": "getById",
  "id": "728984"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "BRCA1[Gene] AND Homo sapiens[Organism]",
  "page": 1
}
```

### Paginate Search Results

```json
{
  "operation": "search",
  "query": "kinase[Title]",
  "page": 2
}
```

### Dynamic Protein Lookup

A previous node can provide the operation and search values dynamically:

```json
{
  "operation": "search",
  "query": "hemoglobin[Title]",
  "page": 1
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
title: Search and retrieve NCBI Protein records
```

### Common Patterns

- **Protein Search:** Manual Trigger → NCBI Protein → Log
- **Record Lookup:** Manual Trigger → NCBI Protein → Log
- **Sequence Enrichment:** NCBI Protein → UniProtKB → Database
- **Genomics Research:** NCBI Gene or BioProject → NCBI Protein → Report
- **Dynamic Search:** Input Data → NCBI Protein → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a protein accession, gene symbol, organism, protein name, or supported NCBI search expression.

#### Protein ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI Protein UID or identifier.

#### Page is invalid

**Cause:** The `page` value is not a positive number or is not supported by the search request.

**Solution:** Use a positive page number, such as `1` or `2`.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency, use pagination carefully, or configure an NCBI API key through Fusion’s secret system.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| Missing query | `search` was selected without `query` | Provide a protein search query |
| Missing ID | `getById` was selected without `id` | Provide a Protein identifier |
| Invalid page | Page is missing, non-positive, or malformed | Use a positive page number |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI Gene](./ncbi-gene.md) - Retrieve gene records
- [NCBI Assembly](./ncbi-assembly.md) - Retrieve genome assembly records
- [NCBI BioProject](./ncbi-bioproject.md) - Retrieve project metadata
- [NCBI SRA](./ncbi-sra.md) - Work with sequencing archive data
- [UniProtKB](./uni-prot-kb.md) - Retrieve protein annotations and sequences

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation and workflow examples for Protein search and lookup |

<!-- /SECTION: changelog -->
