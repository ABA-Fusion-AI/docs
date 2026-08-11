---
node_id: "google-big-query-dataset"
title: "Google BigQuery - Dataset"
description: "List, create, inspect, update, delete, and restore BigQuery datasets"
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
  - dataset
related_nodes:
  - google-big-query-project
  - google-big-query-table
  - google-big-query-job
  - google-big-query-query
---

<!-- SECTION: header -->

# Google BigQuery - Dataset

> **Category:** Data | **Type:** Action Node

Dataset operations: `listDatasets`, `getDataset`, `createDataset`, `updateDataset`, `deleteDataset`, `undeleteDataset`.

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
| `operation`    | `enum`   | ❌ No    | `listDatasets`     | Dataset operation              |
| `projectId`    | `string` | ✅ Yes   | —                  | Google Cloud project ID        |
| `datasetId`    | `string` | ✅ Yes\* | —                  | Dataset ID; optional on create |

\* Required for `getDataset`, `updateDataset`, `deleteDataset`, and `undeleteDataset`.

### Operation-Specific Parameters

| Parameter           | Used By         | Notes                                                                               |
| ------------------- | --------------- | ----------------------------------------------------------------------------------- |
| `datasetListParams` | `listDatasets`  | Optional `maxResults`, `pageToken`, `all`, and `filter`                             |
| `datasetData`       | `createDataset` | Optional `friendlyName`, `description`, `location`, expiration values, and `labels` |
| `datasetUpdateData` | `updateDataset` | Optional `friendlyName`, `description`, default expiration values, and `labels`     |

### Outputs

The node returns the corresponding BigQuery API response. `deleteDataset` returns `{ success: true }`.

<!-- /SECTION: configuration -->
