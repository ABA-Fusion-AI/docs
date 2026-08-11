---
node_id: "scientific-research-state-prepare"
title: "Scientific Research State Prepare"
description: "Prepare a research query, load stored article keys, and resume provider pagination."
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
# Scientific Research State Prepare

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Prepare a research query, load stored article keys, and resume provider pagination.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `query` | `string` | Yes | `—` | Research query extracted from the user prompt. |
| `keywords` | `string` | Yes | `—` | Comma-separated relevance keywords. |
| `specificFilters` | `object` | No | `{}` | Date, journal, citation, access, or author constraints. |
| `limit` | `number` | No | `10` | Results requested from each provider page. |
| `resumeMode` | `enum` | No | `resume` | Resume saved offsets or restart at zero. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Merged Google Sheets responses for the Articles catalog and SearchState checkpoint table.
- **Output:** Prepared query, stable query key, existing article keys, saved state rows, and provider offsets.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

The query-key namespace is versioned so offsets created by incompatible ordering strategies are not reused.
<!-- /SECTION: behavior -->

