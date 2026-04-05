---
node_id: "azure-sql-schema-manage"
title: "Azure SQL Schema Manage"
description: "Execute schema and DDL operations for Azure SQL"
category: "integrations"
subcategory: "azure"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
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
- `schemaName` (default `dbo`)
- `columns`, `ifNotExists` for `createTable`
- `ifExists` for `dropTable`
- `column` for `addColumn`
- `columnName` for `dropColumn`

Outputs:

- `executeSql`: query result object
- DDL ops: `{ rowsAffected }`
