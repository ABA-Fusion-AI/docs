---
node_id: "teams-channel"
title: "Microsoft Teams - Channel"
description: "Channel and channel-message operations in Microsoft Teams"
category: "integrations"
subcategory: "microsoft"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - integration
  - teams
  - channel
  - microsoft-graph
related_nodes:
  - teams-new-channel-trigger
  - teams-new-team-member-trigger
---

# Microsoft Teams - Channel

> **Category:** Integrations | **Type:** Action Node

Operations: `get`, `list`, `create`, `delete`, `update`, `messageCreate`, `messageList`.

## Configuration

Auth: `accessToken` or `tenantId` + `clientId` + `clientSecret`.

Key params:

- `teamId` for all channel operations
- `channelId` for get/delete/update/message operations
- `displayName`, `description` for create/update
- `contentType`, `content` for `messageCreate`
- `top` for `list`/`messageList`

Outputs:

- Graph payload for non-delete operations
- `{ ok: true }` for delete
