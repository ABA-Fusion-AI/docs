---
node_id: "discord-action"
title: "Discord Action"
description: "Send messages, embeds, reactions, replies, and direct messages on Discord using a bot token"
category: "integrations"
subcategory: "messaging"
version: "1.0.0"
language: "en"
last_updated: "2026-03-11"
author: "Fusion Team"
tags:
  - discord
  - messaging
  - bot
  - embed
  - dm
  - action
related_nodes:
  - discord-trigger
  - slack-action
  - webhook-trigger
---

<!-- SECTION: header -->
# Discord Action

> **Category:** Integrations | **Type:** Action Node

Perform messaging actions on Discord — send plain messages or rich embeds, reply to existing messages, edit or delete content, add emoji reactions, and send direct messages to users.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Discord Action** node connects to Discord using a bot token via the `discord.js` client. It establishes a persistent WebSocket connection on `setup()` and executes one of 7 messaging operations per trigger.

All operations interact with the Discord REST API through the discord.js library. The client must be connected before operations can run — it validates the token length (minimum 50 characters) and confirms the connection is ready before each execution.

### Key Features

- **7 Operations:** Full coverage of messaging — send, embed, reply, edit, delete, react, and DM
- **Rich Embeds:** 10 named color options, optional title, description, and timestamp
- **20 Emoji Reactions:** Curated list of common reaction emojis available as an enum
- **Direct Messages:** Send DMs to any user by their Discord user ID
- **Expression Support:** `channelId`, `messageId`, `content`, and `userId` accept dynamic expressions

### Use Cases

- Send a deployment notification to a `#deployments` Discord channel after a CI workflow completes
- Post a rich embed summarizing daily stats to a `#reports` channel on a schedule
- Reply to a specific message with the outcome of an automated task
- React to a message with ✅ or ❌ based on a test result
- Send a DM to a specific user when a critical threshold is breached

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | `string` | ✅ Yes | Discord bot token — at least 50 characters. Generate at **Discord Developer Portal → Applications → Bot → Token** |

### Operation Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `sendMessage` | Operation to execute (see below) |
| `channelId` | `string` | Conditional | — | Discord channel ID — required for all non-DM operations |
| `content` | `string` | Conditional | — | Message text — required for `sendMessage`, `replyToMessage`, `editMessage`, `sendDM` |
| `messageId` | `string` | Conditional | — | Target message ID — required for `replyToMessage`, `editMessage`, `deleteMessage`, `addReaction` |
| `embed` | `object` | Conditional | — | Embed configuration — required for `sendEmbed` (see structure below) |
| `emoji` | `enum` | Conditional | — | Reaction emoji — required for `addReaction` |
| `userId` | `string` | Conditional | — | Target user ID — required for `sendDM` |

### `embed` Structure

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `title` | `string` | ❌ No | — | Embed title |
| `description` | `string` | ❌ No | — | Embed body text |
| `color` | `enum` | ❌ No | `blue` | Embed color: `blue`, `green`, `red`, `yellow`, `purple`, `orange`, `pink`, `gray`, `black`, `white` |
| `timestamp` | `string` | ❌ No | — | ISO 8601 timestamp displayed in the embed footer |

### Available Emojis (`addReaction`)

`👍` `👎` `❤️` `😂` `😢` `😡` `🎉` `🔥` `✅` `❌` `⭐` `💯` `👀` `🚀` `⚡` `💪` `🙏` `👏` `🤔` `😍`

### Available Operations

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `sendMessage` | `channelId`, `content` | Send a plain text message to a channel |
| `sendEmbed` | `channelId`, `embed` | Send a rich embed message to a channel |
| `replyToMessage` | `channelId`, `messageId`, `content` | Reply to an existing message |
| `editMessage` | `channelId`, `messageId`, `content` | Edit an existing bot message |
| `deleteMessage` | `channelId`, `messageId` | Delete a message |
| `addReaction` | `channelId`, `messageId`, `emoji` | Add an emoji reaction to a message |
| `sendDM` | `userId`, `content` | Send a direct message to a user |

> **Note:** The bot must have the required Discord permissions for each operation (e.g. `SEND_MESSAGES`, `MANAGE_MESSAGES` for delete, `ADD_REACTIONS`).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream data — `channelId`, `messageId`, `content`, and `userId` can be resolved via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the operation |
| `error` | `Error` | Emitted if the operation fails or the client is not ready |

### Output Schemas

**`sendMessage` / `sendEmbed` / `replyToMessage`**
```json
{
  "success": true,
  "operation": "sendMessage",
  "message_id": "1234567890123456789",
  "message": "Message sent successfully",
  "timestamp": "2026-03-11T10:00:00.000Z"
}
```

**`editMessage`**
```json
{
  "success": true,
  "operation": "editMessage",
  "message_id": "1234567890123456789",
  "message": "Message edited successfully",
  "timestamp": "2026-03-11T10:00:00.000Z"
}
```

**`deleteMessage`**
```json
{
  "success": true,
  "operation": "deleteMessage",
  "message_id": "1234567890123456789",
  "message": "Message deleted successfully",
  "timestamp": "2026-03-11T10:00:00.000Z"
}
```

**`addReaction`**
```json
{
  "success": true,
  "operation": "addReaction",
  "message_id": "1234567890123456789",
  "emoji": "✅",
  "message": "Reaction added successfully",
  "timestamp": "2026-03-11T10:00:00.000Z"
}
```

**`sendDM`**
```json
{
  "success": true,
  "operation": "sendDM",
  "message_id": "9876543210987654321",
  "message": "DM sent successfully",
  "timestamp": "2026-03-11T10:00:00.000Z"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Send a Plain Message

**Configuration:**
```json
{
  "token": "your-discord-bot-token",
  "operation": "sendMessage",
  "channelId": "1234567890123456789",
  "content": "✅ Deployment to production completed successfully!"
}
```

---

### Example: Send a Rich Embed

Post a styled summary report with a color-coded embed.

**Configuration:**
```json
{
  "token": "your-discord-bot-token",
  "operation": "sendEmbed",
  "channelId": "1234567890123456789",
  "embed": {
    "title": "📊 Daily Report — {{input.date}}",
    "description": "Total orders: **{{input.orders}}**\nRevenue: **€{{input.revenue}}**\nErrors: **{{input.errors}}**",
    "color": "green",
    "timestamp": "{{input.generatedAt}}"
  }
}
```

---

### Example: Add a Reaction Based on Build Result

React with ✅ on success or ❌ on failure.

**Configuration (success branch):**
```json
{
  "token": "your-discord-bot-token",
  "operation": "addReaction",
  "channelId": "{{input.channelId}}",
  "messageId": "{{input.messageId}}",
  "emoji": "✅"
}
```

---

### Example: Send a DM Alert

Send a personal alert to an on-call engineer.

**Configuration:**
```json
{
  "token": "your-discord-bot-token",
  "operation": "sendDM",
  "userId": "{{input.oncallUserId}}",
  "content": "🚨 Alert: CPU usage exceeded 95% on server {{input.serverName}}. Please investigate."
}
```

---

### Example: Reply to a Message

Reply inline to a thread or conversation.

**Configuration:**
```json
{
  "token": "your-discord-bot-token",
  "operation": "replyToMessage",
  "channelId": "{{input.channelId}}",
  "messageId": "{{input.messageId}}",
  "content": "✅ Your request has been processed. Reference: {{input.ticketId}}"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: CI Build → Discord Notification

After a GitHub Actions workflow completes, post the result to a Discord channel.

```json
{
  "nodes": [
    {
      "id": "github-webhook",
      "type": "webhook-trigger"
    },
    {
      "id": "check-result",
      "type": "function",
      "config": {
        "code": "return { status: input.conclusion, runId: input.id, repo: input.repository.full_name, branch: input.head_branch };"
      }
    },
    {
      "id": "notify-discord",
      "type": "discord-action",
      "config": {
        "token": "your-bot-token",
        "operation": "sendEmbed",
        "channelId": "1234567890123456789",
        "embed": {
          "title": "{{input.status === 'success' ? '✅' : '❌'}} Build {{input.status}}",
          "description": "**Repo:** {{input.repo}}\n**Branch:** {{input.branch}}\n**Run:** #{{input.runId}}",
          "color": "{{input.status === 'success' ? 'green' : 'red'}}"
        }
      }
    }
  ]
}
```

### Common Patterns

- **Deploy Notify:** CI webhook → Discord `sendEmbed` (green/red based on result)
- **Alert Fan-out:** Monitoring event → Discord `sendDM` to on-call + `sendMessage` to channel
- **Moderation Bot:** Discord trigger (message) → Function (check keywords) → Discord `deleteMessage` + `addReaction` (❌)
- **Scheduled Report:** Cron → DB query → Discord `sendEmbed` with formatted stats

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Discord client not ready`

**Cause:** The operation was triggered before the bot client finished connecting.

**Solution:** This is managed by the framework. If it persists, verify your bot token is valid and the Discord gateway is reachable.

#### `Discord token appears to be invalid (too short)`

**Cause:** The token is fewer than 50 characters — likely a typo or partial paste.

**Solution:** Copy the full bot token from **Discord Developer Portal → Your App → Bot → Token**. Never share this token.

#### `channelId is required`

**Cause:** A channel-based operation was triggered without a `channelId`.

**Solution:** Provide the full 18-19 digit channel ID. Right-click the channel in Discord (with Developer Mode enabled) → Copy ID.

#### `Missing Permissions` (Discord API error)

**Cause:** The bot lacks the necessary permission in the target channel or guild.

**Solution:** Go to **Server Settings → Roles** and ensure the bot role has `Send Messages`, `Add Reactions`, or `Manage Messages` as required by the operation.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Unknown Channel` | `channelId` doesn't exist or bot can't access it | Check the ID and channel visibility |
| `Unknown Message` | `messageId` not found | The message may have been deleted |
| `Cannot send messages to this user` | DM blocked | User has DMs disabled or doesn't share a server with the bot |
| `Invalid Form Body` | Embed fields are empty | At least `title` or `description` is required for embeds |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Discord Trigger](./discord-trigger.md) - Trigger workflows on Discord message events
- [Slack Action](./slack-action.md) - Send messages and manage Slack channels

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-11 | Initial release |

<!-- /SECTION: changelog -->
