---
node_id: "azure-sql-stored-procedure"
title: "Azure SQL Stored Procedure"
description: "Execute stored procedures with input/output parameters"
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
  - stored-procedure
related_nodes:
  - azure-sql-query
---

# Azure SQL Stored Procedure

> **Category:** Integrations | **Type:** Action Node

Executes a stored procedure with typed input/output params.

## Configuration

| Parameter          | Type     | Required | Description                           |
| ------------------ | -------- | -------- | ------------------------------------- |
| `connectionString` | `string` | ✅ Yes   | Azure SQL connection string           |
| `procedureName`    | `string` | ✅ Yes   | Procedure to execute                  |
| `parameters`       | `array`  | ❌ No    | `{ name, value, type?, direction }[]` |

Supported `type` values include:
`VarChar`, `NVarChar`, `Int`, `BigInt`, `Bit`, `Float`, `DateTime`, `Date`, `UniqueIdentifier`, `SmallInt`, `TinyInt`, `Real`, `Money`, `VarBinary`.

`direction`:

- `input` (default)
- `output`

Output:
`{ recordsets, recordset, rowsAffected, output, returnValue }`
