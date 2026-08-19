---
node_id: "data-platform-document"
title: "Data Platform Document"
description: "Read, list, reingest, cancel, confirm, or delete Data Platform documents."
category: "databases"
subcategory: "Data Platform"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags: [data-platform, rag, document]
related_nodes: [data-platform-document-upload, data-platform-document-wait, data-platform-chunk]
---

<!-- SECTION: overview -->
# Data Platform Document

Manages existing document records. Uploading new bytes belongs to **Data Platform Document Upload**; waiting for ingestion belongs to **Data Platform Document Wait**.
<!-- /SECTION: overview -->

## Operations

| Operation | Required fields | Description |
|---|---|---|
| `get` | `documentId` | Retrieve one document. |
| `list` | — | List/filter documents; returns `{ items, total, offset, limit }`. |
| `reingest` | `documentId` | Queue the existing document for ingestion again. |
| `cancel` | `documentId` | Cancel active processing. |
| `delete` | `documentId` | Delete the document. This is destructive. |
| `confirm` | `documentId`, `datasetId` | Confirm a document against a dataset. Mainly retained for compatible Data Platform flows. |
| `getSourceUrl` | `documentId` | Return a source/download URL. |
| `getStatuses` | — | Return supported processing statuses. |

## List parameters

`offset` defaults to `0`; `limit` defaults to `20` and is capped at `100`. Filter with `processingStatus`, `fileType`, `datasetId`, or `search`. Sort by `created_at`, `updated_at`, `filename`, `file_size`, or `processing_status`, in `asc` or `desc` order.

## Common expressions

```text
{{ outputs.uploadDocument.success.data.id }}
{{ outputs.ensureDataset.success.id }}
```

`reingest`, `cancel`, and `delete` mutate server state. Route their error outputs and avoid unresolved or user-controlled IDs without validation.

<!-- SECTION: examples -->
## Example workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve a document and inspect it
```
<!-- /SECTION: examples -->
