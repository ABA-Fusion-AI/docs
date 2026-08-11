---
node_id: "google-big-query"
title: "Google BigQuery (Legacy)"
description: "Compatibility-only BigQuery node for workflows created before the resource-specific nodes"
category: "data"
subcategory: "databases"
version: "1.1.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - google-cloud
  - bigquery
  - sql
  - analytics
  - legacy
  - compatibility
related_nodes:
  - google-big-query-project
  - google-big-query-dataset
  - google-big-query-table
  - google-big-query-job
  - google-big-query-query
---

# Google BigQuery (Legacy)

> **Compatibility only:** Existing workflows can continue using `google-big-query`. For new workflows, choose one of the focused nodes below.

- [`google-big-query-project`](../google-big-query-project/en.md) — projects and service accounts
- [`google-big-query-dataset`](../google-big-query-dataset/en.md) — datasets
- [`google-big-query-table`](../google-big-query-table/en.md) — tables and table data
- [`google-big-query-job`](../google-big-query-job/en.md) — jobs
- [`google-big-query-query`](../google-big-query-query/en.md) — queries and query results

## Security

Raw SQL submitted by `query` or through an `insertJob` query configuration requires `acknowledgeRisk: true`. Query mode accepts one statement.

Prefer GoogleSQL parameters: use `@name` with `parameterMode: named`, or `?` with `parameterMode: positional`, and supply `queryParameters`. Named parameters are validated and must be unique. Never concatenate webhook or other untrusted values into `query`.

## Workflow example

The [legacy workflow example](./example.workflow.json) remains available for compatibility.
