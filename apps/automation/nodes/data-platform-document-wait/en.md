---
node_id: "data-platform-document-wait"
title: "Data Platform Document Wait"
description: "Wait for automatic document ingestion to complete and fail the workflow when processing fails."
category: "databases"
subcategory: "Data Platform"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags: [data-platform, rag, ingestion, polling]
related_nodes: [data-platform-document-upload, data-platform-retrieval, data-platform-chunk]
---

<!-- SECTION: overview -->
# Data Platform Document Wait

Polls a document until it becomes `completed`. This is the synchronization point between upload and nodes that require searchable chunks.
<!-- /SECTION: overview -->

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `documentId` | string | — | Required document UUID, usually `{{ outputs.uploadDocument.success.data.id }}`. |
| `pollIntervalMs` | number | `2000` | Delay between status checks; 250–60,000 ms. |
| `timeoutMs` | number | `1800000` | Total wait limit; 1 second–24 hours. |

## Status behavior

| Status | Behavior |
|---|---|
| `pending`, `queued`, `processing`, `partitioning`, `chunking`, `summarising`, `vectorization` | Continue polling. |
| `completed` | Emit the final document through `success`. |
| `failed`, `cancelled` | Throw an error containing available processing details. |
| `*_awaiting_approval` | Throw because Automation upload is auto-only. |
| Unknown or empty | Throw instead of waiting forever. |

Always connect the `error` output to a log, notification, or retry policy. Increasing `timeoutMs` will not fix a failed embedding or summarization service; inspect `processing_details` in the emitted error.

<!-- SECTION: examples -->
## Example workflow

```fusion-workflow
src: example.workflow.json
title: Upload and wait for automatic ingestion
```
<!-- /SECTION: examples -->
