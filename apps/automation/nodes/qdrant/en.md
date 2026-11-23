---
node_id: "qdrant"
title: "Qdrant Vector DB"
description: "Create and manage Qdrant collections, upsert and delete points, and run vector similarity search."
category: "peer-only"
subcategory: "Databases & Memory"
version: "1.0.0"
language: "en"
last_updated: "2026-11-22"
author: "Fusion Team"
tags:
  - qdrant
  - vector-database
  - embeddings
  - semantic-search
  - rag
related_nodes:
  - chroma-db
  - weaviate
  - milvus
---

<!-- SECTION: overview -->
# Qdrant Vector DB

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage a **Qdrant** vector database through its REST API: create and delete **collections**, store **points** (a vector + a payload), run **similarity search** and delete points with a **filter**. The node offers **6 operations**:

| Resource | Operations |
|----------|------------|
| Collections | `create_collection`, `get_collection_info`, `delete_collection` |
| Points | `upsert_points`, `search_points`, `delete_points` |

### Use Cases

- **RAG pipelines:** Store document embeddings and retrieve the closest chunks for an LLM prompt.
- **Semantic search:** Find similar products, tickets or articles from a query vector.
- **Recommendations:** Search the nearest items to a user or item vector.
- **Data cleanup:** Delete all points matching a payload filter (e.g. a removed source).

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `qdrantUrl` | `string` | Yes | Qdrant REST URL (default `http://localhost:6333`, or your Qdrant Cloud cluster URL). |
| `apiKey` | `string` | No | API key (Qdrant Cloud or a server with auth enabled), sent in the `api-key` header. |
| `operation` | `enum` | Yes | Operation to perform (default `get_collection_info`). |
| `collectionName` | `string` | Yes | Name of the collection. |
| `vectorSize` | `number` | For `create_collection` | Vector dimension (default `768`; e.g. `1536` for OpenAI `text-embedding-3-small`). |
| `distance` | `enum` | For `create_collection` | Similarity metric: `Cosine` (default), `Euclid` or `Dot`. |
| `points` | `string` | For `upsert_points` | JSON array of points: `[{"id": 1, "vector": [...], "payload": {...}}]`. |
| `vector` | `string` | For `search_points` | Query vector as a JSON array: `[0.12, 0.05, ...]`. |
| `limit` | `number` | No | Number of search results (default `5`). |
| `filter` | `string` | For `delete_points` (optional for search) | Qdrant filter as JSON, e.g. `{"must":[{"key":"city","match":{"value":"London"}}]}`. |
| `withPayload` | `boolean` | No | Include the payload in search results (default `true`). |
| `wait` | `boolean` | No | Wait until Qdrant has applied the change before returning (default `true`). |

> Fields are shown only for the operations that use them.

### Operations

| Operation | Endpoint | Required | Optional |
|-----------|----------|----------|----------|
| `create_collection` | `PUT /collections/{name}` | `collectionName` | `vectorSize`, `distance`, `wait` |
| `get_collection_info` | `GET /collections/{name}` | `collectionName` | — |
| `delete_collection` | `DELETE /collections/{name}` | `collectionName` | — |
| `upsert_points` | `PUT /collections/{name}/points` | `collectionName`, `points` | `wait` |
| `search_points` | `POST /collections/{name}/points/search` | `collectionName`, `vector` | `limit`, `filter`, `withPayload` |
| `delete_points` | `POST /collections/{name}/points/delete` | `collectionName`, `filter` | `wait` |

> Every vector (in `points` and `vector`) must have exactly `vectorSize` numbers. Point IDs must be unsigned integers or UUIDs.

### Connection

- **Self-hosted:** run Qdrant with Docker and use its URL:
  ```bash
  docker run -d -p 6333:6333 qdrant/qdrant
  ```
  If Fusion cannot reach your machine directly, expose the port with a tunnel and use the public URL.
- **Qdrant Cloud:** copy the cluster URL and create an API key in the cluster's **API Keys** tab, then paste them into `qdrantUrl` and `apiKey`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values (e.g. embeddings) in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | The `result` field of the Qdrant response. |
| `error` | `Error` | Emitted on missing fields, invalid JSON or API errors (`Qdrant API Error (<status>): <message>`). |

### Output Examples

#### `create_collection` / `delete_collection`

```json
true
```

#### `upsert_points` / `delete_points`

```json
{ "operation_id": 1, "status": "completed" }
```

#### `search_points`

```json
[
  { "id": 1, "version": 1, "score": 0.0128, "payload": { "text": "Fusion automation" } },
  { "id": 2, "version": 1, "score": 0.00002, "payload": { "text": "Qdrant vectors" } }
]
```

#### `get_collection_info` (excerpt)

```json
{
  "status": "green",
  "optimizer_status": "ok",
  "indexed_vectors_count": 0,
  "points_count": 2,
  "segments_count": 4,
  "config": {
    "params": {
      "vectors": { "size": 768, "distance": "Cosine" },
      "shard_number": 1,
      "replication_factor": 1
    }
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Qdrant Vector DB in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Qdrant Vector DB Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### Create a collection

```json
{
  "qdrantUrl": "https://your-cluster.qdrant.io",
  "apiKey": "{{ secrets.QDRANT_API_KEY }}",
  "operation": "create_collection",
  "collectionName": "documents",
  "vectorSize": 1536,
  "distance": "Cosine"
}
```

#### Upsert points

```json
{
  "operation": "upsert_points",
  "collectionName": "documents",
  "points": "[{\"id\": 1, \"vector\": [0.12, 0.05, ...], \"payload\": {\"text\": \"Fusion automation\"}}]"
}
```

#### Search with a filter

```json
{
  "operation": "search_points",
  "collectionName": "documents",
  "vector": "[0.11, 0.07, ...]",
  "limit": 3,
  "filter": "{\"must\": [{\"key\": \"lang\", \"match\": {\"value\": \"en\"}}]}"
}
```

#### Delete points by filter

```json
{
  "operation": "delete_points",
  "collectionName": "documents",
  "filter": "{\"must\": [{\"key\": \"source\", \"match\": {\"value\": \"old-site\"}}]}"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the Qdrant `apiKey` in Fusion's **Secrets** vault. Do not commit it into version control.

- `delete_collection` and `delete_points` are permanent: protect them from repeated triggers.
- Check the `filter` of `delete_points` carefully. A broad filter deletes every matching point.
- Do not expose a self-hosted Qdrant publicly without an API key.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `points (JSON) is required for upsert.` / `Points must be a JSON array.`
- **Cause:** `points` is empty or is not an array.
- **Solution:** Use `[{"id": 1, "vector": [...], "payload": {...}}]`.

#### `vector (JSON) is required for search.` / `filter (JSON) is required to delete points.`
- **Cause:** The required JSON field is empty.
- **Solution:** Fill in `vector` (search) or `filter` (delete).

#### `Invalid JSON in 'points'` (or `vector`, `filter`)
- **Cause:** The field is not valid JSON (e.g. single quotes).
- **Solution:** Use double quotes: `{"key": "value"}`.

#### `Qdrant API Error (400)` mentioning the vector dimension
- **Cause:** A vector does not have `vectorSize` numbers.
- **Solution:** Use vectors of the same size as the collection (check with `get_collection_info`).

#### `Qdrant API Error (404)`
- **Cause:** The collection does not exist.
- **Solution:** Create it with `create_collection`, or check `collectionName`.

#### `Qdrant API Error (401/403)` or connection error
- **Cause:** Missing or wrong API key, or an unreachable URL (e.g. `localhost` seen from Fusion).
- **Solution:** Check `apiKey` and use a URL that Fusion can reach.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-22 | Full documentation: 6 operations, parameters, connection, output examples, troubleshooting |
| 1.0.0 | 2026-09-03 | Initial release |

<!-- /SECTION: changelog -->
