---
node_id: "google-big-query-table"
title: "Google BigQuery - Table"
description: "Manage BigQuery tables and table data"
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
  - table
related_nodes:
  - google-big-query-project
  - google-big-query-dataset
  - google-big-query-job
  - google-big-query-query
---

<!-- SECTION: header -->

# Google BigQuery - Table

> **Category:** Data | **Type:** Action Node

Table operations: `listTables`, `getTable`, `createTable`, `updateTable`, `deleteTable`, `insertTableData`, `getTableData`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Core Parameters

| Parameter      | Type     | Required | Default            | Description                    |
| -------------- | -------- | -------- | ------------------ | ------------------------------ |
| `clientId`     | `string` | ✅ Yes   | —                  | Google OAuth client ID         |
| `clientSecret` | `string` | ✅ Yes   | —                  | Google OAuth client secret     |
| `redirectUri`  | `string` | ❌ No    | `http://localhost` | OAuth redirect URI             |
| `refreshToken` | `string` | ❌ No    | —                  | OAuth refresh token            |
| `accessToken`  | `string` | ❌ No    | —                  | OAuth access token             |
| `operation`    | `enum`   | ❌ No    | `listTables`       | Table or table-data operation  |
| `projectId`    | `string` | ✅ Yes   | —                  | Google Cloud project ID        |
| `datasetId`    | `string` | ✅ Yes   | —                  | BigQuery dataset ID            |
| `tableId`      | `string` | ✅ Yes\* | —                  | Table ID; optional on creation |

\* Required for `getTable`, `updateTable`, `deleteTable`, `insertTableData`, and `getTableData`.

### Operation-Specific Parameters

| Parameter         | Used By           | Notes                                                           |
| ----------------- | ----------------- | --------------------------------------------------------------- |
| `tableListParams` | `listTables`      | Optional `maxResults` and `pageToken`                           |
| `tableData`       | `createTable`     | Optional metadata, schema, partitioning, clustering, and labels |
| `tableUpdateData` | `updateTable`     | Optional `friendlyName`, `description`, `schema`, and `labels`  |
| `insertRows`      | `insertTableData` | Required array of row values                                    |
| `tableDataParams` | `getTableData`    | Optional `maxResults`, `startIndex`, and `pageToken`            |

### Outputs

The node returns the corresponding BigQuery API response. `deleteTable` returns `{ success: true }`.

<!-- /SECTION: configuration -->
