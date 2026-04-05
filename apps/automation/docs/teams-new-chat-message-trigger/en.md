---
node_id: "teams-new-chat-message-trigger"
title: "Microsoft Teams - On New Chat Message Trigger"
description: "Polls a chat and emits newly detected messages"
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - teams
  - chat-message
  - polling
related_nodes:
  - teams-chat
---

# Microsoft Teams - On New Chat Message Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls `/chats/{chatId}/messages` and emits new messages after warmup.

Params:

- auth fields
- `chatId` (required)
- `pollInterval` (seconds, default `10`)

Outputs:

- `output`: chat message object
- `error`: `{ error, reason }`
