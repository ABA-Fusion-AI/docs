---
node_id: "ncbi-structure"
title: "NCBI Structure"
description: "Search and retrieve molecular structure records from the NCBI Structure database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - ncbi
  - structure
  - pdb
  - molecular-structure
  - proteins
  - bioinformatics
  - life-sciences
related_nodes:
  - ncbi-protein
  - ncbi-gene
  - ncbi-assembly
  - ncbi-bioproject
  - uni-prot-kb
---

<!-- SECTION: header -->
# NCBI Structure

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve molecular structure records from the National Center for Biotechnology Information (NCBI) Structure database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI Structure** node provides workflow access to molecular structure records indexed by NCBI. Use it to search structures by PDB accession, molecule, organism, or another supported NCBI query, or retrieve a specific structure record by identifier.

### Key Features

- **Structure Search:** Find records with a PDB accession, molecule name, organism, or NCBI search expression
- **Record Lookup:** Retrieve a structure record by NCBI identifier
- **Molecular Metadata:** Access structure identifiers, molecule descriptions, source-organism information, and related metadata returned by NCBI
- **PDB Query Support:** Use field-qualified queries such as `1TUP[PDB Accession]`
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Find molecular structures for proteins and other biological macromolecules
- Retrieve structure metadata for structural-biology workflows
- Enrich protein, gene, Assembly, or UniProtKB records with structure information
- Build molecular-structure catalogs and research reports
- Connect PDB-related records to downstream bioinformatics processing

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find structure records; required for `search` |
| `id` | `string` | Conditional | — | NCBI Structure identifier or UID required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a PDB accession, molecule, organism, or another supported query:

```json
{
  "operation": "search",
  "query": "1TUP[PDB Accession]"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI Structure identifier:

```json
{
  "operation": "getById",
  "id": "242790"
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
| `success` | `object` or `array` | Molecular structure record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "242790",
      "pdbAccession": "1TUP",
      "title": "Molecular structure record",
      "source": "NCBI Structure"
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
  "error": "NCBI Structure request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search by PDB Accession

```json
{
  "operation": "search",
  "query": "1TUP[PDB Accession]"
}
```

### Retrieve a Structure by ID

```json
{
  "operation": "getById",
  "id": "242790"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "hemoglobin[All Fields]"
}
```

### Dynamic Structure Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "kinase[All Fields]"
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
title: Search and retrieve NCBI Structure records
```

### Common Patterns

- **Structure Search:** Manual Trigger → NCBI Structure → Log
- **Record Lookup:** Manual Trigger → NCBI Structure → Log
- **Protein Enrichment:** NCBI Protein → NCBI Structure → Database
- **Research Workflow:** NCBI Gene or UniProtKB → NCBI Structure → Report
- **Dynamic Search:** Input Data → NCBI Structure → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a PDB accession, molecule, organism, or supported NCBI search expression.

#### Structure ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI Structure UID or identifier.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency or configure an NCBI API key through Fusion’s secret system.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| Missing query | `search` was selected without `query` | Provide a structure search query |
| Missing ID | `getById` was selected without `id` | Provide a Structure identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI Protein](./ncbi-protein.md) - Retrieve protein records
- [NCBI Gene](./ncbi-gene.md) - Retrieve gene records
- [NCBI Assembly](./ncbi-assembly.md) - Retrieve genome assembly records
- [NCBI BioProject](./ncbi-bioproject.md) - Retrieve project metadata
- [UniProtKB](./uni-prot-kb.md) - Retrieve protein annotations and sequences

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-08 | Initial documentation and workflow examples for Structure search and lookup |

<!-- /SECTION: changelog -->
