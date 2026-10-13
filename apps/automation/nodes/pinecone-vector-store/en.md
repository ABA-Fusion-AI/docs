---
node_id: "pinecone-vector-store"
title: "Pinecone Vector Store"
description: "Manage Pinecone collections and indexes, and query, upsert, fetch, update, or delete vectors."
category: "Vector Database"
version: "1.0.0"
language: "en"
last_updated: "2026-10-13"
author: "Fusion Team"
tags:
  - pinecone
  - vector-database
  - embeddings
  - vector-search
  - collections
  - indexes
related_nodes:
  - function
  - if
  - http-request
---

<!-- SECTION: header -->
# Pinecone Vector Store

> **Category:** Vector Database | **Type:** Action Node

Use the **Pinecone Vector Store** node to manage collections and indexes or perform vector operations in a Pinecone index. Select one operation per node execution. The node returns the result of the corresponding Pinecone client call.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The node supports collection listing, creation, deletion, and inspection; index listing, creation, deletion, inspection, and configuration; and vector statistics, search, deletion, fetching, updating, and upserting. Configure the operation and its associated fields before running the workflow.

The node expects numeric embedding vectors. It does not generate embeddings from text. For a query or upsert, provide vectors produced by an embedding step with dimensions compatible with the target index.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Common fields

| Field | Type | Required | Default | Description |
| ----- | ---- | -------- | ------- | ----------- |
| `apiKey` | `string` | Yes | — | Pinecone API key. |
| `environment` | `enum` | No | `us-east1-gcp` | Legacy pod environment. Accepted values: `us-west1-gcp`, `us-west4-gcp`, `us-central1-gcp`, `us-east1-gcp`, `us-east4-gcp`, `eu-west1-gcp`, and `us-east1-aws`. Serverless index location uses `indexData.cloud` and `indexData.region`. |
| `operation` | `enum` | No | `listIndexes` | Operation to execute. See the table below. |
| `projectId` | `string` | No | — | Legacy field; vector operations no longer require it. |

### Operations

| Operation | Purpose | Operation-specific configuration |
| --------- | ------- | -------------------------------- |
| `listCollections` | List collections. | None. |
| `createCollection` | Create a collection. | `collectionData.name` is required; `collectionData.source` is optional. |
| `deleteCollection` | Delete a collection. | `collectionName` is required. |
| `describeCollection` | Get collection details. | `collectionName` is required. |
| `listIndexes` | List indexes. | None. |
| `createIndex` | Create a dense index. | `indexData.name` and `indexData.dimension` are required. |
| `deleteIndex` | Delete an index. | `indexName` is required. |
| `describeIndex` | Get index details. | `indexName` is required. |
| `configureIndex` | Change index configuration. | `indexName` and `indexConfigData` are required. |
| `describeIndexStats` | Get index statistics. | `indexName` is required; `statsData` is optional. |
| `query` | Find matching vectors. | `indexName` and `queryData` are required. Supply `queryData.vector` or `queryData.id`. |
| `deleteVectors` | Delete vectors by IDs, filter, or namespace-wide deletion. | `indexName` and `deleteData` are required. Supply `deleteData.ids`, `deleteData.filter`, or `deleteData.deleteAll: true`. |
| `fetchVectors` | Fetch vectors by ID. | `indexName` and `fetchData.ids` are required; `ids` must be nonempty. |
| `updateVector` | Update one vector. | `indexName` and `updateData.id` are required. |
| `upsertVectors` | Insert or replace vectors. | `indexName` and a nonempty `upsertData.vectors` array are required. |

### Collection and index fields

| Field | Type | Used by | Details |
| ----- | ---- | ------- | ------- |
| `collectionName` | `string` | `deleteCollection`, `describeCollection` | Name of the target collection. |
| `collectionData` | `object` | `createCollection` | `name` (nonempty string), optional `source` (string). |
| `indexName` | `string` | Index operations other than `listIndexes` and `createIndex`; all vector operations | Name of the target index. |
| `indexData` | `object` | `createIndex` | Required `name` (nonempty string) and `dimension` (number, at least 1). Optional fields are listed below. |
| `indexConfigData` | `object` | `configureIndex` | Optional properties within the required object are listed below. |

`indexData` accepts `metric` (`cosine`, `euclidean`, or `dotproduct`), `cloud` (`aws`, `gcp`, or `azure`), `region` (string), `pods` (number), `replicas` (number), `pod_type` (string), `deletion_protection` (`enabled` or `disabled`), `tags` (string-valued map), and `metadata_config` (any value). Its description specifies serverless AWS `us-east-1` as the default when no cloud or region is supplied. Legacy pod fields remain supported.

`indexConfigData` accepts `replicas` (number), `pod_type` (string), `spec` (any value), `deletion_protection` (`enabled` or `disabled`), and `tags` (string-valued map). The `spec`, deletion protection, and tags fields are passed to the client configuration operation; legacy pod settings remain supported.

### Vector operation fields

| Field | Type | Used by | Contents |
| ----- | ---- | ------- | -------- |
| `statsData` | `object` | `describeIndexStats` | Optional `filter` (any value). |
| `queryData` | `object` | `query` | `vector` (number array) or `id` (string); optional `topK` (number, at least 1), `includeValues` (boolean), `includeMetadata` (boolean), `namespace` (string), and `filter` (any value). |
| `deleteData` | `object` | `deleteVectors` | `ids` (string array), `deleteAll` (boolean), or `filter` (any value); optional `namespace` (string). |
| `fetchData` | `object` | `fetchVectors` | Required nonempty `ids` (string array); optional `namespace` (string). |
| `updateData` | `object` | `updateVector` | Required nonempty `id` (string); optional `values` (number array), `setMetadata` (map), and `namespace` (string). |
| `upsertData` | `object` | `upsertVectors` | Required nonempty `vectors` array; optional `namespace` (string). Each vector requires a nonempty `id` and nonempty numeric `values` array, and may include a `metadata` map. |

The schema exposes these fields only for their relevant operations. Additional checks in the node enforce the required names and payloads before it calls the client.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs and outputs

The node reads its operation and parameters from its configuration. Its tick handler does not use incoming workflow data directly. It returns the selected client method's response without transforming it; the shape therefore depends on the operation and Pinecone response.

If an operation fails, the node throws an error prefixed with `Pinecone operation failed:` followed by the underlying message. A missing index name, collection name, or required operation payload produces a specific validation message under that prefix.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Create a serverless index

```json
{
  "apiKey": "YOUR_PINECONE_API_KEY",
  "operation": "createIndex",
  "indexData": {
    "name": "knowledge-base",
    "dimension": 1536,
    "metric": "cosine",
    "cloud": "aws",
    "region": "us-east-1"
  }
}
```

Set `dimension` to match the embedding model used by your workflow.

### Upsert vectors

```json
{
  "apiKey": "YOUR_PINECONE_API_KEY",
  "operation": "upsertVectors",
  "indexName": "knowledge-base",
  "upsertData": {
    "namespace": "articles",
    "vectors": [
      {
        "id": "article-1",
        "values": [0.12, -0.08, 0.44],
        "metadata": { "title": "Getting started" }
      }
    ]
  }
}
```

The sample vector is shortened for readability. Supply the number of values required by the index dimension.

### Query vectors

```json
{
  "apiKey": "YOUR_PINECONE_API_KEY",
  "operation": "query",
  "indexName": "knowledge-base",
  "queryData": {
    "namespace": "articles",
    "vector": [0.12, -0.08, 0.44],
    "topK": 5,
    "includeMetadata": true
  }
}
```

You can use `queryData.id` instead of `queryData.vector` to query by an existing vector ID. Use a full-length vector when querying by values.

### Delete selected vectors

```json
{
  "apiKey": "YOUR_PINECONE_API_KEY",
  "operation": "deleteVectors",
  "indexName": "knowledge-base",
  "deleteData": {
    "namespace": "articles",
    "ids": ["article-1"]
  }
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

| Error or symptom | What to check |
| ---------------- | ------------- |
| `Collection name is required` | Supply `collectionName` for delete or describe, or `collectionData.name` for create. |
| `Index name is required` | Supply `indexName` for the selected index or vector operation. |
| `Index name and dimension are required` | Supply `indexData.name` and a positive `indexData.dimension`. |
| `Either vector or id must be provided in query data` | Set `queryData.vector` or `queryData.id`. |
| `Either ids, deleteAll, or filter must be provided in delete data` | Specify the intended deletion target in `deleteData`; review `deleteAll` carefully before running. |
| `Fetch data with ids array is required` | Provide a nonempty `fetchData.ids` array. |
| `Update data with id is required` | Provide `updateData.id`. |
| `Upsert data with vectors array is required` | Provide at least one entry in `upsertData.vectors`. |
| Pinecone rejects a vector | Check that its numeric values match the index dimension and that the index name and namespace are correct. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-10-13 | Documented the node's collection, index, and vector operations. |

<!-- /SECTION: changelog -->
