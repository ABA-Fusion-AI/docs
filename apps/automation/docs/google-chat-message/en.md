---
node_id: "google-chat-message"
title: "Google Chat - Message"
description: "Create, read, update, delete, and wait for replies in Google Chat messages"
category: "integrations"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - google-chat
  - message
related_nodes:
  - google-chat-space
  - google-chat-message-trigger
---

<!-- SECTION: header -->

# Google Chat - Message

> **Category:** Integrations | **Type:** Action Node

Message operations: `create`, `get`, `delete`, `update`, `sendAndWait`.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter           | Type     | Required | Default  | Description                                        |
| ------------------- | -------- | -------- | -------- | -------------------------------------------------- |
| `clientId`          | `string` | ❌ No    | —        | OAuth client ID                                    |
| `clientSecret`      | `string` | ❌ No    | —        | OAuth client secret                                |
| `refreshToken`      | `string` | ❌ No    | —        | OAuth refresh token                                |
| `accessToken`       | `string` | ❌ No    | —        | OAuth access token                                 |
| `serviceAccountKey` | `any`    | ❌ No    | —        | Service account JSON key                           |
| `operation`         | `enum`   | ❌ No    | `create` | `create`, `get`, `delete`, `update`, `sendAndWait` |
| `parent`            | `string` | ✅ Yes\* | —        | Space parent (`spaces/{id}` or raw id)             |
| `name`              | `string` | ✅ Yes\* | —        | Message resource name for `get/delete/update`      |
| `text`              | `string` | ❌ No    | —        | Message text                                       |
| `cardsJson`         | `string` | ❌ No    | —        | JSON string for cards body (`create/update`)       |
| `timeoutMs`         | `number` | ❌ No    | `30000`  | Timeout for `sendAndWait`                          |
| `pollIntervalMs`    | `number` | ❌ No    | `2000`   | Poll interval for `sendAndWait`                    |

\* Required for related operations.

### Outputs

| Operation     | Output                                             |
| ------------- | -------------------------------------------------- |
| `create`      | Created message object                             |
| `get`         | Message object                                     |
| `delete`      | `{ ok: true }`                                     |
| `update`      | Updated message object                             |
| `sendAndWait` | Reply messages array, or `{ timeout: true, sent }` |

<!-- /SECTION: configuration -->
