---
node_id: "scientific-sheets-rows"
title: "Scientific Google Sheets Rows"
description: "Map archived scientific articles to the standard 19-column spreadsheet format."
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
# Scientific Google Sheets Rows

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Map archived scientific articles to the standard 19-column spreadsheet format.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

This node has no user-configurable parameters; it applies the standard scientific archive column mapping.
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** An article array, including results accumulated by the PDF archive loop.
- **Output:** header, articleRows, and count for Google Sheets Values nodes.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

The final Article Key column supports exact duplicate prevention on later research runs.
<!-- /SECTION: behavior -->

