---
node_id: "azure-sql-bulk-insert"
title: "Azure SQL Bulk Insert"
description: "Perform bulk insert operations into Azure SQL tables"
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
  - bulk-insert
related_nodes:
  - azure-sql-query
---

# Azure SQL Bulk Insert

> **Category:** Integrations | **Type:** Action Node

Performs high-throughput insert using `mssql` bulk API.

## Configuration

| Parameter          | Type      | Required | Description                                                              |
| ------------------ | --------- | -------- | ------------------------------------------------------------------------ |
| `connectionString` | `string`  | ✅ Yes   | Azure SQL connection string                                              |
| `tableName`        | `string`  | ✅ Yes   | Destination table                                                        |
| `tableColumns`     | `array`   | ✅ Yes   | Column definitions (`name`, `type`, `nullable`, optional size/precision) |
| `rows`             | `array`   | ✅ Yes   | Row values array                                                         |
| `keepNulls`        | `boolean` | ❌ No    | Preserve nulls during bulk insert                                        |

Output:
`{ rowsAffected }`
