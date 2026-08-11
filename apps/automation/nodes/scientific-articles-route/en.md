---
node_id: "scientific-articles-route"
title: "Scientific Articles Route"
description: "Route non-empty scientific result sets to processing and empty sets to checkpointing."
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
# Scientific Articles Route

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Route non-empty scientific result sets to processing and empty sets to checkpointing.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `resultsField` | `string` | No | `results` | Input field containing the article array. |
| `minimumArticles` | `number` | No | `1` | Minimum records required for Has Articles. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An object containing the configured results array.
- **Output:** Has Articles, Empty, or Error without requiring a JavaScript condition.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

Use Empty to advance a pagination checkpoint without running PDF or Sheets article stages.
<!-- /SECTION: behavior -->

