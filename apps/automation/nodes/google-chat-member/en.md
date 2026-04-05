---
node_id: "google-chat-member"
title: "Google Chat - Member"
description: "Get, list, add, and remove members in a Google Chat space"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-chat
  - membership
related_nodes:
  - google-chat-space
---

<!-- SECTION: header -->

# Google Chat - Member

> **Category:** Integrations | **Type:** Action Node

Member operations: `get`, `list`, `add`, `remove`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter           | Type     | Required | Default | Description                                    |
| ------------------- | -------- | -------- | ------- | ---------------------------------------------- |
| `clientId`          | `string` | ❌ No    | —       | OAuth client ID                                |
| `clientSecret`      | `string` | ❌ No    | —       | OAuth client secret                            |
| `refreshToken`      | `string` | ❌ No    | —       | OAuth refresh token                            |
| `accessToken`       | `string` | ❌ No    | —       | OAuth access token                             |
| `serviceAccountKey` | `any`    | ❌ No    | —       | Service account JSON key                       |
| `operation`         | `enum`   | ❌ No    | `list`  | `get`, `list`, `add`, `remove`                 |
| `space`             | `string` | ✅ Yes   | —       | Space ID or resource (`spaces/{id}` supported) |
| `memberId`          | `string` | ✅ Yes\* | —       | Member ID/resource (`get`, `remove`)           |
| `memberJson`        | `string` | ❌ No    | —       | Raw request body JSON (`add`)                  |
| `pageSize`          | `number` | ❌ No    | —       | List page size                                 |
| `pageToken`         | `string` | ❌ No    | —       | List page token                                |

\* Required for `get` and `remove`.

### Outputs

| Operation | Output                    |
| --------- | ------------------------- |
| `get`     | Member object             |
| `list`    | API list response         |
| `add`     | Created membership object |
| `remove`  | `{ ok: true }`            |

<!-- /SECTION: configuration -->
