---
node_id: "azure-sql-schema-manage"
title: "Azure SQL Schema Manage"
description: "Execute schema and DDL operations for Azure SQL"
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
  - schema
related_nodes:
  - azure-sql-query
---

# Azure SQL Schema Manage

> **Category:** Integrations | **Type:** Action Node

Operations:

- `executeSql`
- `createTable`
- `dropTable`
- `addColumn`
- `dropColumn`

## Configuration

Required:

- `connectionString`
- `operation`
- `tableName` for table/column operations

Operation-specific fields:

- `sql`, `parameters` for `executeSql`
- `acknowledgeRisk: true` for `executeSql`
- `schemaName` (default `dbo`)
- `columns`, `ifNotExists` for `createTable`
- `ifExists` for `dropTable`
- `column` for `addColumn`
- `columnName` for `dropColumn`

Custom SQL values must use named parameters. Structured table and column identifiers are bracket-quoted. Column `default` values are treated as escaped string literals; raw default expressions from older workflows are no longer executed.

```fusion-workflow
src: example.workflow.json
title: Create an Azure SQL table safely
```

Outputs:

- `executeSql`: query result object
- DDL ops: `{ rowsAffected }`
