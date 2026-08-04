---
node_id: "vast-ai-create-template"
title: "VastAI: Create Template"
description: "Creates a new template for launching instances. If an identical template already exists, returns the existing template instead of creating a duplicate."
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
# VastAI: Create Template

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Creates a new template for launching instances. If an identical template already exists, returns the existing template instead of creating a duplicate.
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
title: Use VastAI: Create Template in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->
