---
node_id: "qdrant"

title: "Qdrant Vector DB"

description: "Manage collections, upsert points, delete points, and perform semantic search using Qdrant Vector Database."

category: "AI / Vector Database"

version: "1.0.0"

language: "en"

last_updated: "2026-09-03"

author: "Fusion Team"

tags:

- qdrant

- vector-database

- semantic-search

- embeddings

- vector-search

- collections

- points

- similarity-search

- api

related_nodes:

- function

- if

- http-request

- chroma

- cassandra

---

**# Qdrant Vector DB**

> **\*\*Category:\*\*** ai-nodes | **\*\*Type:\*\*** Action Node

Manage collections and vector points using the **\*\*Qdrant\*\*** REST API, including collection creation and deletion, collection information retrieval, vector upserts, semantic search, and filtered point deletion.

The **\*\*Qdrant\*\*** node exposes six operations: `create_collection`, which creates a collection with a configurable vector size and distance metric; `delete_collection`, which deletes an existing collection; `get_collection_info`, which retrieves information about a collection; `upsert_points`, which inserts or updates vector points and their payloads; `search_points`, which performs vector similarity search with optional metadata filtering; and `delete_points`, which removes points matching a Qdrant filter.

**### Supported Features**

\- Create Qdrant collections with configurable vector dimensions and distance metrics

\- Delete existing collections

\- Retrieve collection information

\- Upsert one or more vector points with optional payload metadata

\- Perform semantic/vector similarity search

\- Apply Qdrant DSL filters during searches and point deletion

\- Configure the number of search results returned

\- Optionally include payload metadata in search results

\- Support Qdrant API-key authentication

\- Wait for write operations to be acknowledged by the Qdrant server

**### Use Cases**

\- Create vector collections for embedding-based applications

\- Store embeddings and associated metadata for RAG and semantic search workflows

\- Search documents or records using vector similarity

\- Combine vector search with metadata filters

\- Remove points matching specific metadata conditions

\- Inspect and manage Qdrant collections from within a workflow

\- Use Qdrant Cloud or authenticated Qdrant deployments through an API key

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `qdrantUrl` | `string` | ❌ No | `"http://localhost:6333"` | Qdrant REST URL, such as a local Qdrant instance or a Qdrant Cloud cluster URL. |
| `apiKey` | `string` | ❌ No | — | API key used when Qdrant authentication is enabled. Sent using the `api-key` HTTP header. |
| `operation` | `enum` | ❌ No | `"get_collection_info"` | Operation: `create_collection`, `delete_collection`, `get_collection_info`, `upsert_points`, `search_points`, or `delete_points`. |
| `collectionName` | `string` | ✅ Yes | — | Name of the Qdrant collection. Must contain at least one character. |

**### Create Collection Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `vectorSize` | `number` | ❌ No | `768` | Dimension of vectors stored in the collection. Must be at least `1`. |
| `distance` | `enum` | ❌ No | `"Cosine"` | Distance metric used for vector similarity. Supported values: `Cosine`, `Euclid`, or `Dot`. |
| `wait` | `boolean` | ❌ No | `true` | Wait for Qdrant to acknowledge the collection operation before returning. |

**### Upsert Points Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `points` | `string` | ✅ Yes | — | JSON array of Qdrant points to insert or update. Each point can contain an `id`, `vector`, and optional `payload`. |
| `wait` | `boolean` | ❌ No | `true` | Wait for Qdrant to acknowledge the upsert operation before returning. |

**### Search Points Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `vector` | `string` | ✅ Yes | — | Query vector represented as a JSON array. |
| `limit` | `number` | ❌ No | `5` | Number of similarity-search results to return. |
| `filter` | `string` | ❌ No | — | Optional Qdrant DSL filter represented as JSON. |
| `withPayload` | `boolean` | ❌ No | `true` | Whether payload metadata should be included in search results. |

**### Delete Points Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `filter` | `string` | ✅ Yes | — | JSON Qdrant filter used to select points for deletion. |
| `wait` | `boolean` | ❌ No | `true` | Wait for Qdrant to acknowledge the deletion before returning. |

**---**

**## Operations**

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| `create_collection` | `/collections/{name}` | `PUT` | Create a collection with the configured vector size and distance metric. |
| `delete_collection` | `/collections/{name}` | `DELETE` | Delete the specified collection. |
| `get_collection_info` | `/collections/{name}` | `GET` | Retrieve information about the specified collection. |
| `upsert_points` | `/collections/{name}/points` | `PUT` | Insert or update vector points in the collection. |
| `search_points` | `/collections/{name}/points/search` | `POST` | Search for similar vectors using a query vector and optional filter. |
| `delete_points` | `/collections/{name}/points/delete` | `POST` | Delete points matching a Qdrant filter. |

**---**

**## Request Body Construction**

**### Create Collection**

```json
{
  "vectors": {
    "size": 768,
    "distance": "Cosine"
  }
}
```

**### Upsert Points**

```json
{
  "points": [
    {
      "id": 1,
      "vector": [0.1, 0.2, 0.3],
      "payload": {
        "city": "London"
      }
    }
  ]
}
```

**### Search Points**

```json
{
  "vector": [0.2, 0.1, 0.3],
  "limit": 5,
  "with_payload": true
}
```

If a filter is provided, it is added to the search payload.

**### Delete Points**

```json
{
  "filter": {
    "must": [
      {
        "key": "city",
        "match": {
          "value": "London"
        }
      }
    ]
  }
}
```

**---**

**## Inputs & Outputs**

**### Inputs**

The node does not require workflow input. All configuration is provided through the node configuration.

**### Outputs**

The node returns the Qdrant `result` property directly when present. Otherwise, it returns the complete parsed API response.

The node does not wrap responses in a custom `success`, `operation`, or `data` structure.

**### Output Example**

```json
[
  {
    "id": 1,
    "version": 1,
    "score": 0.95,
    "payload": {
      "city": "London"
    }
  }
]
```

**---**

**## Configuration Examples**

**### Create Collection**

```json
{
  "qdrantUrl": "http://localhost:6333",
  "operation": "create_collection",
  "collectionName": "documents",
  "vectorSize": 768,
  "distance": "Cosine",
  "wait": true
}
```

**### Upsert Points**

```json
{
  "qdrantUrl": "http://localhost:6333",
  "operation": "upsert_points",
  "collectionName": "documents",
  "points": "[{\"id\":1,\"vector\":[0.1,0.2,0.3],\"payload\":{\"city\":\"London\"}}]",
  "wait": true
}
```

**### Search Points**

```json
{
  "qdrantUrl": "http://localhost:6333",
  "operation": "search_points",
  "collectionName": "documents",
  "vector": "[0.2,0.1,0.3]",
  "limit": 5,
  "withPayload": true
}
```

**### Delete Points**

```json
{
  "qdrantUrl": "http://localhost:6333",
  "operation": "delete_points",
  "collectionName": "documents",
  "filter": "{\"must\":[{\"key\":\"city\",\"match\":{\"value\":\"London\"}}]}",
  "wait": true
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Embedding Generation → Qdrant (`upsert_points`)

\- User Query → Embedding Generation → Qdrant (`search_points`) → LLM

\- Qdrant (`search_points`) → Function

\- Qdrant (`delete_points`) → Database/Function

**---**

**## Error Handling**

**### Missing Points**

```text
points (JSON) is required for upsert.
```

**### Invalid Points Type**

```text
Points must be a JSON array.
```

**### Missing Vector**

```text
vector (JSON) is required for search.
```

**### Missing Delete Filter**

```text
filter (JSON) is required to delete points.
```

**### Invalid JSON**

```text
Invalid JSON in '<fieldName>': <JSON parse error>
```

**### API Error**

```text
Qdrant API Error (<status>): <error message>
```

**### Unknown Operation**

```text
Unknown operation: <operation>
```

**---**

**## Troubleshooting**

**### Invalid JSON errors**

Ensure `points`, `vector`, and `filter` contain valid JSON strings.

**### Vector dimension errors**

The vector dimension must match the collection's configured `vectorSize`.

**### Authentication errors**

Provide a valid `apiKey` when the Qdrant deployment requires authentication.

**### Collection not found**

Verify that `collectionName` exists before running point or search operations.

**---**

**## Security**

When `apiKey` is provided, the node sends it using:

```text
api-key: <apiKey>
```

Use HTTPS for remote production deployments and avoid hardcoding API keys in workflow configurations.

**---**

**## Notes**

The node does not generate embeddings automatically. Vectors must be provided directly as JSON.

The node does not automatically create collections, retry failed requests, paginate results, or cache responses.

The `wait` parameter is used by the implementation for `create_collection`, `delete_collection`, `upsert_points`, and `delete_points`. However, the schema's `dependsOn` configuration only exposes `wait` for `create_collection`, `upsert_points`, and `delete_points`.

The default Qdrant URL is:

```text
http://localhost:6333
```

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-03 | Initial release |
