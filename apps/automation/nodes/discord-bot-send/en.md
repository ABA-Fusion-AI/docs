---
node_id: "discord-bot-send"
title: "Discord Bot Send Message"
description: "Send a Discord channel message through the Discord Bot API"
category: "integrations"
subcategory: "messaging"
language: "en"
tags: [discord, bot, messaging]
related_nodes: [discord-action, discord-webhook-send]
---

<!-- SECTION: header -->
# Discord Bot Send Message

> **Category:** Integrations | **Type:** Action Node

Sends a message directly to a Discord channel using a bot token and channel ID.
<!-- /SECTION: header -->

<!-- SECTION: configuration -->
## Configuration

| Parameter | Required | Description |
|---|---:|---|
| `botToken` | Yes | Discord bot token. Supports expressions. |
| `channelId` | Yes | Target Discord channel ID. Supports expressions. |
| `content` | No | Message content. If omitted, the incoming data is serialized and sent. |
<!-- /SECTION: configuration -->

<!-- SECTION: examples -->
## Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send a deployment notification
```
<!-- /SECTION: examples -->
