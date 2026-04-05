---
node_id: "google-chat-space"
title: "Google Chat - Space"
description: "Manage Google Chat spaces"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-chat
  - space
related_nodes:
  - google-chat-member
  - google-chat-message
---

<!-- SECTION: header -->

# Google Chat - Space

> **Category:** Integrations | **Type:** Action Node

Space operations: `list`, `get`, `create`, `delete`, `update`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter           | Type     | Required | Default | Description                                 |
| ------------------- | -------- | -------- | ------- | ------------------------------------------- |
| `clientId`          | `string` | ❌ No    | —       | OAuth client ID                             |
| `clientSecret`      | `string` | ❌ No    | —       | OAuth client secret                         |
| `refreshToken`      | `string` | ❌ No    | —       | OAuth refresh token                         |
| `accessToken`       | `string` | ❌ No    | —       | OAuth access token                          |
| `serviceAccountKey` | `any`    | ❌ No    | —       | Service account JSON key                    |
| `operation`         | `enum`   | ❌ No    | `list`  | `list`, `get`, `create`, `delete`, `update` |
| `pageSize`          | `number` | ❌ No    | —       | Page size (`list`)                          |
| `pageToken`         | `string` | ❌ No    | —       | Page token (`list`)                         |
| `name`              | `string` | ✅ Yes\* | —       | Space resource name (`get/delete/update`)   |
| `spaceJson`         | `string` | ❌ No    | —       | Raw JSON body for `create/update`           |

\* Required for `get`, `delete`, `update`.

### Outputs

| Operation | Output               |
| --------- | -------------------- |
| `list`    | API list response    |
| `get`     | Space object         |
| `create`  | Space object         |
| `delete`  | `{ ok: true }`       |
| `update`  | Updated space object |

<!-- /SECTION: configuration -->
