---
node_id: "mssql"
title: "Microsoft SQL Server"
description: "Run parameterized SQL Server operations."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [mssql, sql-server, sql, database]
---

# Microsoft SQL Server

Provides structured insert, update, delete, and select operations plus a trusted raw-query mode.

## Security

Structured operations validate table and column identifiers and bind all values. Prefer these operations whenever possible.

`SQL Query` requires `sqlQueryParams.acknowledgeRisk: true` and accepts one statement. Use `@name` placeholders in the statement and provide corresponding keys through `binds`. Parameter names are validated. Never concatenate untrusted workflow values into the statement.

## Workflow example

See [example.workflow.json](./example.workflow.json).
