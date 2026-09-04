---
node_id: "ncbi-genome"
title: "NCBI Genome"
description: "Search and retrieve genome records from the NCBI Genome database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-04"
author: "Fusion Team"
tags:
  - ncbi
  - genome
  - genomics
  - genome-assembly
  - bioinformatics
  - research
  - life-sciences
related_nodes:
  - ncbi-assembly
  - ncbi-bioproject
  - ncbi-biosample
  - ncbi-gene
  - ncbi-sra
---

<!-- SECTION: header -->
# NCBI Genome

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve genome records from the National Center for Biotechnology Information (NCBI) Genome database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI Genome** node provides workflow access to genome records and related genomic metadata. Use it to search genomes by organism, accession, strain, or research term, or retrieve a specific genome record by identifier.

### Key Features

- **Genome Search:** Find genome records using an organism, accession, strain, assembly, or NCBI search expression
- **Genome Lookup:** Retrieve a specific genome record by NCBI identifier
- **Genomic Metadata:** Access organism, assembly, accession, sequencing, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override the operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Find reference or representative genomes for an organism
- Retrieve genome metadata for comparative-genomics workflows
- Enrich BioProject, BioSample, or sequencing datasets with genome information
- Locate genomes by accession, strain, or assembly characteristics
- Build genome catalogs and downstream bioinformatics workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find genome records; required for `search` |
| `id` | `string` | Conditional | — | NCBI Genome identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as an organism, accession, strain, or research term:

```json
{
  "operation": "search",
  "query": "Escherichia coli[Organism]"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI Genome identifier:

```json
{
  "operation": "getById",
  "id": "51"
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

The example workflow contains only the placeholder `tap your apikey here`, not a real key. Do not commit an actual NCBI API key in workflow files.

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
| `success` | `object` or `array` | Genome record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "51",
      "organism": "Escherichia coli",
      "source": "NCBI Genome"
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
  "error": "NCBI Genome request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search Genomes by Organism

```json
{
  "operation": "search",
  "query": "Escherichia coli[Organism]"
}
```

### Retrieve a Genome by ID

```json
{
  "operation": "getById",
  "id": "51"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "Escherichia coli[Organism]"
}
```

### Dynamic Genome Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "Homo sapiens reference genome"
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
title: Search and retrieve NCBI Genome records
```

### Common Patterns

- **Genome Search:** Manual Trigger → NCBI Genome → Log
- **Genome Lookup:** Manual Trigger → NCBI Genome → Log
- **Assembly Enrichment:** NCBI Genome → NCBI Assembly → Database
- **Project Enrichment:** NCBI BioProject → NCBI Genome → Function
- **Dynamic Search:** Input Data → NCBI Genome → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide an organism, accession, strain, assembly, keyword, or supported NCBI search expression.

#### Genome ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI Genome identifier.

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
| Missing ID | `getById` was selected without `id` | Provide a Genome identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI Assembly](./ncbi-assembly.md) - Retrieve genome assembly records
- [NCBI BioProject](./ncbi-bioproject.md) - Work with study and project records
- [NCBI BioSample](./ncbi-biosample.md) - Work with biological sample records
- [NCBI Gene](./ncbi-gene.md) - Retrieve gene records and annotations
- [NCBI SRA](./ncbi-sra.md) - Retrieve sequencing read archive data

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-04 | Initial documentation and workflow examples for Genome search and lookup |

<!-- /SECTION: changelog -->
