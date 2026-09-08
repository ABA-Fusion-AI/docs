---
node_id: "ncbi-proteinclusters"
title: "NCBI Protein Clusters"
description: "Search and retrieve representative protein cluster records from the NCBI Protein Clusters database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-08"
author: "Fusion Team"
tags:
  - ncbi
  - protein-clusters
  - protein
  - genomics
  - bioinformatics
  - sequence
  - life-sciences
related_nodes:
  - ncbi-protein
  - ncbi-gene
  - ncbi-assembly
  - ncbi-bioproject
  - uni-prot-kb
---

<!-- SECTION: header -->
# NCBI Protein Clusters

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve representative protein cluster records from the National Center for Biotechnology Information (NCBI) Protein Clusters database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI Protein Clusters** node provides workflow access to NCBI’s curated protein cluster records. Use it to search clusters by protein name, function, accession, or organism, or retrieve a specific cluster by identifier.

### Key Features

- **Cluster Search:** Find protein clusters with a text query such as `ATP synthase`
- **Cluster Lookup:** Retrieve a protein cluster by NCBI identifier
- **Representative Records:** Access cluster identifiers, protein descriptions, organism information, and related metadata returned by NCBI
- **Optional API Key:** Use an NCBI API key for enhanced request limits where supported
- **Dynamic Input:** Override operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, network failures, and API errors to the error output

### Use Cases

- Group related proteins for comparative genomics
- Find homologous or functionally related protein records
- Enrich gene, assembly, BioProject, or protein workflows
- Build protein-family and sequence-analysis catalogs
- Connect NCBI cluster data with UniProtKB or downstream bioinformatics processing

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Text or NCBI search expression used to find protein clusters; required for `search` |
| `id` | `string` | Conditional | — | NCBI Protein Cluster identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a protein name, function, accession, organism, or other supported query:

```json
{
  "operation": "search",
  "apiKey": "",
  "query": "ATP synthase"
}
```

### Get-by-ID Operation

Set `operation` to `getById` and provide the NCBI Protein Cluster identifier:

```json
{
  "operation": "getById",
  "apiKey": "",
  "id": "6115420"
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
| `success` | `object` or `array` | Protein cluster record or search response returned by NCBI |

### Success Output Example

```json
{
  "results": [
    {
      "uid": "6115420",
      "title": "ATP synthase protein cluster",
      "source": "NCBI Protein Clusters"
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
  "error": "NCBI Protein Clusters request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search Protein Clusters

```json
{
  "operation": "search",
  "query": "ATP synthase"
}
```

### Retrieve a Protein Cluster by ID

```json
{
  "operation": "getById",
  "id": "6115420"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "ATP synthase"
}
```

### Dynamic Protein Cluster Search

A previous node can provide the operation and query dynamically:

```json
{
  "operation": "search",
  "query": "ribosomal protein"
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
title: Search and retrieve NCBI Protein Cluster records
```

### Common Patterns

- **Cluster Search:** Manual Trigger → NCBI Protein Clusters → Log
- **Cluster Lookup:** Manual Trigger → NCBI Protein Clusters → Log
- **Sequence Enrichment:** NCBI Protein Clusters → NCBI Protein → Database
- **Genomics Research:** NCBI Gene or BioProject → NCBI Protein Clusters → Report
- **Dynamic Search:** Input Data → NCBI Protein Clusters → Function

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a protein name, function, accession, organism, or supported NCBI search expression.

#### Cluster ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI Protein Cluster identifier.

#### API request is rate-limited

**Cause:** Request volume exceeded the applicable NCBI limit.

**Solution:** Reduce request frequency or configure an NCBI API key through Fusion’s secret system.

#### NCBI request failed

**Cause:** The NCBI service, network, or request parameters are unavailable or invalid.

**Solution:** Verify the operation and required parameter, check the NCBI service status, and retry with a smaller request rate.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| Missing query | `search` was selected without `query` | Provide a protein cluster search query |
| Missing ID | `getById` was selected without `id` | Provide a Protein Cluster identifier |
| Rate limit or HTTP 429 | Too many requests | Slow down requests or use an API key |
| Network-related error | NCBI endpoint unavailable | Check connectivity and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI Protein](./ncbi-protein.md) - Retrieve individual protein records
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
| 1.0.0 | 2026-09-08 | Initial documentation and workflow examples for Protein Clusters search and lookup |

<!-- /SECTION: changelog -->
