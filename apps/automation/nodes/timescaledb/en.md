---
node_id: "timescaledb"
title: "TimescaleDB"
description: "Query and manage time-series data in TimescaleDB."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [timescaledb, postgresql, sql, time-series]
---

# TimescaleDB

Runs time-series queries and manages TimescaleDB hypertables.

## Security

The structured hypertable, bucket, and chunk operations quote identifiers and bind values. Raw `query` and `insert` operations require `acknowledgeRisk: true`, accept one statement, and enforce a read or insert statement respectively.

Use `$1`, `$2`, and subsequent placeholders with the `params` array. Never concatenate webhook or other untrusted values into `sql`.

## Workflow example

See [example.workflow.json](./example.workflow.json).
