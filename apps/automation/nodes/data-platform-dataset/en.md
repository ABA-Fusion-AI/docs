---
node_id: "data-platform-dataset"
title: "Data Platform Dataset"
description: "Ensure, create, retrieve, list, update, archive, and delete Data Platform datasets."
category: "databases"
subcategory: "Data Platform"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags: [data-platform, rag, dataset, knowledge-base]
related_nodes: [data-platform-document-upload, data-platform-document, data-platform-retrieval]
---

<!-- SECTION: overview -->
# Data Platform Dataset

Use this node to manage the dataset that owns documents and defines their retrieval scope. For reusable workflows, prefer `ensure`: it accepts a configured ID, reuses an exact case-insensitive name match, or creates the dataset when neither exists.
<!-- /SECTION: overview -->

## Prerequisites

- Set `DATA_PLATFORM_API_URL` on the Automation **engine**, for example `http://localhost:8001/api/v1`.
- Start the workflow from an authenticated Automation session. There are no URL or authentication inputs on the node.

## Operations

| Operation | Required fields | Result |
|---|---|---|
| `ensure` | `datasetId`, or `name` | Existing or newly created dataset. `datasetId` takes priority. |
| `create` | `name` | Newly created dataset. |
| `get` | `datasetId` | Dataset; optionally includes documents. |
| `list` | — | `{ items, total, offset, limit }`. |
| `update` | `datasetId` | Updated dataset. |
| `delete` | `datasetId` | Data Platform deletion response. This is destructive. |

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `operation` | enum | `list` | Dataset operation. |
| `datasetId` | string | — | Dataset UUID. Expressions are supported. |
| `name` | string | — | Dataset name; required by `create` and by `ensure` without an ID. |
| `description` | string | — | Human-readable purpose. |
| `tags` | array | — | String tags. Blank entries are removed. |
| `datasetMetadata` | object | — | Free-form metadata stored on the dataset. |
| `mergeDatasetMetadata` | boolean | `false` | On update, fetches existing metadata and shallow-merges supplied keys instead of replacing the object. |
| `status` | enum | — | `active` or `archived`. |
| `includeDocuments` | boolean | `false` | Include linked documents for `get`/`list`. |
| `offset` / `limit` | number | `0` / `20` | List pagination; limit is 1–100. |
| `search` / `tag` | string | — | List filters. |
| `sortBy` | enum | `created_at` | `created_at`, `updated_at`, `name`, or `status`. |
| `sortOrder` | enum | `desc` | `asc` or `desc`. |

## Recommended pattern

Label the node `ensureKnowledgeDataset`, use `operation: ensure`, and pass its ID downstream with:

```text
{{ outputs.ensureKnowledgeDataset.success.id }}
```

`ensure` avoids creating a new dataset on every manual run. If multiple datasets share the same name, it returns the oldest exact match; use an explicit `datasetId` when identity must be unambiguous.

## Errors and safety

Missing required IDs/names, authorization failures, unreachable services, and Data Platform validation failures are emitted through `error`. Before `delete`, make sure the expression resolves to the intended UUID.

<!-- SECTION: examples -->
## Example workflow

```fusion-workflow
src: example.workflow.json
title: Ensure a dataset and inspect the result
```
<!-- /SECTION: examples -->
