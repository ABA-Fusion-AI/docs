---
node_id: "google-sheets-trigger"
title: "Google Sheets - New Rows Trigger"
description: "Poll a sheet range and emit newly appended rows"
category: "triggers"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - google-sheets
  - polling
  - rows
related_nodes:
  - google-sheets-values
---

<!-- SECTION: header -->

# Google Sheets - New Rows Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls a range and emits new rows appended since the previous poll.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

| Parameter              | Type      | Required | Default            | Description                                       |
| ---------------------- | --------- | -------- | ------------------ | ------------------------------------------------- |
| `clientId`             | `string`  | ✅ Yes   | —                  | Google OAuth client ID                            |
| `clientSecret`         | `string`  | ✅ Yes   | —                  | Google OAuth client secret                        |
| `redirectUri`          | `string`  | ❌ No    | `http://localhost` | OAuth redirect URI                                |
| `refreshToken`         | `string`  | ✅ Yes   | —                  | OAuth refresh token                               |
| `accessToken`          | `string`  | ❌ No    | —                  | OAuth access token                                |
| `spreadsheetId`        | `string`  | ✅ Yes   | —                  | Target spreadsheet ID                             |
| `range`                | `string`  | ❌ No    | `Sheet1!A:Z`       | Range to monitor                                  |
| `majorDimension`       | `enum`    | ❌ No    | `ROWS`             | `ROWS` or `COLUMNS`                               |
| `headerRows`           | `number`  | ❌ No    | `1`                | Number of top rows excluded from data             |
| `intervalMs`           | `number`  | ❌ No    | `15000`            | Poll interval in milliseconds                     |
| `emitOnStart`          | `boolean` | ❌ No    | `false`            | If `false`, first poll warms baseline only        |
| `valueRenderOption`    | `enum`    | ❌ No    | —                  | `FORMATTED_VALUE`, `UNFORMATTED_VALUE`, `FORMULA` |
| `dateTimeRenderOption` | `enum`    | ❌ No    | —                  | `SERIAL_NUMBER` or `FORMATTED_STRING`             |

### Output

```json
{
  "rows": [],
  "spreadsheetId": "xxx",
  "range": "Sheet1!A:Z",
  "majorDimension": "ROWS",
  "totalRows": 10,
  "timestamp": "2026-03-11T12:00:00.000Z"
}
```

### Notes

- State is persisted per node using `google-sheets-trigger-{nodeId}` cache key.
- Trigger compares current row count vs previous row count to emit only appended rows.

<!-- /SECTION: configuration -->
