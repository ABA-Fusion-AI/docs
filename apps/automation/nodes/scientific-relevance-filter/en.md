---
node_id: "scientific-relevance-filter"
title: "Scientific Relevance Filter"
description: "Filter scholarly results using title/abstract keywords and scientific metadata constraints."
category: "scientific-research"
subcategory: "research-workflow"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:
  - scientific-research
  - scholarly-articles
  - research-automation
related_nodes:
  - scientific-research-state-prepare
  - scientific-relevance-filter
  - scientific-pagination-checkpoint
---

<!-- SECTION: header -->
# Scientific Relevance Filter

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Filter scholarly results using title/abstract keywords and scientific metadata constraints.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `keywords` | `string | array` | No | `—` | Topic keywords to match. |
| `matchMode` | `enum` | No | `any` | Require any or all keywords. |
| `searchFields` | `enum` | No | `titleAndAbstract` | Search title, abstract, or both. |
| `caseSensitive` | `boolean` | No | `false` | Use case-sensitive matching. |
| `wholePhrase` | `boolean` | No | `true` | Match multi-word keywords as phrases. |
| `specificFilters` | `object` | No | `{}` | Journal, date, citation, author, access, DOI, and extensible conditions. |
| `specificMatchMode` | `enum` | No | `all` | Require all or any specific conditions. |
| `includeRejected` | `boolean` | No | `false` | Return rejected results with accepted results. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Normalized scientific results plus optional query-derived filters.
- **Output:** Accepted results, rejected count, filter details, matched keywords, and matchScore metadata.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

matchScore is matched unique keywords divided by total keywords. The latest option is a rolling date filter, not a sort operation.
<!-- /SECTION: behavior -->

