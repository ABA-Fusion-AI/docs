---
node_id: "cosmos-query"
title: "Cosmos Query"
description: "Execute parameterized Azure Cosmos DB for NoSQL queries."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [azure, cosmos-db, nosql, security]
---

# Cosmos Query

Executes Cosmos DB for NoSQL query specifications. Custom query text requires `acknowledgeRisk: true`.

Use `@name` placeholders and provide values in `query.parameters`. Parameter names must include `@`, follow identifier syntax, and be unique. Values may be any valid JSON value.

```json
{
  "statement": "SELECT * FROM c WHERE c.tenantId = @tenantId",
  "parameters": [{ "name": "@tenantId", "value": "tenant-1" }]
}
```

Full SQL text is not written to logs. Use read-only credentials for query-only workflows.

```fusion-workflow
src: example.workflow.json
title: Run a parameterized Cosmos query
```
