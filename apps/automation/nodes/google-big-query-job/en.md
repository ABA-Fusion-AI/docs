---
node_id: "google-big-query-job"
title: "Google BigQuery - Job"
description: "List, inspect, submit, cancel, and delete BigQuery jobs"
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
  - job
related_nodes:
  - google-big-query-project
  - google-big-query-dataset
  - google-big-query-table
  - google-big-query-query
---

<!-- SECTION: header -->

# Google BigQuery - Job

> **Category:** Data | **Type:** Action Node

Job operations: `listJobs`, `getJob`, `insertJob`, `cancelJob`, `deleteJob`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Core Parameters

| Parameter         | Type      | Required | Default            | Description                             |
| ----------------- | --------- | -------- | ------------------ | --------------------------------------- |
| `clientId`        | `string`  | ✅ Yes   | —                  | Google OAuth client ID                  |
| `clientSecret`    | `string`  | ✅ Yes   | —                  | Google OAuth client secret              |
| `redirectUri`     | `string`  | ❌ No    | `http://localhost` | OAuth redirect URI                      |
| `refreshToken`    | `string`  | ❌ No    | —                  | OAuth refresh token                     |
| `accessToken`     | `string`  | ❌ No    | —                  | OAuth access token                      |
| `operation`       | `enum`    | ❌ No    | `listJobs`         | BigQuery job operation                  |
| `projectId`       | `string`  | ✅ Yes   | —                  | Google Cloud project ID                 |
| `jobId`           | `string`  | ✅ Yes\* | —                  | Job ID                                  |
| `acknowledgeRisk` | `boolean` | ✅ Yes\* | `false`            | Confirms trusted SQL in an inserted job |

\* `jobId` is required for `getJob`, `cancelJob`, and `deleteJob`. `acknowledgeRisk: true` is required when an `insertJob` configuration contains SQL.

### Operation-Specific Parameters

| Parameter       | Used By     | Notes                                                                  |
| --------------- | ----------- | ---------------------------------------------------------------------- |
| `jobListParams` | `listJobs`  | Optional user, pagination, state-filter, and projection settings       |
| `jobData`       | `insertJob` | Required object with a job `configuration` and optional `jobReference` |

### Security

SQL in `jobData.configuration.query.query` accepts one statement and requires `acknowledgeRisk: true`. Use GoogleSQL parameters for untrusted values.

### Outputs

The node returns the corresponding BigQuery API response. `deleteJob` returns `{ success: true }`.

<!-- /SECTION: configuration -->
