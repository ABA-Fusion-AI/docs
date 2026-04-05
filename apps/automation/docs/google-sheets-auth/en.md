---
node_id: "google-sheets-auth"
title: "Google Sheets Auth"
description: "Validate Google OAuth credentials for Sheets and Drive APIs"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-sheets
  - auth
related_nodes:
  - google-sheets-spreadsheet
  - google-sheets-values
---

<!-- SECTION: header -->

# Google Sheets Auth

> **Category:** Integrations | **Type:** Action Node

Validates OAuth credentials and optionally returns token info for the provided access token.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

| Parameter      | Type     | Required | Default            | Description                                     |
| -------------- | -------- | -------- | ------------------ | ----------------------------------------------- |
| `clientId`     | `string` | ✅ Yes   | —                  | Google OAuth client ID                          |
| `clientSecret` | `string` | ✅ Yes   | —                  | Google OAuth client secret                      |
| `redirectUri`  | `string` | ❌ No    | `http://localhost` | OAuth redirect URI                              |
| `refreshToken` | `string` | ❌ No    | —                  | OAuth refresh token                             |
| `accessToken`  | `string` | ❌ No    | —                  | OAuth access token (used for token info lookup) |

### Output

```json
{
  "authorized": true,
  "tokenInfo": {}
}
```

`tokenInfo` is `null` when no `accessToken` is provided.

<!-- /SECTION: configuration -->
