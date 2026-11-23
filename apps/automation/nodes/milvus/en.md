---
node_id: "milvus"
title: "Milvus Vector DB"
description: "Create and manage Milvus / Zilliz Cloud collections, insert, upsert, get, query, delete and search vector data with the Milvus RESTful API v2."
category: "peer-only"
subcategory: "Databases & Memory"
version: "1.0.0"
language: "en"
last_updated: "2026-11-23"
author: "Fusion Team"
tags:
  - milvus
  - zilliz
  - vector-database
  - embeddings
  - semantic-search
  - rag
related_nodes:
  - qdrant
  - chroma-db
  - weaviate
---

<!-- SECTION: overview -->
# Milvus Vector DB

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage vector data in **Milvus** or **Zilliz Cloud** (managed Milvus) through the **RESTful API v2**: create and manage **collections**, **insert / upsert** rows, **get**, **query** and **delete** them by ID or filter, and run **vector similarity search**. The node offers **10 operations**:

| Resource | Operations |
|----------|------------|
| Collections | `create_collection`, `describe_collection`, `list_collections`, `drop_collection` |
| Entities (rows) | `insert`, `upsert`, `get`, `query`, `delete` |
| Search | `search` |

> Milvus primary keys are **Int64**. The node keeps them exact: IDs are returned as **strings** (e.g. `"470661024491244064"`) and IDs you pass are not rounded.

### Use Cases

- **RAG pipelines:** Store document chunk embeddings and retrieve the closest chunks for an LLM prompt.
- **Semantic search:** Find similar products, tickets or articles from a query vector.
- **Deduplication:** Search for near-identical items before inserting new ones.
- **Data lifecycle:** Delete rows by filter (e.g. a removed source) or drop a test collection.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `milvusUrl` | `string` | Yes | Milvus endpoint (default `http://localhost:19530`) or Zilliz Cloud **Public Endpoint** (`https://in03-xxx.serverless.<region>.cloud.zilliz.com`). |
| `token` | `string` | If auth is enabled | Zilliz Cloud **Token** / API key, or `user:password` (plain text) for a Milvus server with authentication. Sent as `Authorization: Bearer <token>`. Leave empty for a local Milvus without auth. |
| `operation` | `enum` | Yes | Operation to perform (default `list_collections`). |
| `collectionName` | `string` | All except `list_collections` | Name of the collection. |
| `dimension` | `number` | For `create_collection` | Vector size (e.g. `768`, `1536`). **Zilliz Cloud requires at least 32.** |
| `metricType` | `enum` | No | `L2` (default), `IP` or `COSINE`. |
| `primaryField` | `string` | No | Primary key field name (default `id`). |
| `vectorField` | `string` | No | Vector field name (default `vector`). |
| `autoId` | `boolean` | No | Let Milvus generate primary keys (default `true`). Disable to provide your own IDs. |
| `data` | `string` | For `insert`, `upsert` | JSON array of rows, e.g. `[{"vector": [0.1, 0.2, ...], "text": "hello"}]`. |
| `vectors` | `string` | For `search` | Query vector(s): `[[0.1, 0.2, ...]]` or a single vector `[0.1, 0.2, ...]`. |
| `topK` | `number` | No | Results per query vector (default `10`). |
| `filter` | `string` | For `query`; optional for `search`, `delete` | Scalar filter expression, e.g. `id > 0`, `text == "Milvus"`, `color in ["red","blue"]`. |
| `limit` | `number` | No | Max rows for `query` (default `100`). |
| `id` | `string` | For `get`; or `delete` | One ID or several separated by commas. |
| `outputFields` | `string` | No | **Comma-separated** fields to return (`id,text`), or `*` for all (default). |

> Fields are shown only for the operations that use them.

### Operations

| Operation | Endpoint (`POST`) | Required | Notes |
|-----------|-------------------|----------|-------|
| `create_collection` | `/v2/vectordb/collections/create` | `collectionName`, `dimension` | Quick setup: Int64 primary key, one vector field, dynamic fields enabled. |
| `describe_collection` | `/v2/vectordb/collections/describe` | `collectionName` | Schema, fields, auto ID, consistency level. |
| `list_collections` | `/v2/vectordb/collections/list` | — | Returns collection names. |
| `drop_collection` | `/v2/vectordb/collections/drop` | `collectionName` | Permanent. |
| `insert` | `/v2/vectordb/entities/insert` | `collectionName`, `data` | Returns `insertCount` and `insertIds`. |
| `upsert` | `/v2/vectordb/entities/upsert` | `collectionName`, `data` | Rows must include the primary key. See note below. |
| `get` | `/v2/vectordb/entities/get` | `collectionName`, `id` | Rows by primary key. |
| `query` | `/v2/vectordb/entities/query` | `collectionName`, `filter` | Rows matching a filter. |
| `delete` | `/v2/vectordb/entities/delete` | `collectionName`, `filter` or `id` | With `id`, the node builds `<primaryKey> in [...]` from the collection schema. |
| `search` | `/v2/vectordb/entities/search` | `collectionName`, `vectors` | Nearest neighbours, with optional `filter`. |

> **Upsert and auto ID:** when the collection was created with `autoId: true`, Milvus does not update the row in place: it deletes the old row and inserts it with a **new generated ID**. To keep your own IDs, create the collection with `autoId` disabled and always send `id` in `data`.

### Connection

- **Zilliz Cloud:** in the cluster's **Overview → Connect** tab, copy the **Public Endpoint** into `milvusUrl` and the **Token** into `token`. Free serverless clusters are **paused** after a period of inactivity: click **Resume** in the console before using the node.
- **Self-hosted (Docker):** run Milvus standalone and use `http://<host>:19530`. If Fusion cannot reach your machine directly, expose port 19530 with a tunnel and use the public URL.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values (e.g. embeddings from an LLM node) in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` / `array` | The `data` part of the Milvus response. |
| `error` | `Error` | Missing fields, invalid JSON, HTTP errors (`Milvus API Error (<status>): ...`) or Milvus errors (`Milvus Logic Error (<code>): ...`). Requests time out after 120 s. |

### Output Examples

#### `list_collections`

```json
["testfusion"]
```

#### `insert`

```json
{ "insertCount": 2, "insertIds": ["470661024491244064", "470661024491244065"] }
```

#### `query` (filter `id > 0`, outputFields `id,text`)

```json
[
  { "id": "470661024491244064", "text": "Fusion" },
  { "id": "470661024491244065", "text": "Milvus" }
]
```

#### `search` (vectors `[[0.1, 0.2, 0.3, 0.4]]`, topK 2, metric L2)

```json
[
  { "distance": 0, "id": "470661024491244064", "text": "Fusion" },
  { "distance": 1.1999999, "id": "470661024491244065", "text": "Milvus" }
]
```

#### `delete`

```json
{ "deleteCount": 1 }
```

#### `create_collection` / `drop_collection`

```json
{}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Milvus Vector DB in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Milvus Vector DB Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Create a collection

```json
{
  "milvusUrl": "https://in03-xxx.serverless.aws-eu-central-1.cloud.zilliz.com",
  "token": "{{ secrets.ZILLIZ_TOKEN }}",
  "operation": "create_collection",
  "collectionName": "documents",
  "dimension": 1536,
  "metricType": "COSINE"
}
```

#### Insert rows

```json
{
  "operation": "insert",
  "collectionName": "documents",
  "data": "[{\"vector\": [0.12, 0.05, ...], \"text\": \"Fusion automation\", \"source\": \"docs\"}]"
}
```

#### Search with a filter

```json
{
  "operation": "search",
  "collectionName": "documents",
  "vectors": "[[0.11, 0.07, ...]]",
  "topK": 5,
  "filter": "source == \"docs\"",
  "outputFields": "text,source"
}
```

#### Delete by IDs

```json
{
  "operation": "delete",
  "collectionName": "documents",
  "id": "470661024491244064, 470661024491244065"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the Zilliz / Milvus `token` in Fusion's **Secrets** vault. Do not commit it into version control.

- `drop_collection` and `delete` are permanent: protect them from repeated triggers and check filters carefully (a broad filter deletes every match).
- Do not expose a self-hosted Milvus publicly without authentication.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Milvus Logic Error (90153): The action is unavailable under current cluster status STOPPED.`
- **Cause:** The Zilliz Cloud cluster is paused (free clusters pause when idle).
- **Solution:** Click **Resume** in the Zilliz console and wait until the cluster is **Running**.

#### `Milvus Logic Error (65535): parse output field name failed`
- **Cause:** `outputFields` was given as JSON (`["id","text"]`).
- **Solution:** Use a comma-separated list: `id,text` (or `*`).

#### `collectionName and dimension are required.`
- **Cause:** `create_collection` without `collectionName` or `dimension`.
- **Solution:** Fill in both (at least 32 on Zilliz Cloud).

#### `'data' must be a JSON array of objects.` / `Invalid JSON in 'data'`
- **Cause:** `data` is not a JSON array.
- **Solution:** Use `[{"vector": [...], "field": "value"}]`.

#### `'vectors' must be a JSON array of vectors`
- **Cause:** `vectors` is not a vector or a list of vectors.
- **Solution:** Use `[[0.1, 0.2, ...]]`.

#### Dimension mismatch error on `insert` / `search`
- **Cause:** The vector size differs from the collection `dimension`.
- **Solution:** Check the dimension with `describe_collection`.

#### `Either 'filter' or 'id' must be provided for delete.`
- **Cause:** `delete` without condition.
- **Solution:** Give a `filter` or one or more `id`s.

#### `upsert` returns a new ID
- **Cause:** The collection uses `autoId: true`.
- **Solution:** Expected Milvus behaviour; create the collection with `autoId` disabled to keep your IDs.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-23 | Initial documentation: 10 operations, Int64 ids, connection (Zilliz / self-hosted), output examples, troubleshooting |

<!-- /SECTION: changelog -->
