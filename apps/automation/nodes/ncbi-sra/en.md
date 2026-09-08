---
node_id: "ncbi-sra"
title: "NCBI SRA"
description: "Search and retrieve sequencing run and study records from the NCBI Sequence Read Archive."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - ncbi
  - sra
  - sequencing
  - genomics
  - bioinformatics
  - research
  - life-sciences
related_nodes:
  - ncbi-bioproject
  - ncbi-biosample
  - ncbi-assembly
  - ncbi-protein
  - pub-med-search
---

<!-- SECTION: header -->
# NCBI SRA

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve sequencing run and study records from the National Center for Biotechnology Information (NCBI) Sequence Read Archive (SRA).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI SRA** node provides workflow access to metadata for high-throughput sequencing experiments. Use it to search SRA records by accession, organism, study, or another supported NCBI query, or retrieve a specific record by identifier.

### Key Features

- **SRA Search:** Find sequencing records with an accession, organism, study term, or NCBI search expression
- **Record Lookup:** Retrieve an SRA record by NCBI identifier
- **Sequencing Metadata:** Access study, sample, experiment, run, platform, and organism metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Find sequencing runs for a research project or organism
- Retrieve metadata for an SRA accession before downstream analysis
- Connect sequencing records with BioProject and BioSample data
- Build genomic-data catalogs and experiment inventories
- Enrich bioinformatics workflows with public sequencing metadata

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find SRA records; required for `search` |
| `id` | `string` | Conditional | — | NCBI SRA identifier or UID required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide an accession, organism, study term, or another supported query:

```json
{
  "operation": "search",
  "query": "SRR390728[Accession]"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI SRA identifier:

```json
{
  "operation": "getById",
  "id": "90134"
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
| `success` | `object` or `array` | SRA record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "90134",
      "accession": "SRR390728",
      "title": "Sequencing run metadata",
      "organism": "Homo sapiens"
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
  "error": "NCBI SRA request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search by SRA Accession

```json
{
  "operation": "search",
  "query": "SRR390728[Accession]"
}
```

### Retrieve an SRA Record by ID

```json
{
  "operation": "getById",
  "id": "90134"
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

### Dynamic SRA Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "metagenome[All Fields]"
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
title: Search and retrieve NCBI SRA records
```

### Common Patterns

- **SRA Search:** Manual Trigger → NCBI SRA → Log
- **Record Lookup:** Manual Trigger → NCBI SRA → Log
- **Study Enrichment:** NCBI BioProject → NCBI SRA → Database
- **Sample Analysis:** NCBI BioSample → NCBI SRA → Function
- **Dynamic Search:** Input Data → NCBI SRA → Report

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide an SRA accession, organism, study term, or supported NCBI search expression.

#### SRA ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI SRA UID or identifier.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency or configure an NCBI API key through Fusion’s secret system.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| Missing query | `search` was selected without `query` | Provide an SRA search query |
| Missing ID | `getById` was selected without `id` | Provide an SRA identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI BioProject](./ncbi-bioproject.md) - Retrieve project metadata
- [NCBI BioSample](./ncbi-biosample.md) - Retrieve sample records
- [NCBI Assembly](./ncbi-assembly.md) - Retrieve genome assembly records
- [NCBI Protein](./ncbi-protein.md) - Retrieve protein records
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation and workflow examples for SRA search and lookup |

<!-- /SECTION: changelog -->
