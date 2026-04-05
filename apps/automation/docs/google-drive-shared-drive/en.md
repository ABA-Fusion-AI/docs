---
node_id: "google-drive-shared-drive"
title: "Google Drive - Shared Drive"
description: "Manage Google Shared Drives and list files inside them"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-drive
  - shared-drive
related_nodes:
  - google-drive-search
  - google-drive-file
---

<!-- SECTION: header -->

# Google Drive - Shared Drive

> **Category:** Integrations | **Type:** Action Node

Run shared-drive operations: `create`, `get`, `list`, `update`, `delete`, `listFiles`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter                   | Type      | Required | Default            | Description                                                      |
| --------------------------- | --------- | -------- | ------------------ | ---------------------------------------------------------------- |
| `clientId`                  | `string`  | ✅ Yes   | —                  | Google OAuth client ID                                           |
| `clientSecret`              | `string`  | ✅ Yes   | —                  | Google OAuth client secret                                       |
| `redirectUri`               | `string`  | ❌ No    | `http://localhost` | OAuth redirect URI                                               |
| `refreshToken`              | `string`  | ✅ Yes   | —                  | OAuth refresh token                                              |
| `operation`                 | `enum`    | ❌ No    | `list`             | One of: `create`, `get`, `list`, `update`, `delete`, `listFiles` |
| `name`                      | `string`  | ✅ Yes\* | —                  | Shared drive name (`create`, optional for `update`)              |
| `sharedDriveId`             | `string`  | ✅ Yes\* | —                  | Required for `get`, `update`, `delete`, `listFiles`              |
| `pageSize`                  | `number`  | ❌ No    | `50`               | Page size for `list`                                             |
| `q`                         | `string`  | ❌ No    | —                  | Query for `listFiles`                                            |
| `filePageSize`              | `number`  | ❌ No    | `100`              | Page size for `listFiles`                                        |
| `fields`                    | `string`  | ❌ No    | —                  | Response fields for `listFiles`                                  |
| `includeItemsFromAllDrives` | `boolean` | ❌ No    | `true`             | `listFiles` option                                               |
| `supportsAllDrives`         | `boolean` | ❌ No    | `true`             | `listFiles` option                                               |

\* Required only for listed operations.

### Outputs

| Operation   | Output                      |
| ----------- | --------------------------- |
| `create`    | `{ drive }`                 |
| `get`       | `{ drive }`                 |
| `list`      | `{ drives, nextPageToken }` |
| `listFiles` | `{ files, nextPageToken }`  |
| `update`    | `{ drive }`                 |
| `delete`    | Google API delete response  |

<!-- /SECTION: configuration -->
