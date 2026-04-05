---
node_id: "google-drive-search"
title: "Google Drive - Search"
description: "Search Google Drive files using query expressions"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-drive
  - search
related_nodes:
  - google-drive-file
  - google-drive-folder
---

<!-- SECTION: header -->

# Google Drive - Search

> **Category:** Integrations | **Type:** Action Node

Search Drive files with `q` and advanced list options.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter                   | Type      | Required | Default                    | Description                                        |
| --------------------------- | --------- | -------- | -------------------------- | -------------------------------------------------- |
| `clientId`                  | `string`  | ✅ Yes   | —                          | Google OAuth client ID                             |
| `clientSecret`              | `string`  | ✅ Yes   | —                          | Google OAuth client secret                         |
| `redirectUri`               | `string`  | ❌ No    | `http://localhost`         | OAuth redirect URI                                 |
| `refreshToken`              | `string`  | ✅ Yes   | —                          | OAuth refresh token                                |
| `q`                         | `string`  | ❌ No    | —                          | Drive query string                                 |
| `pageSize`                  | `number`  | ❌ No    | `100`                      | Max files per request                              |
| `fields`                    | `string`  | ❌ No    | `files(...),nextPageToken` | Response fields                                    |
| `includeItemsFromAllDrives` | `boolean` | ❌ No    | `true`                     | Include shared drive items                         |
| `supportsAllDrives`         | `boolean` | ❌ No    | `true`                     | Enable all-drives support                          |
| `corpora`                   | `string`  | ❌ No    | —                          | Search corpus (`user`, `drive`, `allDrives`, etc.) |
| `driveId`                   | `string`  | ❌ No    | —                          | Shared drive ID when targeting a specific drive    |

### Notes

- If `q` contains `id = 'FILE_ID'`, the node calls `files.get` directly and returns a one-item `files` array.
- Returned shape is always `{ files, nextPageToken }`.

<!-- /SECTION: configuration -->
