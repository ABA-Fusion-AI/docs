---
node_id: "scientific-pdf-fetch"
title: "Scientific PDF Fetch"
description: "Resolve and download an openly accessible scientific article PDF."
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
# Scientific PDF Fetch

> **Category:** Scientific Research → Research Workflow | **Type:** Workflow Node

Resolve and download an openly accessible scientific article PDF.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `pdfUrl` | `string` | No | `—` | Optional direct PDF URL. |
| `maxSizeMb` | `number` | No | `25` | Maximum accepted PDF size. |
| `timeoutMs` | `number` | No | `30000` | Download timeout in milliseconds. |
| `strict` | `boolean` | No | `false` | Send unavailable PDFs to Error instead of Unavailable. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** One normalized article, usually from a batch loop.
- **Output:** Success metadata, a File output with base64, Unavailable with article context, or Error.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: behavior -->
## Behavior and usage

The node follows safe HTTP redirects, blocks private-network targets, enforces size/time limits, and does not bypass paywalls.
<!-- /SECTION: behavior -->

