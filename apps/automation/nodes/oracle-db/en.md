---
node_id: "oracle-db"
title: "Oracle Database"
description: "Run parameterized Oracle Database operations."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [oracle, sql, database]
---

# Oracle Database

Provides structured insert, update, delete, and select operations plus trusted raw SQL.

## Security

Structured operations validate schema, table, and column identifiers and bind every value. Prefer them whenever possible.

`SQL Query` requires `sqlQueryParams.acknowledgeRisk: true` and accepts one statement. Use named `:name` or positional bind placeholders and provide values through `binds`. Named bind keys are validated. Never concatenate untrusted workflow values into Oracle SQL.

## Workflow example

See [example.workflow.json](./example.workflow.json).
