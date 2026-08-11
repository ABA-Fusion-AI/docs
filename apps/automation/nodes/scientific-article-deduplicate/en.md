---
node_id: "scientific-article-deduplicate"
title: "Scientific Article Deduplicate"
description: "Generate stable article keys and exclude stored or cross-provider duplicates."
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
# Scientific Article Deduplicate

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Generate stable article keys and exclude stored or cross-provider duplicates.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `existingKeys` | `array | string` | No | `[]` | Article keys already stored in the research catalog. |
| `includeDuplicates` | `boolean` | No | `false` | Include rejected duplicate records in the output. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An article array or an object containing a results array.
- **Output:** Unique results, articleKeys, input/count statistics, and optional duplicateResults.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

Identity preference is DOI, then provider record ID, then a deterministic title/year/first-author hash.
<!-- /SECTION: behavior -->

