---
node_id: "chromadb-manager"
title: "ChromaDB Manager"
description: "Manage collections, add documents, and query ChromaDB."
category: "Vector Database"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:

- chromadb
- vector-database
- embeddings
- rag
- collections
- semantic-search
- vector-search
- documents

related_nodes:
- function
- if
- http-request

---

# ChromaDB Manager

> **Category:** vector-database-nodes | **Type:** Action Node

Manage **ChromaDB** collections, add documents, and run semantic queries — including a generic cross-collection search — against a ChromaDB instance's REST API (v2).

The **ChromaDB Manager** node exposes an `operation` selector covering collection management, document ingestion, single-collection querying, a multi-collection "search all" mode, and a heartbeat health check.

### Supported Features

- Create, get, delete, and list collections
- Get-or-create semantics on collection creation
- Add documents with auto-generated fallback IDs
- Query a single collection with metadata filtering (`where`)
- Search across multiple collections at once, optionally filtered by name prefix
- Relevance sorting (by distance) and top-N slicing for cross-collection search
- Optional Bearer token authentication
- Multi-tenant / multi-database support (`tenant`, `database`)
- Heartbeat health check
- JSON input parsing with dedicated error messages per field

### Use Cases

- Build and maintain a RAG (Retrieval-Augmented Generation) knowledge base
- Ingest documents into a ChromaDB collection from a workflow
- Query a collection for semantically similar documents
- Search across several related collections at once (e.g. all collections prefixed `avis_`)
- Health-check a ChromaDB instance before running a pipeline
- Manage collection lifecycle (create/get/delete) as part of a setup workflow
- Feed query results into an LLM node for context-augmented generation

---

## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `baseUrl` | `string` | ❌ No | `"http://localhost:8000"` | ChromaDB base URL. |
| `apiKey` | `string` | ❌ No | — | ChromaDB API key, sent as a Bearer token if provided. |
| `tenant` | `string` | ❌ No | `"default_tenant"` | ChromaDB tenant name. |
| `database` | `string` | ❌ No | `"default_database"` | ChromaDB database name. |
| `operation` | `enum` | ❌ No | `"query"` | Operation to perform. See [Operations](#operations) below. |

### Operation-Specific Parameters

| Parameter | Type | Required For | Default | Description |
| --------- | ---- | ------------- | ------- | ----------- |
| `collectionName` | `string` | `create_collection`, `get_collection`, `delete_collection`, `add_documents`, `query` | — | Name of the target collection. |
| `metadata` | `string` (JSON object) | `create_collection` (optional) | — | JSON metadata to attach to the created collection. |
| `documents` | `string` (JSON array) | `add_documents` | — | JSON array of text strings to add. |
| `ids` | `string` (JSON array) | `add_documents` (optional) | — | JSON array of document IDs. Auto-generated if omitted or length mismatch. |
| `metadatas` | `string` (JSON array) | `add_documents` (optional) | — | JSON array of metadata objects, one per document. |
| `queryTexts` | `string` (JSON array) | `query`, `search_all` | — | JSON array of query text strings. |
| `nResults` | `number` | `query`, `search_all` (optional) | `5` | Number of results to return. |
| `where` | `string` (JSON object) | `query`, `search_all` (optional) | — | Metadata filter, e.g. `{"author": "john"}`. |
| `collectionPrefix` | `string` | `search_all` (optional) | — | Only search collections whose name starts with this prefix. |

---

## Operations

| Operation | HTTP Call | Description |
| --------- | --------- | ----------- |
| `list_collections` | `GET /api/v2/tenants/{tenant}/databases/{database}/collections` | List all collections in the tenant/database. |
| `heartbeat` | `GET /api/v2/heartbeat` | Health-check the ChromaDB instance. |
| `create_collection` | `POST .../collections` | Create a collection (`get_or_create: true`). Requires `collectionName`. |
| `get_collection` | `GET .../collections/{name}` | Get a single collection by name. Requires `collectionName`. |
| `delete_collection` | `DELETE .../collections/{name}` | Delete a collection by name. Requires `collectionName`. |
| `add_documents` | `POST .../collections/{name}/add` | Add documents to a collection. Requires `collectionName` and `documents`. |
| `query` | `POST .../collections/{name}/query` | Query a single collection. Requires `collectionName` and `queryTexts`. |
| `search_all` | Multiple `GET`/`POST` calls | Query across all (or prefix-filtered) collections and merge/rank results. Requires `queryTexts`. |

---

## Add Documents Behavior

- `documents` must be a JSON array of strings — the node throws an error if missing or not an array.
- `ids`: if the provided array's length matches `documents`, it is used as-is. Otherwise, IDs are **auto-generated** as `"<timestamp>-<index>"` for each document.
- `metadatas`: only sent if its array length matches `documents`; otherwise omitted from the request.

---

## Search All Behavior

The `search_all` operation is a **generic, multi-collection search**:

1. Only the **first** entry of `queryTexts` is used (single query per call).
2. All collections are listed via `list_collections`.
3. If `collectionPrefix` is set, only collections whose name starts with that prefix are searched.
4. Each matching collection is queried **concurrently** (`Promise.all`) with the same query text, `nResults`, and `where` filter.
5. Errors from individual collections are **silently ignored** — a failing collection does not fail the whole operation.
6. Chroma's per-collection nested arrays (`documents`, `distances`, `metadatas`, `ids`) are **flattened** into a single list of `{ collection, document, distance, metadata, id }` objects.
7. Results are **sorted by ascending distance** (lower distance = more relevant for L2/cosine metrics).
8. The final list is **sliced to the top `nResults`** entries across all collections combined.

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

Output shape depends on the operation:

| Operation | Output |
| --------- | ------ |
| `list_collections` | Array of collection objects as returned by ChromaDB. |
| `heartbeat` | ChromaDB heartbeat response object. |
| `create_collection` / `get_collection` | Collection object (id, name, metadata, etc). |
| `delete_collection` | ChromaDB deletion confirmation response. |
| `add_documents` | ChromaDB add response. |
| `query` | Raw ChromaDB query response (`documents`, `distances`, `metadatas`, `ids`, nested per query text). |
| `search_all` | Flattened, distance-sorted array of `{ collection, document, distance, metadata, id }`, limited to `nResults`. |

---

## Output Example

### `query`

```json
{
  "documents": [["ChromaDB is an open-source vector database."]],
  "distances": [[0.184]],
  "metadatas": [[{ "source": "docs" }]],
  "ids": [["doc-1"]]
}
```

### `search_all`

```json
[
  {
    "collection": "avis_2026",
    "document": "Le service client était excellent.",
    "distance": 0.112,
    "metadata": { "rating": 5 },
    "id": "1755000000-0"
  },
  {
    "collection": "avis_2025",
    "document": "Livraison rapide et produit conforme.",
    "distance": 0.201,
    "metadata": { "rating": 4 },
    "id": "1754000000-2"
  }
]
```

---

## Configuration Examples

### List Collections

```json
{
  "operation": "list_collections"
}
```

### Create Collection

```json
{
  "operation": "create_collection",
  "collectionName": "avis_2026",
  "metadata": "{\"description\": \"Customer reviews 2026\"}"
}
```

### Add Documents

```json
{
  "operation": "add_documents",
  "collectionName": "avis_2026",
  "documents": "[\"Great service!\", \"Fast delivery.\"]",
  "metadatas": "[{\"rating\": 5}, {\"rating\": 4}]"
}
```

### Query a Collection

```json
{
  "operation": "query",
  "collectionName": "avis_2026",
  "queryTexts": "[\"customer complaint about delivery\"]",
  "nResults": 5,
  "where": "{\"rating\": {\"$lt\": 3}}"
}
```

### Search Across Collections by Prefix

```json
{
  "operation": "search_all",
  "queryTexts": "[\"customer complaint about delivery\"]",
  "collectionPrefix": "avis_",
  "nResults": 10
}
```

### With Authentication

```json
{
  "baseUrl": "https://chroma.example.com",
  "apiKey": "your-api-key",
  "operation": "query",
  "collectionName": "avis_2026",
  "queryTexts": "[\"delivery delay\"]"
}
```

---

## Workflow Integration

### Sample Workflow: Add Documents → Query

```json
{
  "nodes": [
    {
      "id": "chroma-add",
      "type": "chromadb-manager",
      "config": {
        "operation": "add_documents",
        "collectionName": "avis_2026",
        "documents": "[\"Great service!\"]"
      }
    },
    {
      "id": "chroma-query",
      "type": "chromadb-manager",
      "config": {
        "operation": "query",
        "collectionName": "avis_2026",
        "queryTexts": "[\"service quality\"]"
      }
    }
  ]
}
```

### Sample Workflow: RAG Pipeline

```json
{
  "nodes": [
    {
      "id": "chroma-search-all",
      "type": "chromadb-manager",
      "config": {
        "operation": "search_all",
        "queryTexts": "[\"customer complaint\"]",
        "collectionPrefix": "avis_",
        "nResults": 8
      }
    },
    {
      "id": "build-context",
      "type": "function"
    },
    {
      "id": "llm-answer",
      "type": "llm"
    }
  ]
}
```

### Sample Workflow: Heartbeat → If → Query

```json
{
  "nodes": [
    {
      "id": "chroma-heartbeat",
      "type": "chromadb-manager",
      "config": {
        "operation": "heartbeat"
      }
    },
    {
      "id": "check-alive",
      "type": "if"
    },
    {
      "id": "chroma-query",
      "type": "chromadb-manager",
      "config": {
        "operation": "query",
        "collectionName": "avis_2026",
        "queryTexts": "[\"delivery delay\"]"
      }
    }
  ]
}
```

### Common Patterns

- Schedule → ChromaDB (`create_collection`) → ChromaDB (`add_documents`) — ingestion pipeline
- ChromaDB (`search_all`) → Function (flatten/format) → LLM — RAG context retrieval
- ChromaDB (`heartbeat`) → If → downstream ChromaDB operations — health-gated pipeline
- ChromaDB (`query`) → If (distance threshold) → Notification — relevance alerting

---

## Error Handling

### Missing Required Parameter

```text
collectionName is required for create_collection
collectionName is required
documents must be a JSON array of strings.
queryTexts is required for search_all
```

### Invalid JSON Input

```text
Invalid JSON for '<fieldName>': <parse error message>
```

Raised for any malformed JSON string passed to `metadata`, `documents`, `ids`, `metadatas`, `queryTexts`, or `where`.

### ChromaDB API Error

```text
ChromaDB Error (<status>): <message>
```

Raised when ChromaDB returns a non-OK HTTP status. The message is taken from the response body's `message` field when present, otherwise the HTTP status text.

### Request Failure

```text
ChromaDB Request Failed: <error message>
```

Wraps any network-level failure or the API error above.

### Unimplemented Operation

```text
Operation <operation> not implemented.
```

---

## Troubleshooting

### "collectionName is required for create_collection" / "collectionName is required"

**Cause**

`collectionName` was left empty for an operation that requires it.

**Solution**

Set `collectionName` to a valid, existing (or intended) collection name.

---

### "documents must be a JSON array of strings."

**Cause**

The `documents` parameter for `add_documents` is missing, empty, or not a valid JSON array.

**Solution**

Provide a JSON array string, e.g. `["text one", "text two"]`.

---

### "Invalid JSON for '<fieldName>': ..."

**Cause**

One of the JSON-string parameters (`metadata`, `documents`, `ids`, `metadatas`, `queryTexts`, `where`) contains malformed JSON.

**Solution**

Validate the JSON syntax for the named field before submitting.

---

### "ChromaDB Error (<status>): ..."

**Cause**

ChromaDB rejected the request — common causes include a non-existent collection (`get_collection`, `delete_collection`, `add_documents`, `query`), an invalid `where` filter, or a permissions/auth issue.

**Solution**

Verify the collection exists (`list_collections`), check the `where` filter syntax, and confirm `apiKey` is set correctly if the instance requires authentication.

---

### "ChromaDB Request Failed: ..."

**Cause**

The instance at `baseUrl` is unreachable — wrong URL, ChromaDB not running, or a network/firewall issue.

**Solution**

Confirm `baseUrl` is correct and the ChromaDB server is running and reachable; try the `heartbeat` operation first to verify connectivity.

---

### `search_all` Returns Fewer Results Than Expected

**Cause**

Individual collection query failures are **silently skipped** — a collection with an incompatible embedding function, empty data, or an error will simply not contribute results, with no error surfaced.

**Solution**

Run `query` directly against the suspected collection to see the underlying error, or narrow `collectionPrefix` to isolate the problem collection.

---

### Auto-Generated IDs Look Unexpected

**Cause**

If `ids` is omitted or its length doesn't match `documents`, IDs are auto-generated as `"<Date.now()>-<index>"`, which can produce **duplicate prefixes** for documents added within the same call in rapid succession from the same tick.

**Solution**

Provide an explicit `ids` array matching the length of `documents` if stable, predictable IDs are required.

---

## Security

The node performs outbound HTTP requests to the configured `baseUrl`.

If `apiKey` is set, it is sent as an `Authorization: Bearer <apiKey>` header on every request.

No credentials are required for a ChromaDB instance running without authentication enabled.

---

## Notes

The node returns the raw ChromaDB API response for most operations, except `search_all`, which flattens and re-ranks results client-side.

The node does not:

- Generate or manage embeddings itself (delegated to the ChromaDB server/embedding function)
- Validate `where` filter syntax before sending it
- Retry failed requests
- Deduplicate documents across collections in `search_all`
- Persist or cache query results

It is intended to provide direct, low-level access to ChromaDB collection and query operations, plus a convenience cross-collection search, for downstream RAG and vector-search workflows.

---

## Related

- [Function](./function.md) – Transform, flatten, or format query results
- [If](./if.md) – Route workflows based on query results or heartbeat status
- [HTTP Request](./http-request.md) – Make additional custom ChromaDB API calls

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-12 | Initial release |