---
node_id: "google-sheets-spreadsheet"
title: "Google Sheets - Spreadsheet"
description: "Spreadsheet-level operations for Google Sheets"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-sheets
  - spreadsheet
related_nodes:
  - google-sheets-values
  - google-sheets-trigger
---

<!-- SECTION: header -->

# Google Sheets - Spreadsheet

> **Category:** Integrations | **Type:** Action Node

Run spreadsheet operations: `createSpreadsheet`, `deleteSpreadsheet`, `addSheet`, `deleteSheet`, `clearRange`, `deleteDimension`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Core Parameters

| Parameter      | Type     | Required | Default            | Description                |
| -------------- | -------- | -------- | ------------------ | -------------------------- |
| `clientId`     | `string` | ✅ Yes   | —                  | Google OAuth client ID     |
| `clientSecret` | `string` | ✅ Yes   | —                  | Google OAuth client secret |
| `redirectUri`  | `string` | ❌ No    | `http://localhost` | OAuth redirect URI         |
| `refreshToken` | `string` | ❌ No    | —                  | OAuth refresh token        |
| `accessToken`  | `string` | ❌ No    | —                  | OAuth access token         |
| `operation`    | `enum`   | ❌ No    | `addSheet`         | Spreadsheet operation      |

### Operation-Specific Parameters

| Parameter                    | Used By                                                                         | Notes                                        |
| ---------------------------- | ------------------------------------------------------------------------------- | -------------------------------------------- |
| `spreadsheetId`              | `deleteSpreadsheet`, `addSheet`, `deleteSheet`, `clearRange`, `deleteDimension` | Required in these operations                 |
| `spreadsheetTitle` / `title` | `createSpreadsheet`                                                             | One of them is required                      |
| `sheetId`                    | `deleteSheet`, `deleteDimension`                                                | Optional if `sheetTitle`/`title` is provided |
| `sheetTitle` / `title`       | `addSheet`, `deleteSheet`, `deleteDimension`                                    | For `addSheet`, one is required              |
| `range`                      | `clearRange`                                                                    | Required                                     |
| `rowCount`, `columnCount`    | `addSheet`                                                                      | Defaults: `1000`, `26`                       |
| `dimension`                  | `deleteDimension`                                                               | `ROWS` or `COLUMNS`, default `ROWS`          |
| `startIndex`, `endIndex`     | `deleteDimension`                                                               | Both required                                |

### Outputs

| Operation           | Output                     |
| ------------------- | -------------------------- |
| `createSpreadsheet` | `{ spreadsheetId, url }`   |
| `deleteSpreadsheet` | `{ status }`               |
| `addSheet`          | `{ sheet }`                |
| `deleteSheet`       | `{ result }`               |
| `clearRange`        | `{ clearedRange, status }` |
| `deleteDimension`   | `{ replies }`              |

<!-- /SECTION: configuration -->
