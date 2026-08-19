---
node_id: "data-platform-chunk"
title: "Data Platform Chunk"
description: "List, retrieve, delete, deactivate, and download assets from Data Platform chunks."
category: "databases"
subcategory: "Data Platform"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags: [data-platform, rag, chunk, asset]
related_nodes: [data-platform-document-wait, data-platform-document, data-platform-retrieval]
---

<!-- SECTION: overview -->
# Data Platform Chunk

Inspects and manages the chunks produced during document ingestion. Run it only after **Data Platform Document Wait** succeeds when working with a newly uploaded document.
<!-- /SECTION: overview -->

## Operations

| Operation | Required fields | Description |
|---|---|---|
| `list` | `documentId` | List document chunks; optionally filter active/version. |
| `get` | `chunkId` | Retrieve one chunk. |
| `delete` | `chunkId` | Permanently delete one chunk. |
| `deactivate` | `documentId` | Soft-delete/deactivate matching document chunks. |
| `deleteDocument` | `documentId` | Hard-delete all chunks for a document. |
| `downloadAsset` | `documentId`, `chunkVersion`, `chunkIndex`, `filename` | Download a chunk asset as base64. |

## Parameters

`activeOnly` defaults to `true`. `chunkVersion` is one-based; `chunkIndex` is zero-based. `timeoutMs` defaults to 60 seconds.

## Asset output

`downloadAsset` returns an Automation-friendly base64 object:

```json
{
  "base64": "...",
  "encoding": "base64",
  "filename": "figure-1.png",
  "contentType": "image/png",
  "sizeBytes": 48231,
  "documentId": "document-uuid",
  "chunkVersion": 1,
  "chunkIndex": 0
}
```

`delete`, `deactivate`, and `deleteDocument` are state-changing operations. In particular, `deleteDocument` uses hard deletion; validate the target document ID before running it.

<!-- SECTION: examples -->
## Example workflow

```fusion-workflow
src: example.workflow.json
title: List chunks and inspect the result
```
<!-- /SECTION: examples -->
