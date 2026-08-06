---
node_id: "azure-sql-query"
title: "Azure SQL Query"
description: "Execute parameterized SQL queries against Azure SQL"
category: "integrations"
subcategory: "azure"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags:
  - integration
  - azure
  - sql
  - query
related_nodes:
  - azure-sql-stored-procedure
  - azure-sql-schema-manage
---

# Azure SQL Query

> **Category:** Integrations | **Type:** Action Node

Runs a SQL query with optional parameters.

## Configuration

| Parameter          | Type     | Required | Description                          |
| ------------------ | -------- | -------- | ------------------------------------ |
| `connectionString` | `string` | ✅ Yes   | Azure SQL connection string          |
| `query`            | `string` | ✅ Yes   | SQL statement                        |
| `parameters`       | `array`  | ❌ No    | `{ name, value }[]` input parameters |
| `acknowledgeRisk`  | `boolean` | ✅ Yes  | Must be `true` because custom SQL is executed |

Use named placeholders such as `@tenantId` and supply their values through `parameters`. Parameter names must start with a letter or underscore and contain only letters, numbers, and underscores. Never concatenate workflow data into `query`.

```fusion-workflow
src: example.workflow.json
title: Run a parameterized Azure SQL query
```

Output:
`{ recordset, recordsets, rowsAffected, output }`
