---
node_id: "google-sheets-values"
title: "Google Sheets - Values"
description: "Read and write cell values in Google Sheets"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-sheets
  - values
related_nodes:
  - google-sheets-spreadsheet
  - google-sheets-trigger
---

<!-- SECTION: header -->

# Google Sheets - Values

> **Category:** Integrations | **Type:** Action Node

Value operations: `append`, `appendOrUpdate`, `update`, `get`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Core Parameters

| Parameter       | Type     | Required | Default            | Description                                 |
| --------------- | -------- | -------- | ------------------ | ------------------------------------------- |
| `clientId`      | `string` | ✅ Yes   | —                  | Google OAuth client ID                      |
| `clientSecret`  | `string` | ✅ Yes   | —                  | Google OAuth client secret                  |
| `redirectUri`   | `string` | ❌ No    | `http://localhost` | OAuth redirect URI                          |
| `refreshToken`  | `string` | ❌ No    | —                  | OAuth refresh token                         |
| `accessToken`   | `string` | ❌ No    | —                  | OAuth access token                          |
| `operation`     | `enum`   | ❌ No    | `append`           | `append`, `appendOrUpdate`, `update`, `get` |
| `spreadsheetId` | `string` | ✅ Yes   | —                  | Target spreadsheet ID                       |
| `range`         | `string` | ✅ Yes   | —                  | A1 range                                    |

### Write Parameters (`append`, `appendOrUpdate`, `update`)

| Parameter          | Type   | Required | Default          | Description                                            |
| ------------------ | ------ | -------- | ---------------- | ------------------------------------------------------ |
| `values`           | `any`  | ✅ Yes   | —                | Input values (array, object rows, JSON string, scalar) |
| `valueInputOption` | `enum` | ❌ No    | `USER_ENTERED`\* | Used by `appendOrUpdate` and `update`                  |

\* `append` always uses `USER_ENTERED`.

### Read Parameters (`get`)

| Parameter        | Type     | Required | Default | Description                                   |
| ---------------- | -------- | -------- | ------- | --------------------------------------------- |
| `majorDimension` | `enum`   | ❌ No    | —       | `ROWS` or `COLUMNS`                           |
| `rowId`          | `any`    | ❌ No    | —       | If provided, returns matching row only        |
| `rowIdColumn`    | `any`    | ❌ No    | —       | Column index (1-based number) or header name  |
| `headerRowIndex` | `number` | ❌ No    | `1`     | Header row index when `rowIdColumn` is string |

### Outputs

| Operation            | Output                          |
| -------------------- | ------------------------------- |
| `append`             | `{ updatedRange, updatedRows }` |
| `appendOrUpdate`     | `{ updatedRange, updatedRows }` |
| `update`             | `{ updatedRange, updatedRows }` |
| `get` (no `rowId`)   | `{ values }`                    |
| `get` (with `rowId`) | `{ row, rowIndex }`             |

<!-- /SECTION: configuration -->
