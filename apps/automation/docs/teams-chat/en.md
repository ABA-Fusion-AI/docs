---
node_id: "teams-chat"
title: "Microsoft Teams - Chat"
description: "Chat message operations in Microsoft Teams via Graph API"
category: "integrations"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - teams
  - chat
  - microsoft-graph
related_nodes:
  - teams-new-chat-trigger
  - teams-new-chat-message-trigger
---

# Microsoft Teams - Chat

> **Category:** Integrations | **Type:** Action Node

Operations: `list`, `get`, `create`, `delete`, `sendAndWait`.

## Configuration

Auth: provide `accessToken` or `tenantId` + `clientId` + `clientSecret`.

Key params:

- `chatId` for all ops except some list contexts
- `messageId` for `get`/`delete`
- `contentType` (`text`/`html`) and `content` for `create`/`sendAndWait`
- `top` for `list`
- `waitForSeconds`, `pollIntervalSeconds` for `sendAndWait`

Outputs:

- Standard Graph payload for read/create/list
- `{ ok: true }` for delete
- `sendAndWait`: `{ posted, replies }`
