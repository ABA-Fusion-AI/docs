---
node_id: "scientific-articles-extract"
title: "Scientific Articles Extract"
description: "Extract accepted articles into a plain array for loops and downstream processing."
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
# Scientific Articles Extract

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Extract accepted articles into a plain array for loops and downstream processing.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `resultsField` | `string` | No | `results` | Input field containing accepted articles. |
| `allowEmpty` | `boolean` | No | `false` | Return an empty array instead of Error. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** A relevance-filter response or another object containing an article array.
- **Output:** The extracted article array on Success, or Error when empty and allowEmpty is disabled.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

This replaces a custom Function node before batch or PDF loops.
<!-- /SECTION: behavior -->

