---
node_id: "snowflake"
title: "Snowflake"
description: "Execute queries and manage schemas in Snowflake cloud data warehouse via the Snowflake SQL REST API."
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
# Snowflake

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Execute queries and manage schemas in Snowflake cloud data warehouse via the Snowflake SQL REST API.

This node uses OAuth, key-pair JWT, or programmatic access tokens. Select the matching `tokenType` and store the token in Fusion's credential system.
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
title: Use Snowflake in a workflow
```
<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Custom SQL requires `acknowledgeRisk: true`. Use `?` placeholders and the `bindings` JSON object for dynamic values:

```json
{
  "sql": "SELECT * FROM events WHERE tenant_id = ? AND status = ?",
  "bindings": "{\"1\":{\"type\":\"TEXT\",\"value\":\"tenant-1\"},\"2\":{\"type\":\"TEXT\",\"value\":\"active\"}}",
  "acknowledgeRisk": true
}
```

Binding positions must be positive integers. Binding values must be strings and types are restricted to supported Snowflake SQL API types. Database names used by `listSchemas` are quoted safely. Account identifiers and statement handles are validated before being used in URLs.

Snowflake does not support bindings in multi-statement SQL API requests. Prefer one statement per execution and use a least-privilege role.
<!-- /SECTION: security -->
