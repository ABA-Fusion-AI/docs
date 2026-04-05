---
node_id: "google-drive-folder"
title: "Google Drive - Folder"
description: "Create, read, and delete folders in Google Drive"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-drive
  - folder
related_nodes:
  - google-drive-file
  - google-drive-search
---

<!-- SECTION: header -->

# Google Drive - Folder

> **Category:** Integrations | **Type:** Action Node

Folder operations in Drive: `create`, `get`, and `delete`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter      | Type       | Required | Default            | Description                       |
| -------------- | ---------- | -------- | ------------------ | --------------------------------- |
| `clientId`     | `string`   | ✅ Yes   | —                  | Google OAuth client ID            |
| `clientSecret` | `string`   | ✅ Yes   | —                  | Google OAuth client secret        |
| `redirectUri`  | `string`   | ❌ No    | `http://localhost` | OAuth redirect URI                |
| `refreshToken` | `string`   | ✅ Yes   | —                  | OAuth refresh token               |
| `operation`    | `enum`     | ❌ No    | `create`           | One of: `create`, `get`, `delete` |
| `name`         | `string`   | ❌ No    | —                  | Folder name (`create`)            |
| `parents`      | `string[]` | ❌ No    | —                  | Parent folder IDs (`create`)      |
| `folderId`     | `string`   | ✅ Yes\* | —                  | Folder ID (`get`, `delete`)       |

\* Required only for `get` and `delete`.

### Outputs

| Operation | Output                     |
| --------- | -------------------------- |
| `create`  | `{ folder }`               |
| `get`     | `{ folder }`               |
| `delete`  | Google API delete response |

<!-- /SECTION: configuration -->
