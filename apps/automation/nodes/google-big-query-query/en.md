---
node_id: "google-big-query-query"
title: "Google BigQuery - Query"
description: "Run parameterized GoogleSQL and retrieve BigQuery query results"
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - integration
  - google-cloud
  - bigquery
  - sql
  - query
related_nodes:
  - google-big-query-project
  - google-big-query-dataset
  - google-big-query-table
  - google-big-query-job
---

<!-- SECTION: header -->

# Google BigQuery - Query

> **Category:** Data | **Type:** Action Node

Query operations: `query`, `getQueryResults`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Core Parameters

| Parameter         | Type      | Required | Default            | Description                            |
| ----------------- | --------- | -------- | ------------------ | -------------------------------------- |
| `clientId`        | `string`  | ✅ Yes   | —                  | Google OAuth client ID                 |
| `clientSecret`    | `string`  | ✅ Yes   | —                  | Google OAuth client secret             |
| `redirectUri`     | `string`  | ❌ No    | `http://localhost` | OAuth redirect URI                     |
| `refreshToken`    | `string`  | ❌ No    | —                  | OAuth refresh token                    |
| `accessToken`     | `string`  | ❌ No    | —                  | OAuth access token                     |
| `operation`       | `enum`    | ❌ No    | `query`            | `query` or `getQueryResults`           |
| `projectId`       | `string`  | ✅ Yes   | —                  | Google Cloud project ID                |
| `acknowledgeRisk` | `boolean` | ✅ Yes\* | `false`            | Confirms that submitted SQL is trusted |
| `jobId`           | `string`  | ✅ Yes\* | —                  | Job ID used by `getQueryResults`       |

\* `acknowledgeRisk: true` is required for `query`; `jobId` is required for `getQueryResults`.

### Operation-Specific Parameters

| Parameter            | Used By           | Notes                                                                                                   |
| -------------------- | ----------------- | ------------------------------------------------------------------------------------------------------- |
| `queryData`          | `query`           | Required query text plus cache, dry-run, result-limit, timeout, default-dataset, and parameter settings |
| `queryResultsParams` | `getQueryResults` | Optional pagination, start index, timeout, and location settings                                        |

### Security

Query mode accepts one SQL statement. Prefer GoogleSQL parameters: use `@name` with `parameterMode: named`, or `?` with `parameterMode: positional`, and supply `queryParameters`. Never concatenate untrusted values into `queryData.query`.

### Outputs

The node returns the corresponding BigQuery query or query-results API response.

<!-- /SECTION: configuration -->

## Workflow example

See the focused [safe-query workflow example](./example.workflow.json).
