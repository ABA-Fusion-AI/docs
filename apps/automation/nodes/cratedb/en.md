---
node_id: "cratedb"
title: "CrateDB"
description: "Distributed SQL database queries with CrateDB"
category: "peer-only"
subcategory: "integrations"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [integration, peer-only]
related_nodes: []
---

<!-- SECTION: overview -->
# CrateDB

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Distributed SQL database queries with CrateDB
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
title: Use CrateDB in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

CrateDB SQL is a raw-query interface and requires `acknowledgeRisk: true`. The node accepts exactly one statement and verifies that its type matches the selected operation: read-only query statements for `query`, and `INSERT` for `insert`.

Use `?` placeholders and provide values through the `args` array. Never interpolate webhook or other untrusted values into `sql`. Store credentials in Fusion's credential system.
<!-- /SECTION: security -->
