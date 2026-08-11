---
node_id: "scientific-search-pages-combine"
title: "Scientific Search Pages Combine"
description: "Combine normalized scholarly provider pages and preserve pagination metadata."
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
# Scientific Search Pages Combine

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Combine normalized scholarly provider pages and preserve pagination metadata.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `keywords` | `string | array` | No | `—` | Keywords forwarded to relevance filtering. |
| `specificFilters` | `object` | No | `{}` | Structured research constraints forwarded downstream. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An array of normalized provider search responses.
- **Output:** One result list with provider names, counts, filters, and providerPagination metadata.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

This node combines pages; duplicate removal is handled by Scientific Article Deduplicate.
<!-- /SECTION: behavior -->

