---
node_id: "bio-api"
title: "BioAPI"
description: "A powerful abstraction of genomics databases. BioAPI is part of the Multiomix project. Access gene information, expression data, pathways, ontologies, and drug-gene interactions."
category: "healthcare-life-sciences"
subcategory: "biology-life-sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-09-03"
author: "Fusion Team"
tags:
  - "bioapi"
  - "genomics"
  - "genes"
  - "biology"
  - "bioinformatics"
related_nodes:
  - "my-gene"
  - "my-variant"
  - "uni-prot-kb"
---

<!-- SECTION: header -->

# BioAPI

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

A powerful abstraction of genomics databases. BioAPI is part of the Multiomix project. Access gene information, expression data, pathways, ontologies, and drug-gene interactions.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **BioAPI** node provides access to multiple genomics and bioinformatics resources through the Multiomix BioAPI service.

### Key Features

- Search gene symbols
- Retrieve gene information
- Retrieve gene expression data
- Query gene ontology terms
- Retrieve pathway information
- Retrieve drug-gene interactions
- Query functional gene interactions
- Access OncoKB-related information
- Support multiple genomics operations from a single node

### Processing Flow

1. Select the BioAPI operation.
2. Configure the parameters required by the selected operation.
3. Build the corresponding BioAPI endpoint and request body.
4. Send the request to the Multiomix BioAPI service.
5. Return the JSON response to the workflow.

### Use Cases

- Search and normalize gene identifiers
- Retrieve genomics information
- Analyze gene expression
- Explore gene ontology relationships
- Retrieve metabolic pathway information
- Query pharmacogenomic data
- Analyze functional gene interactions
- Retrieve oncology-related gene information

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `operation` | Enum | Yes | BioAPI operation to execute. |
| `geneIds` | String | Conditional | JSON array of gene identifiers used by operations requiring multiple genes. |
| `geneId` | String | Conditional | Single gene identifier used by operations requiring one gene. |
| `tissue` | Enum | Conditional | Tissue used for gene expression queries. |
| `type` | Enum | No | Response type for gene expression. Values: `json`, `gzip`. Default: `json`. |
| `query` | String | Conditional | Search value used by `geneSymbolsFinder`. |
| `limit` | Number | No | Maximum number of results returned by `geneSymbolsFinder`. Default: `50`. |
| `filterType` | Enum | Conditional | Gene ontology filter type: `intersection`, `union`, or `enrichment`. |
| `ontologyType` | String | Conditional | JSON array containing ontology types. |
| `relationType` | String | Conditional | JSON array containing ontology relation types. |
| `pathwaySource` | Enum | Conditional | Pathway database used by `pathwayGenes`. |
| `pathwayId` | String | Conditional | Pathway identifier. |
| `termId` | String | Conditional | Gene Ontology term identifier. |
| `generalDepth` | Number | No | General relation depth used by `relatedTerms`. |
| `relations` | String | Conditional | JSON array of GO relation types. |
| `toRoot` | Number | No | Depth to root used by `relatedTerms`. Default: `0`. |
| `minCombinedScore` | Number | No | Minimum STRING combined score used by `stringRelations`. |

### Supported Operations

- `drugsPharmGkb`
- `drugsRegulatingGene`
- `expressionOfGenes`
- `geneSymbols`
- `geneSymbolsFinder`
- `genesOfItsGroup`
- `informationOfGenes`
- `genesToTerms`
- `relatedTerms`
- `pathwayGenes`
- `pathwaysInCommon`
- `stringRelations`
- `informationOfOncokb`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

Parameters can be configured directly in the node.

Some gene identifiers can also be resolved from incoming workflow data when they are not configured directly.

For example, `geneIds` expects a JSON array represented as a string:

```json
["BRCA1","BRCA2","EGFR"]
```

### Outputs

The node returns the JSON response produced by the selected BioAPI endpoint.

The exact output structure depends on the selected operation.

Examples include:

- Gene symbols
- Gene metadata
- Expression values
- Gene Ontology terms
- Pathway information
- Drug-gene relationships
- STRING interactions
- OncoKB information

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Search Gene Symbols

Configuration:

```json
{
  "operation": "geneSymbolsFinder",
  "query": "BRCA",
  "limit": 5
}
```

This searches for gene symbols matching `BRCA`.

### Retrieve Gene Information

```json
{
  "operation": "informationOfGenes",
  "geneIds": "[\"BRCA1\",\"EGFR\"]"
}
```

### Retrieve Pathway Genes

```json
{
  "operation": "pathwayGenes",
  "pathwaySource": "kegg",
  "pathwayId": "hsa00740"
}
```

### Workflow Example

```fusion-workflow
src: example.workflow.json
title: BioAPI Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Invalid JSON in Gene IDs

**Cause:** A parameter such as `geneIds`, `ontologyType`, `relationType`, or `relations` contains invalid JSON.

**Solution:** Provide a valid JSON array.

Example:

```json
["BRCA1","BRCA2"]
```

### BioAPI Resource Not Found

**Cause:** The requested BioAPI endpoint or resource returned HTTP `404`.

**Solution:** Verify the selected operation and identifiers.

### BioAPI Request Error

**Cause:** The BioAPI service returned an HTTP error.

**Solution:** Verify the request parameters and retry the workflow.

### Missing Required Operation Parameters

**Cause:** The selected operation requires parameters such as a gene ID, pathway ID, or ontology term that were not provided.

**Solution:** Configure the parameters required by the selected operation before execution.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related Nodes

- **My Gene** — Retrieve gene information from genomic data services.
- **My Variant** — Retrieve genetic variant information.
- **UniProtKB** — Access protein information from UniProtKB.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---|---|---|
| `1.0.0` | `2026-09-03` | Initial documentation. |

<!-- /SECTION: changelog -->