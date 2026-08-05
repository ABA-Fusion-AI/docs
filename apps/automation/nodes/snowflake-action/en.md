---
node_id: "snowflake-action"
title: "Snowflake Action"
description: "Execute queries and manage Snowflake data warehouse objects via the Snowflake SQL REST API."
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
# Snowflake Action

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Execute queries and manage Snowflake data warehouse objects via the Snowflake SQL REST API.

The node supports custom queries plus database, schema, table, column, and query-status discovery operations.
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

Custom SQL requires `acknowledgeRisk: true`. Put dynamic values in Snowflake SQL API bindings rather than concatenating them into SQL:

```json
{
  "sql": "SELECT * FROM orders WHERE customer_id = ?",
  "bindings": "{\"1\":{\"type\":\"TEXT\",\"value\":\"customer-1\"}}",
  "acknowledgeRisk": true
}
```

`listSchemas`, `listTables`, and `listColumns` safely quote object identifiers. Account identifiers and statement handles are validated before URL construction.

This version replaces the legacy `username` and `password` fields with `accessToken` and `tokenType`. Supported token types are `OAUTH`, `KEYPAIR_JWT`, and `PROGRAMMATIC_ACCESS_TOKEN`. Existing workflows using Basic authentication must migrate their Snowflake credentials before deployment.

Store tokens in Fusion's credential system and use a least-privilege Snowflake role. Snowflake does not support bindings in multi-statement SQL API requests.
<!-- /SECTION: security -->
