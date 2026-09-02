---
node_id: "ncbi-assembly"
title: "NCBI Assembly"
description: "Search and retrieve genome assembly records from the NCBI Assembly database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - ncbi
  - assembly
  - genomics
  - genome
  - bioinformatics
  - life-sciences
related_nodes:
  - pub-med-search
  - pmc
  - uni-prot-kb
  - http-request
---

<!-- SECTION: header -->
# NCBI Assembly

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve genome assembly records from the National Center for Biotechnology Information (NCBI) Assembly database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI Assembly** node retrieves information about assembled genomes and related assembly records. It is useful for identifying genome builds, examining assembly metadata, and connecting genomic reference information to downstream research workflows.

### Key Features

- **Assembly Lookup:** Retrieve an assembly record by its NCBI identifier
- **Genome Metadata:** Access assembly and genome information returned by NCBI
- **NCBI Integration:** Work with NCBI’s public biological data services
- **Optional API Key:** Use an NCBI API key to access enhanced request rates where supported
- **Dynamic Input:** Override configured operation and identifier values from incoming workflow data
- **Research Ready:** Pass assembly records to bioinformatics, reporting, or data-processing nodes
- **Error Routing:** Route invalid requests, rate limits, and API failures to the error output

### Use Cases

- Retrieve metadata for a reference genome assembly
- Identify assembly information for a BioProject or accession
- Enrich genomic research records with NCBI data
- Build genome comparison and bioinformatics workflows
- Validate assembly identifiers before downstream analysis
- Connect NCBI assembly data to PubMed or PMC research workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `getById` | Assembly operation supported by the node. The example uses `getById`. |
| `id` | `string` | Yes for `getById` | — | NCBI Assembly identifier, UID, or supported accession value |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Assembly Identifier

The `id` value identifies the assembly to retrieve. The example uses:

```text
11968211
```

Depending on the configured operation and NCBI endpoint, supported identifiers may include an NCBI UID or an assembly accession such as `GCF_000001405.40`.

### API and Authentication

NCBI public services can be used without an API key, subject to stricter rate limits. An NCBI API key provides enhanced access for supported NCBI APIs.

Store the key in Fusion’s secret system and reference it dynamically:

```json
{
  "apiKey": "{{secrets.ncbiApiKey}}"
}
```

The example workflow contains only the placeholder `tap your NCBI API key here`, not a real key. Do not commit an actual NCBI API key in workflow files.

### Request Limits

NCBI rate limits depend on the API service and whether an API key is supplied. Respect NCBI usage policies and avoid high-frequency requests without an appropriate key and request strategy.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional dynamic input containing `operation`, `id`, and `apiKey` overrides |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | NCBI Assembly record or API response for the requested operation |

### Success Output Example

```json
{
  "assembly": {
    "id": "11968211",
    "accession": "GCF_000001405.40",
    "organism": "Homo sapiens",
    "assembly_name": "GRCh38.p14",
    "assembly_level": "Chromosome"
  }
}
```

The exact fields depend on the NCBI service and the assembly record returned.

### Error Output

Invalid identifiers, missing required fields, rate limits, unavailable NCBI services, network failures, and API errors are routed to the error output.

```json
{
  "success": false,
  "error": "NCBI Assembly request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Retrieve an Assembly by ID

```json
{
  "operation": "getById",
  "id": "11968211"
}
```

### Retrieve an Assembly with an API Key

```json
{
  "operation": "getById",
  "id": "11968211",
  "apiKey": "{{secrets.ncbiApiKey}}"
}
```

### Retrieve by Accession

When supported by the selected operation, use an NCBI assembly accession:

```json
{
  "operation": "getById",
  "id": "GCF_000001405.40"
}
```

### Dynamic Assembly Lookup

A previous node can provide the identifier dynamically:

```json
{
  "id": "GCA_000001405.29"
}
```

Keep the API key in Fusion’s secret system even when the assembly ID comes from incoming data.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve an NCBI genome assembly record
```

### Common Patterns

- **Assembly Lookup:** Manual Trigger → NCBI Assembly → Log
- **Research Enrichment:** PubMed or PMC → NCBI Assembly → Database
- **Dynamic Identifier:** Input Data → NCBI Assembly → Function
- **Genome Reporting:** NCBI Assembly → Function → Report
- **Rate-Controlled Retrieval:** Schedule → NCBI Assembly → Storage

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Assembly ID is missing

**Cause:** The required `id` parameter was not provided for the selected operation.

**Solution:** Provide an NCBI Assembly UID or supported accession value.

#### Assembly ID is invalid

**Cause:** The identifier does not match an available NCBI Assembly record or uses an unsupported format.

**Solution:** Verify the UID or accession in NCBI and use a supported identifier format.

#### Rate limit exceeded

**Cause:** The workflow sent requests faster than the allowed rate for the NCBI service.

**Solution:** Reduce request frequency, add a delay, or configure an NCBI API key where supported.

#### API key is rejected

**Cause:** The API key is invalid, expired, revoked, or passed in an unsupported way.

**Solution:** Verify the key in the NCBI account settings and reference it through `{{secrets.ncbiApiKey}}`.

#### No assembly record found

**Cause:** The requested UID or accession does not identify a record available through the selected service.

**Solution:** Check the identifier and try a current assembly accession.

#### NCBI service unavailable

**Cause:** The NCBI endpoint or network connection is temporarily unavailable.

**Solution:** Retry later and check NCBI service availability.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 400` | Invalid operation or identifier | Verify `operation` and `id` |
| `HTTP 401` | Invalid API key | Configure a valid secret-backed key or remove it for public access |
| `HTTP 403` | Access or policy restriction | Check NCBI service permissions and usage policy |
| `HTTP 404` | Assembly record not found | Verify the UID or accession |
| `HTTP 429` | Rate limit exceeded | Slow requests or use an API key where supported |
| `HTTP 5xx` | NCBI service failure | Retry after a short delay |
| `Network error` | Connection failure | Check connectivity and NCBI availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [PubMed Search](../pub-med-search/en.md) - Search biomedical publications
- [PMC](../pmc/en.md) - Retrieve biomedical literature from PubMed Central
- [UniProt Knowledgebase](../uni-prot-kb/en.md) - Query protein and genome-related data
- [HTTP Request](../http-request/en.md) - Send generic HTTP requests

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation |

<!-- /SECTION: changelog -->
