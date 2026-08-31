---
node_id: "my-gene"
title: "MyGene.info"
description: "MyGene.info API - Gene query web services. Search and retrieve gene annotations from multiple databases."
category: "healthcare-life-sciences"
subcategory: "biology-life-sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:
  - gene
  - genetics
  - biology
  - bioinformatics
  - mygene
  - genomics
  - life-sciences
related_nodes:
  - ensembl
  - my-variant
  - uni-prot-kb
---

<!-- SECTION: header -->
# MyGene.info

> **Category:** Healthcare & Life Sciences | **Type:** Action Node

Search and retrieve gene annotations using the MyGene.info API.

<!-- /SECTION: header -->

<!-- SECTION: overview -->
## Overview

The MyGene.info node provides access to MyGene.info gene query web services.

It supports metadata retrieval, gene searches, individual gene lookups, and batch gene retrieval.

The node connects to:

```text
https://mygene.info/v3
```

No API key is required.

### Supported Operations

- `getMetadata`
- `getMetadataFields`
- `queryGet`
- `queryPost`
- `getGene`
- `getGeneBatch`

<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

### Operation

Select the operation to perform.

| Operation | Description |
|---|---|
| `getMetadata` | Retrieve MyGene.info service metadata |
| `getMetadataFields` | Retrieve available gene annotation fields |
| `queryGet` | Search genes using an HTTP GET request |
| `queryPost` | Search one or multiple genes using an HTTP POST request |
| `getGene` | Retrieve a single gene by Entrez or Ensembl ID |
| `getGeneBatch` | Retrieve multiple genes in a single request |

### Query

Used by `queryGet` and `queryPost`.

Example:

```text
CDK2
```

For `queryPost`, one or more query values can be provided as a comma-separated list:

```text
CDK2,BRCA1
```

### Scopes

Used by `queryPost` to specify one or more fields to search.

Example:

```text
symbol
```

Multiple scopes can be comma-separated.

### Gene ID

Used by `getGene`.

Supported identifiers include Entrez Gene IDs and Ensembl Gene IDs.

Example:

```text
1017
```

### Gene IDs

Used by `getGeneBatch`.

Provide comma-separated gene identifiers.

Example:

```text
1017,672
```

Up to 1000 IDs are processed by the node.

### Fields

Comma-separated list of fields to return.

Default:

```text
all
```


### Species

Restricts results to one or more species.

Examples:

```text
human
mouse
9606
```

Multiple species can be provided as a comma-separated list when supported by the selected operation.

### Size

Maximum number of matching results.

Default:

```text
10
```

Maximum:

```text
1000
```

### From

Number of matching hits to skip for pagination.

Default:

```text
0
```

### Fetch All

When enabled, requests all matching query results using the MyGene.info scrolling mechanism.

### Scroll ID

Scroll identifier returned by a previous query using `fetch_all`.

### Sort

Comma-separated list of fields used to sort query results.

Prefix a field with `-` for descending order.

### Facets

Single field or comma-separated list of fields for faceted queries.

### Facet Size

Number of facet buckets to return.

Default:

```text
10
```

Maximum:

```text
1000
```

### Species Facet Filter

Species filter used with faceted queries.

### Entrez Only

When enabled, returns only hits containing valid Entrez Gene IDs.

### Ensembl Only

When enabled, returns only hits containing valid Ensembl Gene IDs.

### Callback

Optional JSONP callback parameter.

Available for `queryGet` and `getGene`.

### Dotfield

When enabled, returned data is flattened using dot notation.

### Email

Optional email address for API usage tracking.

<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs and Outputs

### Input

The node accepts data through its `input` connection.

Parameters configured directly on the node take precedence. When relevant values are not configured, the node can resolve them from incoming data.

For query operations, supported input properties include:

```json
{
  "q": "CDK2"
}
```

The following aliases are also recognized:

```text
query
search
term
```

For single-gene retrieval, supported properties include:

```text
id
geneId
gene_id
entrezgene
ensembl
```

For batch retrieval:

```text
ids
geneIds
gene_ids
```

A plain string input can also be used as the query for `queryGet` or `queryPost`, or as the gene ID for `getGene`.

### Success

Returns the JSON response received from MyGene.info.

The response structure depends on the selected operation.

### Error

Errors are returned through the `error` output and include the selected operation in the error message.

Missing required parameters, unavailable resources, or unsuccessful API responses cause the selected operation to fail.

<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Examples

### Get Service Metadata

Configuration:

```text
Operation: getMetadata
```

Returns MyGene.info service metadata including build and source information.

### Get Metadata Fields

Configuration:

```text
Operation: getMetadataFields
```

Returns the available annotation fields and their metadata.

### Search Gene with GET

Configuration:

```text
Operation: queryGet
Q: CDK2
Fields: all
Species: human
Size: 10
From: 0
Fetch All: false
Dotfield: false
```

Example response structure:

```json
{
  "total": 16,
  "hits": [
    {
      "_id": "1017",
      "symbol": "CDK2",
      "taxid": 9606
    }
  ]
}
```

### Search Multiple Genes with POST

Configuration:

```text
Operation: queryPost
Q: CDK2,BRCA1
Scopes: symbol
Fields: all
Species: human
Size: 10
```

The node builds a POST request body similar to:

```json
{
  "q": [
    "CDK2",
    "BRCA1"
  ],
  "scopes": [
    "symbol"
  ]
}
```

The tested request returned results for both `CDK2` and `BRCA1`.

### Get a Single Gene

Configuration:

```text
Operation: getGene
Id: 1017
Fields: all
Species: human
Dotfield: false
```

This retrieves detailed annotations for gene ID `1017`.

### Get Multiple Genes

Configuration:

```text
Operation: getGeneBatch
Ids: 1017,672
Species: human
```

The node builds a POST request body containing:

```json
{
  "ids": [
    "1017",
    "672"
  ]
}
```

The tested request returned two gene records.

### Workflow Example

```fusion-workflow
src: example.workflow.json
title: MyGene.info Example
```

The example workflow uses:

```text
Manual Trigger → MyGene.info → Log
```

with the representative configuration:

```json
{
  "operation": "getGeneBatch",
  "species": "human",
  "ids": "1017,672"
}
```

<!-- /SECTION: examples -->

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Query Is Required

For `queryGet`, provide a query in the node configuration or through incoming data.


### Gene ID Is Required

For `getGene`, provide an Entrez or Ensembl gene identifier.

Example:

```text
1017
```

### Gene IDs Are Required

For `getGeneBatch`, provide comma-separated identifiers.

Example:

```text
1017,672
```

### Resource Not Found

If MyGene.info returns HTTP `404`, the node reports the resource as not found and includes the requested URL in the error.

### API Errors

Non-successful MyGene.info responses are returned with the HTTP status, status text, and response details when available.

### Batch Size

`getGeneBatch` processes at most the first 1000 comma-separated IDs.

<!-- /SECTION: troubleshooting -->

<!-- SECTION: related -->
## Related Nodes

- **Ensembl** — Access genomic and gene information from Ensembl.
- **MyVariant.info** — Search and retrieve human genetic variant annotations.
- **UniProtKB** — Access protein sequence and functional annotation data.

<!-- /SECTION: related -->

<!-- SECTION: changelog -->
## Changelog

### 1.0.0 — 2026-08-31

- Added documentation for MyGene.info.
- Documented all six supported operations.
- Added tested metadata, query, single-gene, and batch examples.
- Added example workflow.

<!-- /SECTION: changelog -->