---
node_id: "slack-trigger"
title: "Slack Trigger"
description: "Trigger workflows on Slack events — messages, reactions, channel changes, and member activity — using Slack Socket Mode"
category: "triggers"
subcategory: "communication"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - slack
  - trigger
  - events
  - real-time
  - socket-mode
related_nodes:
  - slack-action
  - function
  - kafka-trigger
---

<!-- SECTION: header -->
# Slack Trigger

> **Category:** Triggers | **Type:** Trigger Node

Listen to real-time Slack events using Socket Mode and trigger a workflow for each incoming event — messages, reactions, channel lifecycle events, and member activity.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Slack Trigger** node connects to Slack using the **Bolt SDK in Socket Mode**, which establishes a persistent WebSocket connection without requiring a public HTTPS endpoint. It listens for multiple event types simultaneously and emits a trigger output for each received event with a structured payload.

Bot messages are automatically filtered out to prevent feedback loops. An optional `channelId` filter limits events to a specific channel.

### Key Features

- **Socket Mode:** Real-time WebSocket connection — no public URL required
- **7 Event Types:** message, reaction_added, reaction_removed, channel_created, channel_deleted, member_joined_channel, member_left_channel
- **Channel Filter:** Optionally restrict triggers to a specific channel
- **Bot Message Filtering:** Messages from bots are automatically ignored
- **Structured Payloads:** Each event type has a consistent, typed output structure
- **Pause / Resume:** Stop and restart the WebSocket connection without losing configuration

### Use Cases

- Respond to incoming Slack messages with automated replies
- Trigger an approval workflow when a specific reaction is added
- Log channel membership changes for audit purposes
- Notify a system when a new Slack channel is created
- Build a Slack bot that reacts to user messages in real time

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | `string` | ✅ Yes | Slack Bot Token (`xoxb-...`) — requires event subscription scopes |
| `signingSecret` | `string` | ✅ Yes | Slack App signing secret — found in **api.slack.com → Basic Information** |
| `appToken` | `string` | ✅ Yes | Slack App-level token (`xapp-...`) — required for Socket Mode, generate in **App-Level Tokens** |

### Filter Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `channelId` | `string` | ❌ No | — | If set, only events from this channel will trigger the workflow |

### Setting Up Slack Socket Mode

1. Go to **api.slack.com → Your App → Socket Mode** and enable it
2. Generate an **App-Level Token** with the `connections:write` scope — this is your `appToken`
3. In **Event Subscriptions**, subscribe to the events listed below
4. In **OAuth & Permissions**, add the required Bot Token Scopes
5. Install the app in your workspace and copy the `Bot User OAuth Token`

### Required Event Subscriptions & Scopes

| Event | Required Bot Scope |
|-------|--------------------|
| `message.channels` | `channels:history` |
| `reaction_added` / `reaction_removed` | `reactions:read` |
| `channel_created` | `channels:read` |
| `channel_deleted` | `channels:read` |
| `member_joined_channel` / `member_left_channel` | `channels:read` |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it activates automatically when a Slack event is received. It does not accept inputs from other nodes.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted for each received Slack event |
| `error` | `Error` | Emitted on WebSocket or Bolt SDK errors |

### Output Schemas by Event Type

**`message`** (new message posted by a user)
```json
{
  "operation": "message",
  "channel": "C08XXXXXXXX",
  "user": "U08XXXXXXXX",
  "text": "Can you run the staging deployment?",
  "ts": "1741600000.000100",
  "thread_ts": null,
  "event_ts": "1741600000.000100"
}
```

> Bot messages are automatically filtered — only human-posted messages are emitted.

**`reaction_added`**
```json
{
  "operation": "reaction_added",
  "user": "U08XXXXXXXX",
  "reaction": "white_check_mark",
  "channel": "C08XXXXXXXX",
  "message_ts": "1741600000.000100",
  "event_ts": "1741600010.000200"
}
```

**`reaction_removed`**
```json
{
  "operation": "reaction_removed",
  "user": "U08XXXXXXXX",
  "reaction": "x",
  "channel": "C08XXXXXXXX",
  "message_ts": "1741600000.000100",
  "event_ts": "1741600015.000300"
}
```

**`channel_created`**
```json
{
  "operation": "channel_created",
  "channel": "C09XXXXXXXX",
  "channel_name": "project-alpha",
  "creator": "U08XXXXXXXX",
  "event_ts": "1741600020"
}
```

**`channel_deleted`**
```json
{
  "operation": "channel_deleted",
  "channel": "C09XXXXXXXX",
  "event_ts": "1741600030000"
}
```

**`member_joined_channel`**
```json
{
  "operation": "member_joined_channel",
  "user": "U09YYYYYYYY",
  "channel": "C08XXXXXXXX",
  "channel_type": "C",
  "inviter": "U08XXXXXXXX",
  "event_ts": "1741600040.000500"
}
```

**`member_left_channel`**
```json
{
  "operation": "member_left_channel",
  "user": "U09YYYYYYYY",
  "channel": "C08XXXXXXXX",
  "channel_type": "C",
  "event_ts": "1741600050.000600"
}
```

| Field | Description |
|-------|-------------|
| `operation` | The event type that fired the trigger |
| `channel` | Slack channel ID where the event occurred |
| `user` | Slack user ID of the actor |
| `ts` / `event_ts` | Slack timestamp of the event (used to reference messages) |
| `thread_ts` | Thread timestamp if the message is part of a thread |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Listen to All Messages in a Channel

Trigger for every human message in channel `C08XXXXXXXX`.

**Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "signingSecret": "your-signing-secret",
  "appToken": "xapp-your-app-token",
  "channelId": "C08XXXXXXXX"
}
```

**Output (on new message):**
```json
{
  "operation": "message",
  "channel": "C08XXXXXXXX",
  "user": "U08XXXXXXXX",
  "text": "deploy staging",
  "ts": "1741600000.000100",
  "thread_ts": null,
  "event_ts": "1741600000.000100"
}
```

---

### Example: Listen Across All Channels

Leave `channelId` empty to receive events from all channels the bot is a member of.

**Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "signingSecret": "your-signing-secret",
  "appToken": "xapp-your-app-token"
}
```

---

### Example: React to Specific Reactions

Use a Function node after the trigger to filter for a specific reaction and act on it.

**Trigger Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "signingSecret": "your-signing-secret",
  "appToken": "xapp-your-app-token",
  "channelId": "C08XXXXXXXX"
}
```

**Function node (filter + route):**
```js
if (input.operation !== 'reaction_added') return null;
if (input.reaction !== 'white_check_mark') return null;
return { approved: true, messageTs: input.message_ts, approver: input.user };
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Reaction-Based Approval Gate

A team lead reacts with ✅ to approve a deployment request posted by a bot.

```json
{
  "nodes": [
    {
      "id": "wait-for-approval",
      "type": "slack-trigger",
      "position": { "x": 100, "y": 100 },
      "config": {
        "token": "xoxb-your-bot-token",
        "signingSecret": "your-signing-secret",
        "appToken": "xapp-your-app-token",
        "channelId": "C08XXXXXXXX"
      }
    },
    {
      "id": "check-approval",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "if (input.operation !== 'reaction_added' || input.reaction !== 'white_check_mark') return null; return input;"
      }
    },
    {
      "id": "trigger-build",
      "type": "jenkins-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "operation": "triggerBuild",
        "jobName": "deploy-production"
      }
    },
    {
      "id": "notify-result",
      "type": "slack-action",
      "position": { "x": 700, "y": 100 },
      "config": {
        "token": "xoxb-your-bot-token",
        "operation": "message",
        "messageOperation": "reply",
        "messageChannelId": "C08XXXXXXXX",
        "threadTs": "{{input.message_ts}}",
        "text": "🚀 Deployment triggered by <@{{input.user}}>."
      }
    }
  ]
}
```

### Common Patterns

- **Bot Reply:** Slack Trigger (`message`) → Function (parse command) → [Slack Action](./slack-action.md) (reply in thread)
- **Approval Gate:** [Slack Action](./slack-action.md) post request → Slack Trigger (`reaction_added`) → Continue workflow
- **Member Onboarding:** Slack Trigger (`member_joined_channel`) → Send welcome DM
- **Audit Log:** Slack Trigger (`any event`) → PostgreSQL insert
- **Command Handler:** Slack Trigger (`message`) → Function (check `text.startsWith('/deploy')`) → Jenkins trigger build

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Node starts but never triggers

**Cause:** The Slack app is not subscribed to the relevant events, or the bot is not a member of the target channel.

**Solution:** In **api.slack.com → Event Subscriptions**, verify the required events are subscribed. Invite the bot to the target channel with `/invite @YourBot`.

#### `invalid_app_token` error

**Cause:** The `appToken` is missing or incorrect.

**Solution:** Generate an App-Level Token in **api.slack.com → Basic Information → App-Level Tokens** with the `connections:write` scope. The token must start with `xapp-`.

#### `signing_secret_verification_failed`

**Cause:** The `signingSecret` does not match the app's signing secret.

**Solution:** Copy the signing secret from **api.slack.com → Basic Information → App Credentials**.

#### Bot messages trigger the workflow

**Cause:** This should not happen — bot messages are automatically filtered by checking `msg.bot_id`.

**Solution:** If you are seeing bot-originated triggers, verify that the message does not have a missing `bot_id` (e.g. some workflow bots may not set this field). Add a guard in a Function node: `if (input.user === 'USLACKBOT') return null;`.

#### `channelId` filter not working

**Cause:** The channel ID format may be wrong (e.g. using a channel name instead of ID).

**Solution:** Channel IDs start with `C` (e.g. `C08XXXXXXXX`). Use the [Slack Action](./slack-action.md) `channel → list` to retrieve the correct ID.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `invalid_auth` | Invalid Bot Token | Check `token` value and scopes |
| `invalid_app_token` | Invalid App-Level Token | Regenerate `appToken` with `connections:write` scope |
| `not_allowed_token_type` | Wrong token type | Use `xoxb-` for `token` and `xapp-` for `appToken` |
| `channel_not_found` | Channel not in bot's scope | Invite the bot to the channel |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Slack Action](./slack-action.md) - Send messages, manage channels, upload files
- [Kafka Trigger](./kafka-trigger.md) - Real-time event trigger via Kafka
- [Redis Subscribe](./redis-subscribe.md) - Pub/Sub trigger via Redis

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
