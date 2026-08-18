---
node_id: "g2-p"
title: "G2P"
description: "Genomics 2 Proteins Portal API - Gene-Transcript-Protein Isoform-Structure mapping, protein feature annotations, and isoform sequence alignments from the Broad Institute."
category: "Healthcare & Life Sciences"
subcategory: "Biology & Life Sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - genomics
  - proteomics
  - genes
  - proteins
  - isoforms
  - broad-institute
  - bioinformatics
related_nodes:
  - ncbi-gene
  - ncbi-protein
  - uni-prot-kb
---

<!-- SECTION: header -->
# G2P

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Query the Genomics 2 Proteins Portal API from the Broad Institute to retrieve Gene-Transcript-Protein Isoform-Structure mapping, protein feature annotations, and isoform sequence alignments for genomic and proteomic research workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **G2P** node provides access to the Genomics 2 Proteins Portal API, enabling workflows to map genes to transcripts, proteins, and isoforms, as well as retrieve protein feature annotations and structural alignments. It is useful for bioinformatics research, genomic data enrichment, and protein annotation pipelines.

### Key Features

- **Gene-Transcript-Protein Mapping:** Retrieve relationships between genes, transcripts, and protein isoforms
- **Protein Feature Annotations:** Get detailed protein feature information from the Broad Institute dataset
- **Isoform Sequence Alignments:** Access isoform-level sequence alignment data
- **UniProt Integration:** Support queries using UniProt identifiers and gene names
- **Structured Output:** Return machine-readable JSON for analysis and downstream processing
- **Research-Grade Data:** Access curated data from the Broad Institute

### Typical Use Cases

- Retrieve protein features for a gene of interest
- Map genes to their transcript and protein isoforms
- Enrich genomic variant annotations with protein data
- Build gene-to-protein reference tables
- Support computational biology and bioinformatics workflows
- Analyze protein structure and isoform variations
- Integrate genomic data into research pipelines

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | — | Operation to perform. Accepts `getProteinFeatures`, `getGeneMapping`, `getIsoformAlignment`, or similar G2P operations. |
| `geneName` | `string` | ❌ No | — | Gene name (e.g., `BRCA1`, `TP53`) used for gene-based queries. |
| `uniprotId` | `string` | ❌ No | — | UniProt identifier (e.g., `P38398`) used for protein-based queries. |
| `ensemblId` | `string` | ❌ No | — | Ensembl gene identifier for transcript and isoform mapping. |
| `transcriptId` | `string` | ❌ No | — | Ensembl transcript identifier for isoform-specific queries. |
| `format` | `enum` | ❌ No | `json` | Response format. Typically `json` or `xml`. |
| `filters` | `object` | ❌ No | — | Optional filters for narrowing results (e.g., filter by protein family or structural class). |
| `limit` | `number` | ❌ No | `100` | Maximum number of results to return. |

### Supported Operations

| Operation | Description | Required Parameters |
|-----------|-------------|-------------------|
| `getProteinFeatures` | Retrieve protein feature annotations | `geneName` or `uniprotId` |
| `getGeneMapping` | Get gene-to-transcript-protein mapping | `geneName` or `ensemblId` |
| `getIsoformAlignment` | Retrieve isoform sequence alignments | `transcriptId` or `geneName` |

### Example

```text
operation: "getProteinFeatures"
geneName: "BRCA1"
uniprotId: "P38398"
format: "json"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` or `string` | Workflow input used to override configured parameters such as gene name, transcript ID, or operation. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | G2P API response containing mapping, protein features, or alignment data. |
| `error` | `object` | Error details when the request fails or validation fails. |

### Success Response Example: getProteinFeatures

```json
{
  "geneName": "BRCA1",
  "uniprotId": "P38398",
  "proteinFeatures": [
    {
      "featureType": "domain",
      "description": "RING-type zinc finger",
      "start": 1,
      "end": 100
    },
    {
      "featureType": "domain",
      "description": "serine-rich region",
      "start": 200,
      "end": 300
    }
  ],
  "isoforms": [
    {
      "transcriptId": "ENST00000357654",
      "proteinId": "ENSP00000350283",
      "sequence": "MDLSALRPVF..."
    }
  ]
}
```

### Success Response Example: getGeneMapping

```json
{
  "geneName": "BRCA1",
  "ensemblId": "ENSG00000012048",
  "transcripts": [
    {
      "transcriptId": "ENST00000357654",
      "proteinId": "ENSP00000350283",
      "biotype": "protein_coding"
    }
  ],
  "count": 5
}
```

### Error Response Example

```json
{
  "success": false,
  "error": "Gene not found or API request failed",
  "geneName": "BRCA1"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Get Protein Features for BRCA1

```text
operation: "getProteinFeatures"
geneName: "BRCA1"
format: "json"
```

**Result:**

```json
{
  "geneName": "BRCA1",
  "proteinFeatures": [
    {
      "featureType": "domain",
      "description": "RING-type zinc finger",
      "start": 1,
      "end": 100
    }
  ]
}
```

### Example: Map Gene to Transcripts and Proteins

Use the node to retrieve all transcript and protein isoforms for a gene, then route results into a database or downstream analysis pipeline.

```text
operation: "getGeneMapping"
geneName: "TP53"
format: "json"
```

### Example: Genomic Enrichment Workflow

Integrate G2P queries into variant annotation pipelines to enrich genetic variants with protein-level information and isoform details.

<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->
