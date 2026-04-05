---
node_id: "discord-trigger"
title: "Discord Trigger"
description: "Trigger workflows on Discord message events — listens for message creation, edits, and deletions with optional channel filtering"
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
  - trigger
  - real-time
related_nodes:
  - discord-action
  - slack-trigger
  - webhook-trigger
---

<!-- SECTION: header -->
# Discord Trigger

> **Category:** Integrations | **Type:** Trigger Node

Listen to Discord message events in real time and trigger a workflow whenever a message is created, updated, or deleted — with optional filtering by channel.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Discord Trigger** node connects to Discord via the `discord.js` WebSocket client and registers listeners for three message lifecycle events: `messageCreate`, `messageUpdate`, and `messageDelete`. It emits a workflow trigger for each matching event.

When `channelIds` is provided, only events from those channels are forwarded. When left empty, all channels the bot can read are monitored.

Each emitted payload includes a fully structured `DiscordMessage` object with author details, channel info, guild data, attachments, embeds, and mentions.

### Key Features

- **3 Events:** `messageCreate`, `messageUpdate`, `messageDelete`
- **Channel Filtering:** Optionally restrict to one or more specific channel IDs
- **Rich Message Payload:** Full author, channel, guild, attachment, embed, and mention data
- **Partial Event Protection:** `messageUpdate` and `messageDelete` skip partial (uncached) messages
- **Graceful Lifecycle:** Supports pause (listeners remain registered) and stop (full disconnect)

### Use Cases

- Trigger a moderation pipeline whenever a message is sent to a monitored channel
- Log all message edits and deletions to a database for compliance auditing
- React to specific keywords in messages by chaining with a Function node
- Build a Discord bot that responds to commands sent in a `#commands` channel
- Feed Discord messages into a sentiment analysis or AI summarization workflow

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `token` | `string` | ✅ Yes | — | Discord bot token — minimum 50 characters |
| `channelIds` | `string[]` | ❌ No | `[]` (all channels) | List of channel IDs to monitor. Empty = listen to all accessible channels |

> **Getting a channel ID:** Enable Developer Mode in Discord (**Settings → Advanced → Developer Mode**), then right-click any channel and choose **Copy Channel ID**.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted on each matching message event |
| `error` | `Error` | Emitted if the client fails or encounters an error |

### Output Schema

All three events share the same structure:

```json
{
  "session": {
    "event": "messageCreate",
    "messageId": "1234567890123456789",
    "channelId": "9876543210987654321",
    "guildId": "1122334455667788990"
  },
  "data": {
    "event": "messageCreate",
    "message": {
      "id": "1234567890123456789",
      "content": "Hello, this is a test message!",
      "author": {
        "id": "1111111111111111111",
        "username": "driss",
        "bot": false,
        "avatar": "abc123hash"
      },
      "channel": {
        "id": "9876543210987654321",
        "name": "general",
        "type": 0
      },
      "guild": {
        "id": "1122334455667788990",
        "name": "Fusion Dev Server"
      },
      "timestamp": "2026-03-11T10:00:00.000Z",
      "attachments": [],
      "embeds": [],
      "mentions": []
    },
    "timestamp": "2026-03-11T10:00:00.000Z"
  }
}
```

### `message` Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique Discord message ID (Snowflake) |
| `content` | `string` | Plain text content of the message |
| `author.id` | `string` | Discord user ID of the sender |
| `author.username` | `string` | Display name of the sender |
| `author.bot` | `boolean` | Whether the sender is a bot |
| `author.avatar` | `string \| undefined` | Avatar hash (use with Discord CDN URL) |
| `channel.id` | `string` | Channel ID where the message was sent |
| `channel.name` | `string \| undefined` | Channel name (undefined for DMs) |
| `channel.type` | `number` | Discord channel type (0 = text, 1 = DM, etc.) |
| `guild` | `object \| undefined` | Guild info — undefined for DM messages |
| `timestamp` | `string` | ISO 8601 creation timestamp |
| `attachments` | `array` | File attachments (id, filename, url, size) |
| `embeds` | `array` | Raw embed objects included in the message |
| `mentions` | `array` | List of mentioned users (id, username) |

### Event Types

| Event | `data.event` | Trigger Condition |
|-------|-------------|-------------------|
| `messageCreate` | `"messageCreate"` | A new message is posted |
| `messageUpdate` | `"messageUpdate"` | An existing message is edited |
| `messageDelete` | `"messageDelete"` | A message is deleted |

> `messageUpdate` and `messageDelete` are skipped for partial messages (messages not in discord.js cache).

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Monitor All Channels

Listen to all message events across every channel the bot can read.

**Configuration:**
```json
{
  "token": "your-discord-bot-token",
  "channelIds": []
}
```

---

### Example: Monitor Specific Channels

Restrict to a `#support` and `#alerts` channel.

**Configuration:**
```json
{
  "token": "your-discord-bot-token",
  "channelIds": ["1234567890123456789", "9876543210987654321"]
}
```

---

### Example: Filter Messages with a Function Node

Listen for messages and process only those containing a keyword.

```json
{
  "nodes": [
    {
      "id": "discord-listener",
      "type": "discord-trigger",
      "config": {
        "token": "your-bot-token",
        "channelIds": ["1234567890123456789"]
      }
    },
    {
      "id": "keyword-filter",
      "type": "function",
      "config": {
        "code": "if (!input.data.message.content.toLowerCase().includes('!help')) return null; return input;"
      }
    }
  ]
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Discord Message → Jira Issue (Command Bot)

When a user posts `!bug <description>` in `#bug-reports`, automatically create a Jira issue.

```json
{
  "nodes": [
    {
      "id": "discord-listener",
      "type": "discord-trigger",
      "config": {
        "token": "your-bot-token",
        "channelIds": ["1234567890123456789"]
      }
    },
    {
      "id": "parse-command",
      "type": "function",
      "config": {
        "code": "const msg = input.data.message.content; if (!msg.startsWith('!bug ')) return null; return { description: msg.replace('!bug ', ''), author: input.data.message.author.username, messageId: input.data.message.id, channelId: input.session.channelId };"
      }
    },
    {
      "id": "create-jira-issue",
      "type": "jira-action",
      "config": {
        "operation": "createIssue",
        "projectKey": "BUGS",
        "issueData": {
          "summary": "{{input.description}}",
          "description": "Reported by {{input.author}} via Discord",
          "issuetype": { "name": "Bug" }
        }
      }
    },
    {
      "id": "confirm-discord",
      "type": "discord-action",
      "config": {
        "token": "your-bot-token",
        "operation": "replyToMessage",
        "channelId": "{{input.channelId}}",
        "messageId": "{{input.messageId}}",
        "content": "✅ Jira issue created: **{{input.key}}**"
      }
    }
  ]
}
```

### Common Patterns

- **Keyword Bot:** Message event → Function (parse command) → Action (reply/react/store)
- **Moderation:** Message event → Function (check blocked words) → Discord `deleteMessage` + `sendDM`
- **Audit Log:** All message events → MongoDB insert (compliance logging)
- **Relay:** Discord message → Slack `send` (cross-platform relay)
- **Sentiment Analysis:** Message event → Function (call AI API) → Discord `addReaction` (👍/👎)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Client not initialized`

**Cause:** `run()` was called before `setup()` completed.

**Solution:** This is managed by the framework. If it occurs, check for token validation errors in `setup()`.

#### Messages from channels not received

**Cause:** The bot lacks the `VIEW_CHANNEL` or `READ_MESSAGE_HISTORY` permission in those channels, or the channel IDs in `channelIds` are incorrect.

**Solution:** Verify the bot's role permissions for each target channel. Confirm channel IDs with Developer Mode → right-click → Copy ID.

#### `messageUpdate` events not triggering

**Cause:** Discord.js only emits `messageUpdate` for messages that are cached (typically recent messages). Very old messages may not trigger this event.

**Solution:** This is a Discord.js cache limitation. For comprehensive edit tracking, increase the message cache size in the bot configuration or use Discord's audit log API.

#### Bot receives its own messages

**Cause:** Discord sends all messages to the listener, including those sent by the bot itself.

**Solution:** Add a filter in a downstream Function node: `if (input.data.message.author.bot) return null;`

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| `Token is invalid` | Wrong or expired bot token | Regenerate the token in the Developer Portal |
| `Disallowed intents` | Bot not granted `MESSAGE_CONTENT` privileged intent | Enable it in Developer Portal → Bot → Privileged Gateway Intents |
| `Already running` | `run()` called while already active | Normal — the node logs and skips re-registration |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Discord Action](./discord-action.md) - Send messages, embeds, reactions, and DMs on Discord
- [Slack Trigger](./slack-trigger.md) - Trigger workflows on Slack message events

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-11 | Initial release |

<!-- /SECTION: changelog -->
