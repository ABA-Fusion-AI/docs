---
node_id: "discord-webhook-send"
title: "Discord Webhook Send"
description: "Send a Discord message through an incoming webhook URL"
category: "integrations"
subcategory: "messaging"
language: "en"
tags: [discord, webhook, messaging]
related_nodes: [discord-action, discord-bot-send]
---

<!-- SECTION: header -->
# Discord Webhook Send

> **Category:** Integrations | **Type:** Action Node

Sends a message to Discord through an incoming webhook URL. It does not require a bot connection.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Required | Description |
|---|---:|---|
| `webhookUrl` | Yes | Discord incoming webhook URL. Supports expressions. |
| `content` | No | Message content; incoming data is used when omitted. |
| `username` | No | Display name for the webhook message. |
| `avatarPreset` | No | Built-in avatar choice, or `CUSTOM`. |
| `customAvatarUrl` | Conditional | Avatar URL when `avatarPreset` is `CUSTOM`. |
| `embeds` | No | Array of Discord embed objects. |
<!-- /SECTION: configuration -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send an incoming-webhook notification
```
<!-- /SECTION: examples -->
