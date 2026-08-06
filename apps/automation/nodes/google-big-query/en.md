---
node_id: "google-big-query"
title: "Google BigQuery"
description: "Manage BigQuery resources and execute parameterized GoogleSQL."
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-05"
author: "Fusion Team"
tags: [google-cloud, bigquery, sql, analytics]
---

# Google BigQuery

Manages BigQuery datasets, tables, jobs, and query execution.

## Security

Raw SQL submitted by `query` or through an `insertJob` query configuration requires `acknowledgeRisk: true`. Query mode accepts one statement.

Prefer GoogleSQL parameters: use `@name` with `parameterMode: named`, or `?` with `parameterMode: positional`, and supply `queryParameters`. Named parameters are validated and must be unique. Never concatenate webhook or other untrusted values into `query`.

## Workflow example

See [example.workflow.json](./example.workflow.json).
