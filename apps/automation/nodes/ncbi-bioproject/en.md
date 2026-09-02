---
node_id: "ncbi-bioproject"
title: "NCBI BioProject"
description: "Search and retrieve study metadata from the NCBI BioProject database. "
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - ncbi
  - bioproject
  - genomics
  - bioinformatics
  - research
  - life-sciences
related_nodes:
  - ncbi-assembly
  - ncbi-biosample
  - ncbi-sra
  - pub-med-search
  - uni-prot-kb
---

<!-- SECTION: header -->
# NCBI BioProject

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve study metadata from the National Center for Biotechnology Information (NCBI) BioProject database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI BioProject** node provides workflow access to NCBI study and project records. Use it to find projects with a text query or retrieve a specific project by its NCBI identifier.

### Key Features

- **Project Search:** Find BioProject records with a text query, accession, organism, or study term
- **Project Lookup:** Retrieve a BioProject record by NCBI identifier
- **Study Metadata:** Access project titles, descriptions, organisms, accessions, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Find sequencing and genomics projects by organism or research topic
- Retrieve project metadata for downstream bioinformatics workflows
- Connect BioProject studies with BioSample, SRA, Assembly, or publication records
- Build research catalogs and project-monitoring workflows
- Enrich internal study records with public NCBI metadata

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text used to search BioProject records; required for `search` |
| `id` | `string` | Conditional | — | NCBI BioProject identifier or UID required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query such as a BioProject accession, organism, or research term:

```json
{
  "operation": "search",
  "query": "PRJNA168"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI project identifier:

```json
{
  "operation": "getById",
  "id": "168"
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

The example workflow contains only the placeholder `put your NCBI API key here`, not a real key. Do not commit an actual NCBI API key in workflow files.

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
| `success` | `object` or `array` | BioProject record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "168",
      "accession": "PRJNA168",
      "title": "A sequencing project",
      "description": "Project metadata returned by NCBI"
    }
  ]
}
```

The exact fields depend on the selected operation and the NCBI response.

### Error Output

Invalid operations, missing query or ID values, rate limits, network failures, and NCBI API errors are routed to the error output.

```json
{
  "success": false,
  "error": "NCBI BioProject request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search BioProject Records

```json
{
  "operation": "search",
  "query": "PRJNA168"
}
```

### Retrieve a BioProject by ID

```json
{
  "operation": "getById",
  "id": "168"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "Homo sapiens"
}
```

### Dynamic BioProject Lookup

A previous node can provide the operation and search value dynamically:

```json
{
  "operation": "search",
  "query": "metagenome"
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
title: Search and retrieve NCBI BioProject records
```

### Common Patterns

- **Project Search:** Manual Trigger → NCBI BioProject → Log
- **Project Lookup:** Manual Trigger → NCBI BioProject → Log
- **Research Enrichment:** NCBI BioProject → NCBI BioSample or NCBI SRA → Database
- **Dynamic Search:** Input Data → NCBI BioProject → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a BioProject accession, organism, study term, or supported NCBI search expression.

#### Project ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI BioProject UID or identifier.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency, use batching where appropriate, or configure an NCBI API key through Fusion’s secret system.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| Missing query | `search` was selected without `query` | Provide a search query |
| Missing ID | `getById` was selected without `id` | Provide a BioProject identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI Assembly](./ncbi-assembly.md) - Retrieve genome assembly records
- [NCBI BioSample](./ncbi-biosample.md) - Work with biological sample records
- [NCBI SRA](./ncbi-sra.md) - Retrieve sequencing read archive data
- [PubMed Search](./pub-med-search.md) - Search related biomedical literature
- [UniProtKB](./uni-prot-kb.md) - Retrieve protein and annotation data

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation and workflow examples for BioProject search and lookup |

<!-- /SECTION: changelog -->
