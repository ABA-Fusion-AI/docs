---
node_id: "google-perspective"
title: "Google Perspective"
description: "Analyze comment attributes or submit suggested moderation scores with the Perspective API."
category: "web-search"
subcategory: "search-reference"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [google, perspective, moderation, toxicity]
related_nodes: [google-translate-action]
---

<!-- SECTION: overview -->
# Google Perspective

> **Category:** Web Search&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Analyze text for toxicity, threats, spam, profanity, identity attacks, and other moderation attributes. The node also supports Perspective's suggested-score operation.
<!-- /SECTION: overview -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|---|---|---:|---|---|
| `apiKey` | string | Yes | — | Perspective API key. |
| `operation` | enum | No | `analyzeComment` | Analyze a comment or submit a suggested score. |
| `text` | string | Yes | — | Text to process. |
| `languages` | string | No | `en` | Comma-separated language codes. |
| `attributes` | array | No | `TOXICITY` | Moderation attributes to score. |
| `doNotStore` | boolean | No | `true` | Ask the API not to retain the comment. |
| `scoreValue` | number | For suggestScore | `0` | Suggested probability from 0 to 1. |
| `clientToken` | string | No | — | Client correlation token. |
| `sessionId` | string | No | — | Identifier joining related requests. |
<!-- /SECTION: configuration -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Optional upstream event that triggers the configured request.
- **Success:** Operation, analyzed text, normalized scores, detected languages, and raw API response.
- **Error:** Perspective API error details.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Analyze a comment for toxicity
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store the API key as a workflow secret and keep `doNotStore` enabled unless retention is explicitly required.
<!-- /SECTION: security -->
