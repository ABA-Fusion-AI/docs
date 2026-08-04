---
node_id: "vast-ai-add-network-disk"
title: "VastAI: Add Network Disk"
description: "Adds a network disk to be used to create network volume offers, or adds machines to an existing network disk."
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
# VastAI: Add Network Disk

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Adds a network disk to be used to create network volume offers, or adds machines to an existing network disk.
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
title: Use VastAI: Add Network Disk in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->
