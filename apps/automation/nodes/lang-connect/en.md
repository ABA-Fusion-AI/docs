---
node_id: "lang-connect"
title: "LangConnect Client"
description: "Manage vector collections and index documents using LangConnect API."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-09-24"
author: "Fusion Team"
tags: [integration, peer-only, langconnect, documents, vector-search]
related_nodes: []
---

<!-- SECTION: overview -->
# LangConnect Client

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage vector collections and documents through a LangConnect API. The node can create, retrieve, update, and delete collections; list, upload, search, and delete documents; or delete multiple document or file IDs with a per-ID result summary.

Use it to index reference files, search a collection, or maintain documents used by downstream workflows.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

### Connection

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `baseUrl` | String | No | `http://localhost:8000` | Base URL of the LangConnect API. One trailing slash is removed before appending the endpoint. |
| `apiKey` | String | No | None | Token used when authentication is enabled. Sent as `Authorization: Bearer <apiKey>`. |
| `operation` | Enum | No | `list_collections` | Operation to execute; see the Operations table. |

Supply the token without the `Bearer` prefix. If `apiKey` is empty or omitted, the node sends no authorization header. The API must be reachable from the workflow runtime; `localhost` refers to that runtime's host.

### Operation Parameters

The configuration exposes fields according to the selected operation. The supplied schema does not mark any parameter with `expression: true`.

| Parameter | Type | Default | Used by / requirements |
| --- | --- | --- | --- |
| `collectionId` | String | None | Required for every operation except `list_collections` and `create_collection`. Identifies the collection. |
| `collectionName` | String | None | Required for `create_collection`; optional for `update_collection`. |
| `metadata` | JSON string | None | Optional collection metadata for create/update, or document metadata for upload. Intended to contain an object. |
| `fileUrl` | String | None | Required for `upload_document`. URL from which the runtime downloads a file. |
| `chunkSize` | Number | `1000` | Maximum characters per chunk for upload, sent as `chunk_size`. |
| `chunkOverlap` | Number | `200` | Character overlap between chunks for upload, sent as `chunk_overlap`. |
| `query` | String | None | Required for `search_documents`. Search text. |
| `limit` | Number | `10` | Result limit for `list_documents` and `search_documents`. |
| `offset` | Number | `0` | Pagination offset for `list_documents` only. |
| `searchType` | Enum | `similarity` | Search mode: `similarity` or `mmr`. Sent as `search_type`. |
| `filter` | JSON string | None | Optional metadata filter for `search_documents`. Intended to contain an object. |
| `documentId` | String | None | Required for `delete_document`. Document/chunk ID, or file ID when `deleteBy` is `file_id`. |
| `deleteBy` | Enum | `document_id` | Used by single and bulk deletion. `document_id` targets a chunk; `file_id` targets all chunks belonging to a file. |
| `targetIds` | JSON string | None | Required for `bulk_delete_documents`. JSON array of non-empty string IDs. |

### JSON Fields and Validation

Enter valid JSON with double quotes, for example:

- `metadata`: `{"domain":"medical"}`
- `filter`: `{"author":"admin"}`
- `targetIds`: `["id1","id2"]`

Empty `metadata` or `filter` values are treated as omitted. Their JSON syntax is checked, but their object structure is not validated by the node. Bulk deletion checks that `targetIds` is an array containing only non-empty strings; an empty array is accepted and produces zero counts.

The schema does not enforce integer or range constraints for chunk sizes, limits, or offsets, and does not validate URL or UUID formats. Use values supported by your LangConnect server.
<!-- /SECTION: configuration -->

<!-- SECTION: operations -->
## Operations

Paths are relative to `baseUrl`. Standard requests use JSON headers; file uploads use multipart form data.

| Operation | Method and endpoint | Behavior |
| --- | --- | --- |
| `list_collections` | `GET /collections` | Retrieve collections. No pagination parameters are exposed for this operation. |
| `get_collection` | `GET /collections/{collectionId}` | Retrieve one collection. |
| `create_collection` | `POST /collections` | Send `name` and optional parsed `metadata`. |
| `update_collection` | `PATCH /collections/{collectionId}` | Send optional `name` and parsed `metadata`. Omitted fields are excluded; the node allows an empty update body. |
| `delete_collection` | `DELETE /collections/{collectionId}` | Request collection deletion. |
| `list_documents` | `GET /collections/{collectionId}/documents?limit={limit}&offset={offset}` | Retrieve one page of documents. |
| `upload_document` | `POST /collections/{collectionId}/documents` | Download one file from `fileUrl`, then upload it as multipart form data. |
| `search_documents` | `POST /collections/{collectionId}/documents/search` | Send `query`, `limit`, `search_type`, and optional parsed `filter`. |
| `delete_document` | `DELETE /collections/{collectionId}/documents/{documentId}?delete_by={deleteBy}` | Delete by document/chunk ID or file ID. |
| `bulk_delete_documents` | One document `DELETE` request per target ID | Apply `deleteBy` to each ID and return a combined result. |

### File Upload

The node first downloads `fileUrl` without adding the LangConnect API token or custom authentication headers. Use a URL accessible to the runtime without those headers. The incoming workflow payload is not used as file content.

The downloaded file is loaded into a Blob and attached to the multipart field `files`. Its filename comes from the final URL path segment, excluding the query string, with `upload.txt` as a fallback. When parsed metadata is truthy, it is wrapped in a one-element array and sent as `metadatas_json`. Chunk settings are sent as text fields. The runtime sets the multipart content type and boundary automatically.

### Bulk Deletion

Bulk deletion sends individual requests in batches of up to five concurrent requests. Each batch finishes before the next begins. Failures are collected while the remaining IDs continue to be processed; IDs are not deduplicated.

For this operation, collection IDs, target IDs, and the deletion mode are URL-encoded. Other operations interpolate IDs directly into their paths.

The node does not automatically fetch additional pages, retry failed requests, or configure a custom timeout. Its `stop()` method does not cancel requests already in progress.
<!-- /SECTION: operations -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An incoming event triggers execution. Its data is ignored by the handler; request values come from the node configuration.
- **Standard result:** The parsed LangConnect JSON response is returned unchanged. Its structure depends on the selected operation and the server response.
- **No-content result:** A successful HTTP `204` from a standard request returns `{ "success": true }`. Upload uses a separate handler that always attempts to parse successful responses as JSON.
- **Errors:** Missing required fields, invalid JSON, network failures, and unsuccessful API requests normally throw an error. Bulk deletion collects individual request failures in its result instead.

### Bulk Delete Result

```json
{
  "successfulIds": ["document-1"],
  "failedIds": [
    {
      "id": "document-2",
      "status": 404,
      "error": "LangConnect API Error (404): Not Found"
    }
  ],
  "totalRequested": 2,
  "totalDeleted": 1
}
```

This is an illustrative result. `status` contains the HTTP code extracted from a standard API error, or `null` when unavailable, such as for a network or JSON parsing failure. `totalDeleted` counts successful ID requests, not the number of chunks removed by file-ID deletion.

Inspect `failedIds` even when the action returns normally: some or all deletions may have failed. Validation errors occur before processing begins and still throw an error.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: errors -->
## Errors & Troubleshooting

| Error or condition | What to check |
| --- | --- |
| `collectionName is required.` | Supply a non-empty name when creating a collection. |
| `collectionId is required.` | Select a collection for the requested operation. |
| `fileUrl is required.` | Supply a file URL for upload. |
| `query is required.` | Supply non-empty search text. |
| `collectionId and documentId are required.` | Supply both IDs for single deletion. |
| `collectionId and targetIds are required.` | Supply a collection ID and a JSON array string for bulk deletion. |
| `Invalid JSON format in '<field>': ...` | Correct the JSON syntax in `metadata`, `filter`, or `targetIds`; use double quotes. |
| `targetIds must be a JSON array.` | Use an array rather than an object or scalar value. |
| `targetIds must contain non-empty string IDs only.` | Remove empty strings and non-string values. |
| `Failed to download file from URL: <url>` | The source returned an unsuccessful HTTP response. Check file availability and access. |
| `LangConnect API Error (<status>): <message>` | Check the API address, token, IDs, and request values. The message uses a truthy JSON `detail` field when available, otherwise the HTTP status text. |
| `Upload Failed (<status>): <body>` | Inspect the server's upload response for the rejection reason. |
| JSON parsing failure after a successful response | Standard requests except HTTP `204`, and all successful uploads, expect JSON response bodies. |
| Network failure | Check that both the API and any download URL are reachable from the workflow runtime. |

An unsupported operation reaching the handler throws `Operation <operation> not implemented.`; the schema restricts selection to the ten documented operations.
<!-- /SECTION: errors -->

<!-- SECTION: examples -->
## Examples

Replace the example URL and IDs with values from your LangConnect instance. Supply `apiKey` separately if authentication is enabled.

### Create a Collection

```json
{
  "baseUrl": "https://langconnect.example.com",
  "operation": "create_collection",
  "collectionName": "Knowledge Base",
  "metadata": "{\"domain\":\"support\"}"
}
```

### Upload a Document

```json
{
  "baseUrl": "https://langconnect.example.com",
  "operation": "upload_document",
  "collectionId": "<collection-uuid>",
  "fileUrl": "https://files.example.com/guide.pdf",
  "metadata": "{\"author\":\"support\"}",
  "chunkSize": 1000,
  "chunkOverlap": 200
}
```

### Search Documents

```json
{
  "baseUrl": "https://langconnect.example.com",
  "operation": "search_documents",
  "collectionId": "<collection-uuid>",
  "query": "How do I reset my account password?",
  "limit": 5,
  "searchType": "similarity",
  "filter": "{\"author\":\"support\"}"
}
```

### Delete Multiple Documents

```json
{
  "baseUrl": "https://langconnect.example.com",
  "operation": "bulk_delete_documents",
  "collectionId": "<collection-uuid>",
  "deleteBy": "document_id",
  "targetIds": "[\"document-1\",\"document-2\"]"
}
```

Use `file_id` and file IDs to target all chunks belonging to the specified files. Review the returned `failedIds` before treating the batch as complete.

### Example Workflow

The accompanying workflow contains examples for all ten operations with manual triggers and Log nodes. Configure its connection, file URL, and IDs for your environment before running the relevant branch. Review the targets of deletion operations before executing them.

```fusion-workflow
src: example.workflow.json
title: Manage collections and documents with LangConnect
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Keep real API tokens out of shared examples and workflow exports. Use HTTPS for remote API connections and file downloads. Document uploads send the downloaded file and configured metadata to the LangConnect server.
<!-- /SECTION: security -->
