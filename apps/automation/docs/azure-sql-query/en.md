---
node_id: "azure-sql-query"
title: "Azure SQL Query"
description: "Execute parameterized SQL queries against Azure SQL"
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

Output:
`{ recordset, recordsets, rowsAffected, output }`
