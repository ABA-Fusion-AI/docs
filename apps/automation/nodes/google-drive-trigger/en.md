---
node_id: "google-drive-trigger"
title: "Google Drive - New File Trigger"
description: "Poll Google Drive and emit newly seen files"
category: "triggers"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - google-drive
  - polling
  - files
related_nodes:
  - google-drive-file
  - google-drive-search
---

<!-- SECTION: header -->

# Google Drive - New File Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls Google Drive and emits files not seen before by this node instance.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter      | Type      | Required | Default            | Description                             |
| -------------- | --------- | -------- | ------------------ | --------------------------------------- |
| `clientId`     | `string`  | ✅ Yes   | —                  | Google OAuth client ID                  |
| `clientSecret` | `string`  | ✅ Yes   | —                  | Google OAuth client secret              |
| `redirectUri`  | `string`  | ❌ No    | `http://localhost` | OAuth redirect URI                      |
| `refreshToken` | `string`  | ✅ Yes   | —                  | OAuth refresh token                     |
| `folderId`     | `string`  | ❌ No    | —                  | Restrict poll to files in a folder      |
| `driveId`      | `string`  | ❌ No    | —                  | Shared drive ID                         |
| `q`            | `string`  | ❌ No    | —                  | Additional Drive query filter           |
| `intervalMs`   | `number`  | ❌ No    | `15000`            | Poll interval in milliseconds           |
| `emitOnStart`  | `boolean` | ❌ No    | `false`            | If `false`, first poll warms cache only |
| `pageSize`     | `number`  | ❌ No    | `100`              | Number of files per page                |

### Trigger Behavior

1. Builds query from `folderId` + `q`.
2. Lists matching files with pagination.
3. Compares IDs with cached seen IDs.
4. Emits only new files (unless warmup run).
5. Persists seen IDs to workflow context key `google-drive-cache-{nodeId}`.

### Output

`output` emits an array of file objects with fields:

- `id`, `name`, `mimeType`, `parents`, `createdTime`, `modifiedTime`

<!-- /SECTION: configuration -->
