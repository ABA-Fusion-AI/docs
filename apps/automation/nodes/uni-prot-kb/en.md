---
node_id: "uni-prot-kb"
title: "UniProtKB"
description: "Search UniProtKB and retrieve protein sequences, annotations, and functional information."
category: "Healthcare & Life Sciences"
subcategory: "Biology & Life Sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - uniprot
  - uniprotkb
  - protein
  - biology
  - bioinformatics
  - sequence
  - genomics
  - life-sciences
related_nodes:
  - pub-med-search
  - clinical-trials-search
  - bar
  - function
---

<!-- SECTION: header -->
# UniProtKB

> **Category:** Healthcare & Life Sciences | **Subcategory:** Biology & Life Sciences | **Type:** Action Node

Access the UniProt Knowledgebase (UniProtKB) to retrieve protein sequences, functional annotations, names, gene associations, organism information, and related biological data.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **UniProtKB** node provides workflow access to the public UniProt protein knowledgebase. It supports retrieving a single entry by accession, searching with UniProt query syntax, and streaming large result sets.

### Key Features

- **Accession Lookup:** Retrieve one UniProtKB entry by accession, such as `P05067`
- **Protein Search:** Search by gene, organism, protein name, accession, or other supported fields
- **Streaming:** Process large search result sets without requiring one large response
- **Field Selection:** Request only the fields needed by downstream workflow steps
- **Isoform Support:** Include isoform data when supported by the selected stream request
- **Download Mode:** Support download-oriented stream requests when configured
- **No Authentication Required:** The workflow example uses the public UniProt REST API without an API key or access token

### Use Cases

- Retrieve protein annotations for research and analysis
- Find human or organism-specific proteins by gene name
- Build bioinformatics and life-sciences data pipelines
- Enrich experimental records with protein names, sequences, or gene associations
- Export filtered UniProtKB result sets for downstream processing
- Connect protein knowledge to literature, clinical, or computational workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | — | Operation to perform: `getByAccession`, `search`, or `stream` |
| `accession` | `string` | Conditional | — | UniProtKB accession used by `getByAccession`, for example `P05067` |
| `query` | `string` | Conditional | — | UniProt query expression used by `search` or `stream` |
| `sort` | `string` | No | — | Sort expression for search results, for example `accession asc` |
| `fields` | `string` | No | — | Comma-separated UniProtKB return fields, for example `accession,id,length` |
| `includeIsoform` | `boolean` | No | `false` | Include isoform information when supported by the request |
| `download` | `string` | No | `false` | Download-oriented stream setting as demonstrated by the workflow |

Required parameters depend on the selected operation. `getByAccession` requires `accession`; `search` and `stream` require `query`.

### Operation Reference

| Operation | Purpose | Example |
|-----------|---------|---------|
| `getByAccession` | Retrieve one UniProtKB record | `accession: "P05067"` |
| `search` | Search and return matching records | `query: "gene:BRCA1 AND organism_id:9606"` |
| `stream` | Stream matching records, useful for larger result sets | `query: "accession:P05067"` |

### UniProt Query Examples

```text
gene:BRCA1 AND organism_id:9606
accession:P05067
protein_name:kinase
organism_id:9606 AND reviewed:true
```

### API and Authentication

The workflow example contains no `apiKey`, access token, authorization header, or secret reference. UniProt provides public REST endpoints for these operations, so no credential is required for this node configuration.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Optional dynamic accession or query data. An object can provide fields such as `operation`, `accession`, `query`, `fields`, and `sort`. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` or `array` | UniProtKB entry, search result set, or streamed result data depending on the selected operation |

Example accession response shape:

```json
{
  "results": [
    {
      "primaryAccession": "P05067",
      "uniProtkbId": "APP_HUMAN",
      "proteinDescription": {
        "recommendedName": {
          "fullName": {
            "value": "Amyloid-beta precursor protein"
          }
        }
      },
      "organism": {
        "scientificName": "Homo sapiens",
        "taxonId": 9606
      }
    }
  ]
}
```

Search and stream responses may include pagination, total-result metadata, selected fields, or a list of records depending on the operation and request options.

### Error Output

Invalid queries, missing required parameters, unavailable records, malformed field selections, network failures, and upstream API errors are routed to `error`.

```json
{
  "success": false,
  "error": "UniProtKB request failed",
  "operation": "getByAccession",
  "accession": "INVALID"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Retrieve an Entry by Accession

```json
{
  "operation": "getByAccession",
  "accession": "P05067"
}
```

### Search Human BRCA1 Entries

```json
{
  "operation": "search",
  "query": "gene:BRCA1 AND organism_id:9606",
  "sort": "accession asc",
  "fields": "accession,id,protein_name,gene_names,organism_name,length"
}
```

### Stream Selected Fields

```json
{
  "operation": "stream",
  "query": "gene:BRCA1 AND organism_id:9606",
  "fields": "accession,id,protein_name,gene_names,organism_name,length"
}
```

### Stream an Accession with Isoforms

```json
{
  "operation": "stream",
  "query": "accession:P05067",
  "fields": "accession,id,protein_name,length",
  "includeIsoform": true,
  "download": "false"
}
```

### Dynamic Query from a Previous Node

Pass an object through `input`:

```json
{
  "operation": "search",
  "query": "protein_name:kinase AND organism_id:9606",
  "fields": "accession,protein_name,gene_names,length"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve, search, and stream UniProtKB protein data
```

### Common Patterns

- **Single entry:** Manual Trigger → UniProtKB (`getByAccession`) → Log
- **Gene search:** Manual Trigger → UniProtKB (`search`) → Function
- **Large result set:** Trigger → UniProtKB (`stream`) → Storage or analysis node
- **Research enrichment:** UniProtKB → PubMed Search → Report or database
- **Sequence pipeline:** UniProtKB → Function → Sequence analysis node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Accession is required

**Cause:** `getByAccession` was selected without an `accession` value.

**Solution:** Provide a valid UniProtKB accession such as `P05067`.

### Query is required

**Cause:** `search` or `stream` was selected without a `query` value.

**Solution:** Add a UniProt query expression, for example `gene:BRCA1 AND organism_id:9606`.

### No matching entries

**Cause:** The query is too restrictive, uses an unsupported field, or the identifier does not exist.

**Solution:** Test a simpler query first, verify field names, and confirm the accession or organism identifier.

### Invalid fields or sort expression

**Cause:** `fields` or `sort` contains a field name or syntax not accepted by UniProtKB.

**Solution:** Start with known fields such as `accession`, `id`, `protein_name`, `gene_names`, `organism_name`, and `length`, then add fields incrementally.

### Large response or slow stream

**Cause:** The query matches many records or requests more fields than needed.

**Solution:** Narrow the query, request only required fields, and use `stream` for large result sets.

### Authentication question

**Cause:** The workflow appears to expect an API key or access token.

**Solution:** No credential is configured or required in the provided workflow. UniProt's public REST API is used directly.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [PubMed Search](./pub-med-search.md) — Search biomedical literature
- [Clinical Trials Search](./clinical-trials-search.md) — Find clinical trial records
- [Function](./function.md) — Transform UniProtKB results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial documentation |

<!-- /SECTION: changelog -->
