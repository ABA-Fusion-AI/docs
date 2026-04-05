---
node_id: "google-chat-message-trigger"
title: "Google Chat - New Message Trigger"
description: "Poll a Google Chat space and emit new messages"
category: "triggers"
subcategory: "google"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - google-chat
  - polling
  - messages
related_nodes:
  - google-chat-message
---

<!-- SECTION: header -->

# Google Chat - New Message Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls a space for new messages and emits unseen items.

<!-- /SECTION: header -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter           | Type      | Required | Default | Description                             |
| ------------------- | --------- | -------- | ------- | --------------------------------------- |
| `clientId`          | `string`  | ❌ No    | —       | OAuth client ID                         |
| `clientSecret`      | `string`  | ❌ No    | —       | OAuth client secret                     |
| `refreshToken`      | `string`  | ❌ No    | —       | OAuth refresh token                     |
| `accessToken`       | `string`  | ❌ No    | —       | OAuth access token                      |
| `serviceAccountKey` | `any`     | ❌ No    | —       | Service account JSON key                |
| `parent`            | `string`  | ✅ Yes   | —       | Space parent (`spaces/{id}` or raw id)  |
| `intervalMs`        | `number`  | ❌ No    | `15000` | Poll interval in milliseconds           |
| `pageSize`          | `number`  | ❌ No    | —       | Max messages per poll                   |
| `emitOnStart`       | `boolean` | ❌ No    | `false` | If `false`, first poll only warms cache |

### Behavior

1. Lists space messages on each poll.
2. Keeps unseen IDs in memory.
3. Warmup poll records existing messages unless `emitOnStart=true`.
4. Emits full message objects for newly seen entries.

### Output

`output` emits an array of message objects.

<!-- /SECTION: configuration -->
