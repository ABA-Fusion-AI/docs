---
node_id: "google-big-query-project"
title: "Google BigQuery - Project"
description: "List BigQuery projects and get a project's service account"
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
  - project
related_nodes:
  - google-big-query-dataset
  - google-big-query-table
  - google-big-query-job
  - google-big-query-query
---

<!-- SECTION: header -->

# Google BigQuery - Project

> **Category:** Data | **Type:** Action Node

Project operations: `listProjects`, `getServiceAccount`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter      | Type     | Required | Default            | Description                            |
| -------------- | -------- | -------- | ------------------ | -------------------------------------- |
| `clientId`     | `string` | ✅ Yes   | —                  | Google OAuth client ID                 |
| `clientSecret` | `string` | ✅ Yes   | —                  | Google OAuth client secret             |
| `redirectUri`  | `string` | ❌ No    | `http://localhost` | OAuth redirect URI                     |
| `refreshToken` | `string` | ❌ No    | —                  | OAuth refresh token                    |
| `accessToken`  | `string` | ❌ No    | —                  | OAuth access token                     |
| `operation`    | `enum`   | ❌ No    | `listProjects`     | `listProjects` or `getServiceAccount`  |
| `projectId`    | `string` | ✅ Yes\* | —                  | Project ID used by `getServiceAccount` |

\* Required only for `getServiceAccount`.

### Outputs

| Operation           | Output                           |
| ------------------- | -------------------------------- |
| `listProjects`      | BigQuery project list response   |
| `getServiceAccount` | Project service-account response |

<!-- /SECTION: configuration -->
