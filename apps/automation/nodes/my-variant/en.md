---
node_id: "my-variant"
title: "MyVariant.info"
description: "Search and retrieve genetic variant annotations using the MyVariant.info API."
category: "science-research"
subcategory: "biology-life-sciences"
version: "1.0.0"
language: "en"
last_updated: "2026-09-01"
author: "Fusion Team"
tags:
  - myvariant
  - genetics
  - genomics
  - variants
  - bioinformatics
  - biology
  - api
related_nodes:
  - my-gene
  - ensembl
  - uni-prot-kb
  - bio-api
---

<!-- SECTION: header -->
# MyVariant.info

> **Category:** Biology & Life Sciences | **Type:** Action Node

Search and retrieve genetic variant annotations from multiple databases using the MyVariant.info API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **MyVariant.info** node provides access to MyVariant.info genetic variant query web services.

It supports metadata retrieval, genetic variant searches, direct variant retrieval, and batch variant retrieval through the MyVariant.info API.

The node provides six operations:

- `getMetadata` — Retrieve MyVariant.info metadata.
- `getMetadataFields` — Retrieve available annotation fields.
- `queryGet` — Search variants using a GET request.
- `queryPost` — Search one or more variants using a POST request.
- `getVariant` — Retrieve a specific variant by ID.
- `getVariantBatch` — Retrieve multiple variants in a single request.

### Key Features

- **Metadata Retrieval:** Retrieve MyVariant.info build and source metadata.
- **Field Discovery:** Retrieve available genetic annotation fields.
- **Variant Search:** Search variant annotations using query strings.
- **GET and POST Queries:** Supports both GET and POST query operations.
- **Single Variant Retrieval:** Retrieve annotations for a specific variant ID.
- **Batch Retrieval:** Retrieve up to 1000 variant IDs in a single batch operation.
- **Field Selection:** Limit returned annotation fields.
- **Pagination:** Configure result size and offset.
- **Faceting:** Request facets for supported fields.
- **Sorting:** Sort query results by one or more fields.
- **Input Resolution:** Resolve query strings and variant IDs from incoming workflow data.
- **API Error Handling:** Returns descriptive errors for failed MyVariant.info requests.

### Use Cases

- Search genetic variants
- Retrieve variant annotations
- Query variants using dbSNP identifiers
- Retrieve annotations for HGVS-based variant IDs
- Retrieve multiple variants in batches
- Explore available MyVariant.info annotation fields
- Build bioinformatics workflows
- Integrate genetic variant data into automated pipelines

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Operations

| Operation | HTTP Method | Endpoint | Description |
|-----------|-------------|----------|-------------|
| `getMetadata` | `GET` | `/metadata` | Retrieve MyVariant.info metadata. |
| `getMetadataFields` | `GET` | `/metadata/fields` | Retrieve available annotation fields. |
| `queryGet` | `GET` | `/query` | Search variants using query parameters. |
| `queryPost` | `POST` | `/query` | Search variants using a JSON request body. |
| `getVariant` | `GET` | `/variant/{id}` | Retrieve a specific variant. |
| `getVariantBatch` | `POST` | `/variant` | Retrieve multiple variants. |

The default operation is:

```text
queryGet
```

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ❌ No | `queryGet` | Operation to perform. |
| `q` | `string` | Conditional | — | Query string used by `queryGet` and `queryPost`. Required for `queryGet`; optional for `queryPost`. |
| `scopes` | `string` | ❌ No | — | Comma-separated fields to search for `queryPost`. |
| `id` | `string` | Conditional | — | HGVS-based variant ID. Required for `getVariant`. |
| `ids` | `string` | Conditional | — | Comma-separated variant IDs. Required for `getVariantBatch`. |
| `fields` | `string` | ❌ No | `all` | Comma-separated fields to return. |
| `size` | `number` | ❌ No | `10` | Maximum number of results. Range: 1-1000. |
| `from` | `number` | ❌ No | `0` | Number of matching results to skip. |
| `fetch_all` | `boolean` | ❌ No | `false` | Enable retrieval of all unsorted query hits. |
| `scroll_id` | `string` | ❌ No | — | Scroll identifier used to retrieve additional results. |
| `sort` | `string` | ❌ No | — | Comma-separated fields used for sorting. |
| `facets` | `string` | ❌ No | — | Field or fields used for faceted queries. |
| `facet_size` | `number` | ❌ No | `10` | Number of facet buckets. Range: 1-1000. |
| `callback` | `string` | ❌ No | — | Callback parameter for JSONP calls. |
| `dotfield` | `boolean` | ❌ No | `false` | Return flattened data using dotfield notation. |
| `email` | `string` | ❌ No | — | Optional email for API usage tracking. |

### `getMetadata`

Retrieves metadata about the MyVariant.info data build and available sources.

No additional parameters are required.

The node sends:

```text
GET /metadata
```

### `getMetadataFields`

Retrieves information about the annotation fields available through MyVariant.info.

No additional parameters are required.

The node sends:

```text
GET /metadata/fields
```

### `queryGet`

Searches MyVariant.info using an HTTP GET request.

`q` is required.

Example:

```text
Operation: queryGet
Q: rs58991260
```

The node sends a request equivalent to:

```text
GET /query?q=rs58991260
```

### `queryPost`

Searches MyVariant.info using an HTTP POST request.

Example:

```text
Operation: queryPost
Q: rs58991260
Scopes: dbsnp.rsid
```

The node converts `q` and `scopes` into arrays.

Equivalent request body:

```json
{
  "q": ["rs58991260"],
  "scopes": ["dbsnp.rsid"]
}
```

Comma-separated values are split and trimmed before being added to the request body.

### `getVariant`

Retrieves a specific genetic variant using its HGVS-based identifier.

`id` is required.

Example:

```text
Operation: getVariant
Id: chr1:g.218631822G>A
```

The variant ID is URL encoded before being added to the endpoint path.

### `getVariantBatch`

Retrieves multiple variants using a POST request.

`ids` is required and accepts comma-separated variant identifiers.

Example:

```text
chr1:g.218631822G>A,chr7:g.55241707G>T
```

The node splits and trims the IDs before sending them.

A maximum of 1000 IDs is included in the request body.

### Common Query Parameters

Default values are not added to the query string when they retain their defaults.

For example:

```text
fields = all
size = 10
from = 0
fetch_all = false
facet_size = 10
dotfield = false
```

are omitted unless their behavior requires them to be sent.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow data that can provide query strings or variant IDs when they are not configured directly. |

For object input, the node resolves query values from:

```text
q
query
search
term
```

Variant IDs can be resolved from:

```text
id
variantId
variant_id
```

Batch variant IDs can be resolved from:

```text
ids
variantIds
variant_ids
```

Configured values take precedence over values received from incoming workflow data.

A string input can also be used when the corresponding configured value is missing.

For `queryGet` and `queryPost`, a string input can become `q`.

For `getVariant`, a string input can become `id`.

### Outputs

The node returns the JSON response received directly from the MyVariant.info API.

The exact response structure depends on the selected operation.

#### Query Response

A `queryGet` request can return:

```json
{
  "took": 2,
  "total": 1,
  "max_score": 21.30118,
  "hits": [
    {
      "_id": "chr1:g.218631822G>A",
      "_score": 21.30118
    }
  ]
}
```

#### Variant Response

A `getVariant` request returns the requested variant and its available annotations.

Example:

```json
{
  "_id": "chr1:g.218631822G>A",
  "_version": 1
}
```

Additional annotation fields are returned depending on the available MyVariant.info data.

#### Batch Response

`getVariantBatch` returns an array containing the results for the requested variant IDs.

### Error Output

Errors are thrown using the following format:

```text
MyVariant.info <operation> failed: <error message>
```

A `404` response produces an underlying error in the following format:

```text
MyVariant.info Resource not found (404): <url>
```

Other unsuccessful HTTP responses use:

```text
MyVariant.info API Error: <status> <status text>
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Retrieve API Metadata

**Configuration**

```text
Operation: getMetadata
```

The node retrieves MyVariant.info metadata, including build and source information.

---

### Example: Retrieve Metadata Fields

**Configuration**

```text
Operation: getMetadataFields
```

The node retrieves the available annotation field definitions.

---

### Example: Search by dbSNP Identifier

**Configuration**

```text
Operation: queryGet
Q: rs58991260
Fields: all
Size: 10
From: 0
```

Example response:

```json
{
  "took": 2,
  "total": 1,
  "max_score": 21.30118,
  "hits": [
    {
      "_id": "chr1:g.218631822G>A",
      "_score": 21.30118
    }
  ]
}
```

---

### Example: POST Query

**Configuration**

```text
Operation: queryPost
Q: rs58991260
Scopes: dbsnp.rsid
```

Equivalent request body:

```json
{
  "q": ["rs58991260"],
  "scopes": ["dbsnp.rsid"]
}
```

---

### Example: Retrieve a Variant

**Configuration**

```text
Operation: getVariant
Id: chr1:g.218631822G>A
Fields: all
```

Example response:

```json
{
  "_id": "chr1:g.218631822G>A",
  "_version": 1
}
```

---

### Example: Retrieve Multiple Variants

**Configuration**

```text
Operation: getVariantBatch
Ids: chr1:g.218631822G>A,chr7:g.55241707G>T
Fields: all
Size: 10
```

The node converts the comma-separated IDs into an array and sends them using a POST request.

---

### Example: Resolve Query from Input

If `q` is not configured, `queryGet` can resolve it from incoming workflow data.

Input:

```json
{
  "query": "rs58991260"
}
```

The resolved query becomes:

```text
rs58991260
```

---

### Example: Resolve Variant ID from Input

For `getVariant`, incoming data can provide:

```json
{
  "variantId": "chr1:g.218631822G>A"
}
```

when `id` is not configured directly.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Query Genetic Variant
```

### Common Patterns

- **Variant Search:** Manual Trigger → MyVariant.info → Log
- **Variant Annotation:** Input Data → MyVariant.info → Processing Node
- **Variant Lookup:** Previous Node → MyVariant.info → Log
- **Batch Retrieval:** Variant IDs → MyVariant.info → Processing Node
- **Bioinformatics Pipeline:** MyVariant.info → Data Processing → Storage

The included example workflow uses:

```text
Manual Trigger → MyVariant.info → Log
```

with:

```text
Operation: queryGet
Q: rs58991260
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "q (query) is required for queryGet operation"

**Cause**

The `queryGet` operation could not resolve a query from either the configured `q` parameter or incoming workflow data.

**Solution**

Configure `q`:

```text
rs58991260
```

or provide one of the supported input fields:

```text
q
query
search
term
```

---

#### "id is required for getVariant operation"

**Cause**

The `getVariant` operation could not resolve a variant ID.

**Solution**

Configure an HGVS-based variant ID:

```text
chr1:g.218631822G>A
```

or provide `id`, `variantId`, or `variant_id` from the previous node.

---

#### "ids is required for getVariantBatch operation"

**Cause**

The `getVariantBatch` operation could not resolve any variant IDs.

**Solution**

Provide comma-separated variant IDs:

```text
chr1:g.218631822G>A,chr7:g.55241707G>T
```

Incoming data can also provide `ids`, `variantIds`, or `variant_ids`.

---

#### Resource Not Found

**Cause**

MyVariant.info returned HTTP status `404`.

**Error**

```text
MyVariant.info Resource not found (404): <url>
```

**Solution**

Verify that the requested variant or endpoint is valid.

---

#### API Request Error

**Cause**

MyVariant.info returned an unsuccessful HTTP response.

**Error format**

```text
MyVariant.info API Error: <status> <status text>
```

**Solution**

Check the query parameters, variant identifiers, and MyVariant.info API availability.

---

#### Unexpected Query or Variant

**Cause**

A value configured directly on the node takes precedence over incoming workflow data.

**Solution**

Check the node configuration before relying on `q`, `query`, `search`, `term`, `id`, `variantId`, `variant_id`, `ids`, `variantIds`, or `variant_ids` from the previous node.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- MyGene.info
- Ensembl
- UniProtKB
- Bio API

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-01 | Initial release of the MyVariant.info documentation. |

<!-- /SECTION: changelog -->