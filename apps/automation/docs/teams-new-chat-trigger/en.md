---
node_id: "teams-new-chat-trigger"
title: "Microsoft Teams - On New Chat Trigger"
description: "Polls chats and emits newly detected chats"
category: "triggers"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - trigger
  - teams
  - chat
  - polling
related_nodes:
  - teams-chat
---

# Microsoft Teams - On New Chat Trigger

> **Category:** Triggers | **Type:** Trigger Node

Polls `/me/chats` and emits chats not seen before.

Params:

- auth fields (`accessToken` or tenant/client credentials)
- `pollInterval` (seconds, default `30`)

Outputs:

- `output`: chat object
- `error`: `{ error, reason }`
