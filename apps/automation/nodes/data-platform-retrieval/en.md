---
node_id: "data-platform-retrieval"
title: "Data Platform Retrieval"
description: "Run semantic, hybrid, or keyword retrieval across a dataset or selected documents."
category: "databases"
subcategory: "Data Platform"
version: "1.0.0"
language: "en"
last_updated: "2026-08-19"
author: "Fusion Team"
tags: [data-platform, rag, retrieval, semantic-search, hybrid-search]
related_nodes: [data-platform-dataset, data-platform-document-wait, data-platform-chunk]
---

<!-- SECTION: overview -->
# Data Platform Retrieval

Searches completed Data Platform documents. Scope the search to one dataset or an explicit set of document IDs, then choose keyword, semantic, or hybrid ranking.
<!-- /SECTION: overview -->

## Required sequence

For newly uploaded content, run:

```text
Document Upload → Document Wait → Retrieval
```

Starting retrieval immediately after upload can return no matches because finalization only queues ingestion.

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `query` | string | — | Required search question or phrase. |
| `searchMode` | enum | `hybrid` | `semantic`, `hybrid`, or `keyword`. |
| `scope` | enum | `dataset` | Search a `dataset` or selected `documents`. |
| `datasetId` | string | — | Required when scope is `dataset`. |
| `documentIds` | string[] | — | Required and non-empty when scope is `documents`. |
| `topK` | number | `5` | Results returned, 1–20. |
| `candidateK` | number | — | Candidate pool before reranking, 1–200. |
| `queryExpansion` | enum | `none` | `none` or `multi_query`. |
| `rerank` | boolean | `true` | Apply Data Platform reranking. |
| `contentTypes` | string[] | — | Filter chunk content types. |
| `fileTypes` | string[] | — | Filter source types such as `pdf` or `md`. |
| `metadataFilters` | object | — | Exact metadata filters supported by Data Platform. |
| `createdFrom` / `createdTo` | string | — | Creation-time boundaries. |
| `updatedFrom` / `updatedTo` | string | — | Update-time boundaries. |
| `debug` | boolean | `false` | Include backend retrieval diagnostics. |
| `timeoutMs` | number | `60000` | Request timeout. |

## Choosing a search mode

- `keyword` is best for exact identifiers and does not depend on semantic similarity.
- `semantic` is best for paraphrases and conceptual questions.
- `hybrid` combines lexical and semantic evidence and is the recommended general default.

Use a larger `candidateK` than `topK` when reranking. For example, retrieve 25 candidates and return the best 5.

## Example

```json
{
  "query": "What does Data Platform do with uploaded documents?",
  "searchMode": "hybrid",
  "scope": "dataset",
  "datasetId": "{{ outputs.ensureKnowledgeDataset.success.id }}",
  "topK": 5,
  "candidateK": 25,
  "rerank": true
}
```

<!-- SECTION: examples -->
## Example workflow

```fusion-workflow
src: example.workflow.json
title: Search a dataset and inspect ranked results
```
<!-- /SECTION: examples -->
