---
node_id: "vast-ai-reset-apikey"
title: "VastAI: Reset API Key"
description: "Resets the API key for the authenticated user. This operation generates a new API key and invalidates the old one."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-08-04"
author: "Fusion Team"
tags: [integration, peer-only]
related_nodes: []
---

<!-- SECTION: overview -->
# VastAI: Reset API Key

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Resets the API key for the authenticated user. This operation generates a new API key and invalidates the old one.
<!-- /SECTION: overview -->

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

- **Input:** Data and configuration supplied by the preceding workflow node.
- **Success:** The integration response is emitted through the success output.
- **Error:** Validation, authentication, network, or remote-service failures are emitted through the error output.
<!-- /SECTION: inputs-outputs -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use VastAI: Reset API Key in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->
