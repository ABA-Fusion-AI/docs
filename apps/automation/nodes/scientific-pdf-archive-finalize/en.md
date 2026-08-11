---
node_id: "scientific-pdf-archive-finalize"
title: "Scientific PDF Archive Finalize"
description: "Attach Drive metadata or preserve the article when PDF archival is unavailable or fails."
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
# Scientific PDF Archive Finalize

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Attach Drive metadata or preserve the article when PDF archival is unavailable or fails.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `status` | `enum` | No | `unavailable` | archived, unavailable, or upload-error. |
| `article` | `object` | No | `—` | Original article from the PDF loop. |
| `pdf` | `object` | No | `—` | Downloaded PDF metadata. |
| `reason` | `string` | No | `—` | Fallback failure reason. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Google Drive upload output, PDF unavailable output, or an error response.
- **Output:** The original article enriched with PDF status and Drive metadata, with base64 removed.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

Use this node on every PDF branch so an unavailable file never blocks article metadata storage.
<!-- /SECTION: behavior -->

