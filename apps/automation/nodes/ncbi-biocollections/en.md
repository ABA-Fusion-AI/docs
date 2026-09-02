---
node_id: "ncbi-biocollections"
title: "NCBI BioCollections"
description: "Search and retrieve metadata for biological collections from the NCBI BioCollections database."
category: "Healthcare & Life Sciences"
subcategory: "NCBI"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - ncbi
  - biocollections
  - biodiversity
  - biological-collections
  - museums
  - herbaria
  - bioinformatics
related_nodes:
  - ncbi-assembly
  - pub-med-search
  - pmc
  - uni-prot-kb
---

<!-- SECTION: header -->
# NCBI BioCollections

> **Category:** Healthcare & Life Sciences | **Subcategory:** NCBI | **Type:** Action Node

Search and retrieve metadata for biological collections, museums, herbaria, and culture collections from the NCBI BioCollections database.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **NCBI BioCollections** node provides access to curated metadata about biological collections. It supports searching collection records and retrieving a specific record by identifier, helping workflows connect specimen, biodiversity, taxonomy, and research data.

### Key Features

- **Collection Search:** Search records using a text query such as an institution or collection name
- **Record Lookup:** Retrieve a BioCollections record by identifier
- **Collection Metadata:** Access institution codes, collection codes, names, types, locations, and links when available
- **NCBI Integration:** Work with NCBI’s biological data resources
- **Optional API Key:** Use an NCBI API key for enhanced request rates where supported
- **Dynamic Input:** Override operation, query, identifier, and API key from incoming workflow data
- **Error Routing:** Route invalid requests, rate limits, and API failures to the error output

### Use Cases

- Find museums, herbaria, and culture collections by name or code
- Retrieve collection metadata for specimen-data workflows
- Enrich biodiversity and taxonomy records
- Validate institution and collection codes
- Connect collection information to genomic or biomedical research pipelines
- Build collection directories and data-quality reports

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `search` | Operation supported by the node: `search` or `getById` |
| `query` | `string` | Conditional | — | Search term for `search`, such as `ATCC` or an institution name |
| `id` | `string` | Conditional | — | BioCollections record identifier required for `getById` |
| `apiKey` | `string` | No | — | Optional NCBI API key for enhanced request limits |

### Search Operation

Set `operation` to `search` and provide a query:

```json
{
  "operation": "search",
  "query": "ATCC"
}
```

Queries can use collection names, institution names, institution codes, collection codes, or supported BioCollections search syntax.

### Get-by-ID Operation

Set `operation` to `getById` and provide the record identifier:

```json
{
  "operation": "getById",
  "id": "7496"
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

The example workflow contains only the placeholder `tap your API key here`, not a real key. Do not commit an actual NCBI API key in workflow files.

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
| `success` | `object` or `array` | BioCollections record or search response returned by NCBI |

### Search Output Example

```json
{
  "results": [
    {
      "id": "7496",
      "institution_code": "ATCC",
      "collection_code": "ATCC",
      "institution_name": "American Type Culture Collection",
      "collection_type": "culture collection",
      "country": "United States"
    }
  ]
}
```

The exact fields depend on the selected operation and the NCBI response.

### Error Output

Invalid operations, missing query values, unknown record IDs, rate limits, network failures, and NCBI API errors are routed to the error output.

```json
{
  "success": false,
  "error": "NCBI BioCollections request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Search by Collection Name or Code

```json
{
  "operation": "search",
  "query": "ATCC"
}
```

### Retrieve a Collection by ID

```json
{
  "operation": "getById",
  "id": "7496"
}
```

### Search with an API Key

```json
{
  "operation": "search",
  "apiKey": "{{secrets.ncbiApiKey}}",
  "query": "ATCC"
}
```

### Dynamic BioCollections Lookup

A previous node can provide the operation and search value dynamically:

```json
{
  "operation": "search",
  "query": "culture collection"
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
title: Search and retrieve NCBI BioCollections records
```

### Common Patterns

- **Collection Search:** Manual Trigger → NCBI BioCollections → Log
- **Record Lookup:** Manual Trigger → NCBI BioCollections → Log
- **Research Enrichment:** NCBI BioCollections → NCBI Assembly → Database
- **Code Validation:** Input Data → NCBI BioCollections → Function
- **Collection Directory:** NCBI BioCollections → Function → Report

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Search query is missing

**Cause:** The `query` parameter was not provided for the `search` operation.

**Solution:** Provide a collection name, institution name, collection code, institution code, or supported search expression.

#### Record ID is missing

**Cause:** The `id` parameter was not provided for the `getById` operation.

**Solution:** Provide a valid NCBI BioCollections record identifier.

#### Record ID is invalid

**Cause:** The identifier does not match an available BioCollections record.

**Solution:** Search first, select a returned record ID, and use it with `getById`.

#### API key is rejected

**Cause:** The key is invalid, expired, revoked, or passed in an unsupported way.

**Solution:** Verify the key in the NCBI account settings and reference it through `{{secrets.ncbiApiKey}}`.

#### Rate limit exceeded

**Cause:** The workflow sent requests faster than allowed by the NCBI service.

**Solution:** Reduce request frequency, add a delay, or configure an NCBI API key where supported.

#### No collection found

**Cause:** The query does not match a BioCollections record or uses unsupported search syntax.

**Solution:** Try a broader query or use the supported institution, collection, or name search formats.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 400` | Invalid operation, query, or ID | Verify `operation`, `query`, and `id` |
| `HTTP 401` | Invalid API key | Configure a valid secret-backed key or remove it for public access |
| `HTTP 403` | Access or policy restriction | Check NCBI service permissions and usage policy |
| `HTTP 404` | Collection record not found | Verify the record ID or search again |
| `HTTP 429` | Rate limit exceeded | Slow requests or use an API key where supported |
| `HTTP 5xx` | NCBI service failure | Retry after a short delay |
| `Network error` | Connection failure | Check connectivity and NCBI availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [NCBI Assembly](../ncbi-assembly/en.md) - Retrieve genome assembly records
- [PubMed Search](../pub-med-search/en.md) - Search biomedical publications
- [PMC](../pmc/en.md) - Retrieve biomedical literature from PubMed Central
- [UniProt Knowledgebase](../uni-prot-kb/en.md) - Query protein and biological data

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation |

<!-- /SECTION: changelog -->
