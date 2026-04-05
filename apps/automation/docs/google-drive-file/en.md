---
node_id: "google-drive-file"
title: "Google Drive - File"
description: "Upload, download, update, and delete files in Google Drive"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-drive
  - file
  - storage
related_nodes:
  - google-drive-folder
  - google-drive-search
  - google-drive-trigger
---

<!-- SECTION: header -->

# Google Drive - File

> **Category:** Integrations | **Type:** Action Node

Run file operations in Drive: `upload`, `download`, `delete`, and `update`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter       | Type       | Required | Default                      | Description                                      |
| --------------- | ---------- | -------- | ---------------------------- | ------------------------------------------------ |
| `clientId`      | `string`   | ✅ Yes   | —                            | Google OAuth client ID                           |
| `clientSecret`  | `string`   | ✅ Yes   | —                            | Google OAuth client secret                       |
| `redirectUri`   | `string`   | ❌ No    | `http://localhost`           | OAuth redirect URI                               |
| `refreshToken`  | `string`   | ✅ Yes   | —                            | OAuth refresh token                              |
| `operation`     | `enum`     | ❌ No    | `upload`                     | One of: `upload`, `download`, `delete`, `update` |
| `name`          | `string`   | ❌ No    | —                            | File name (`upload`, `update`)                   |
| `mimeType`      | `string`   | ❌ No    | `application/octet-stream`\* | MIME type                                        |
| `contentBase64` | `string`   | ✅ Yes\* | —                            | Base64 content for `upload`/`update`             |
| `parents`       | `string[]` | ❌ No    | —                            | Parent folder IDs (`upload`)                     |
| `fileId`        | `string`   | ✅ Yes\* | —                            | Target file ID (`download`, `delete`, `update`)  |

\* Required only in listed operations.

### Outputs

| Operation  | Output                     |
| ---------- | -------------------------- |
| `upload`   | `{ file }`                 |
| `download` | `{ base64, size }`         |
| `delete`   | Google API delete response |
| `update`   | `{ file }`                 |

<!-- /SECTION: configuration -->
