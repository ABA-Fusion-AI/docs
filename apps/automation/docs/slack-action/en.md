---
node_id: "slack-action"
title: "Slack Action"
description: "Perform Slack operations — send messages, manage channels, upload files, add reactions, and retrieve user information"
category: "integrations"
subcategory: "communication"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - slack
  - messaging
  - communication
  - channels
  - action
related_nodes:
  - slack-trigger
  - function
  - cron
---

<!-- SECTION: header -->
# Slack Action

> **Category:** Integrations | **Type:** Action Node

Connect to the Slack Web API and perform operations across five resource categories — messages, channels, files, reactions, and users.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Slack Action** node authenticates using a Slack Bot Token and interacts with the Slack Web API. It supports 20 distinct sub-operations across five resource categories, covering the most common Slack automation scenarios — from sending and threading messages to managing channel membership, uploading files, and reacting to messages.

### Key Features

- **5 Resource Categories:** channel, message, file, reaction, user
- **20 Sub-Operations:** covering the full lifecycle of Slack interactions
- **Thread Support:** Reply to existing messages using a `thread_ts`
- **File Uploads:** Upload files via URL directly to a channel
- **Emoji Reactions:** Add or remove reactions from a curated set of 40+ emojis
- **Expression Support:** Channel IDs, message text, user IDs, timestamps, and file names accept dynamic expressions

### Use Cases

- Send notifications or alerts to a Slack channel from any workflow
- Post threaded replies to keep related messages grouped
- Create channels and invite users as part of an onboarding workflow
- Upload generated reports or files to a Slack channel
- React to messages to signal workflow status (✅ done, 🔥 urgent)
- Look up user information for routing or enrichment

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | `string` | ✅ Yes | Slack Bot Token — starts with `xoxb-`. Generate in **api.slack.com → OAuth & Permissions** |

### Operation Structure

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Resource category: `channel`, `message`, `file`, `reaction`, or `user` |
| `channelOperation` | `enum` | Conditional | Sub-operation when `operation` is `channel` |
| `messageOperation` | `enum` | Conditional | Sub-operation when `operation` is `message` |
| `fileOperation` | `enum` | Conditional | Sub-operation when `operation` is `file` |
| `reactionOperation` | `enum` | Conditional | Sub-operation when `operation` is `reaction` |
| `userOperation` | `enum` | Conditional | Sub-operation when `operation` is `user` |

---

### Resource: channel

| Sub-operation | Parameters | Description |
|---------------|------------|-------------|
| `list` | — | List all public channels in the workspace |
| `get` | `channelId` | Get details of a specific channel |
| `create` | `channelName`, `isPrivate` | Create a new channel |
| `rename` | `channelId`, `newName` | Rename a channel |
| `members` | `channelId` | List all member user IDs of a channel |
| `invite` | `channelId`, `inviteUserIds` | Invite one or more users to a channel (comma-separated user IDs) |

### Resource: message

| Sub-operation | Parameters | Description |
|---------------|------------|-------------|
| `send` | `messageChannelId`, `text` | Send a new message to a channel |
| `list` | `messageChannelId` | List recent messages in a channel |
| `get` | `messageChannelId`, `messageMessageTs` | Retrieve a specific message by timestamp |
| `update` | `messageChannelId`, `messageMessageTs`, `text` | Edit an existing message |
| `delete` | `messageChannelId`, `messageMessageTs` | Delete a message |
| `reply` | `messageChannelId`, `threadTs`, `text` | Post a reply in a thread |

### Resource: file

| Sub-operation | Parameters | Description |
|---------------|------------|-------------|
| `upload` | `fileUrl`, `fileName`, `fileChannelId` | Upload a file from a URL to a channel |
| `list` | `fileChannelId` | List files shared in a channel |
| `info` | `fileId` | Get metadata for a specific file |
| `delete` | `fileId` | Delete a file |

### Resource: reaction

| Sub-operation | Parameters | Description |
|---------------|------------|-------------|
| `add` | `reactionChannelId`, `reactionMessageTs`, `reaction` | Add an emoji reaction to a message |
| `remove` | `reactionChannelId`, `reactionMessageTs`, `reaction` | Remove an emoji reaction from a message |
| `list` | — | List all reactions made by the authenticated bot |

### Resource: user

| Sub-operation | Parameters | Description |
|---------------|------------|-------------|
| `get` | `userUserId` | Get full profile for a user |
| `list` | — | List all users in the workspace |
| `getStatus` | `userUserId` | Get the current status of a user |

### Supported Emoji Reactions

`thumbsup` · `thumbsdown` · `heart` · `fire` · `star` · `eyes` · `tada` · `rocket` · `white_check_mark` · `heavy_check_mark` · `x` · `warning` · `bulb` · `clap` · `pray` · `raised_hands` · `muscle` · `100` · `ok_hand` · `point_right` · `point_left` · `point_up` · `point_down` · `plus_one` · `minus_one` · `smile` · `joy` · `laughing` · `heart_eyes` · `thinking_face` · `face_with_monocle` · `confused` · `disappointed` · `cry` · `rage` · `boom` · `zap` · `hourglass_flowing_sand` · `clock` · `alarm_clock`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — available via expressions in channel IDs, message text, user IDs, etc. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result from the Slack Web API |
| `error` | `Error` | Emitted if the API call fails |

### Output Schemas by Sub-operation

**`message → send`**
```json
{
  "ok": true,
  "channel": "C08XXXXXXXX",
  "ts": "1741600000.000100",
  "message": {
    "text": "Deployment completed successfully ✅",
    "user": "U08XXXXXXXX",
    "ts": "1741600000.000100"
  }
}
```

**`message → reply`**
```json
{
  "ok": true,
  "channel": "C08XXXXXXXX",
  "ts": "1741600010.000200",
  "message": { "text": "All tests passed.", "thread_ts": "1741600000.000100" }
}
```

**`channel → get`**
```json
{
  "ok": true,
  "channel": {
    "id": "C08XXXXXXXX",
    "name": "deployments",
    "is_private": false,
    "num_members": 12
  }
}
```

**`channel → members`**
```json
{
  "ok": true,
  "members": ["U08XXXXXXXX", "U09YYYYYYYY", "U07ZZZZZZZZ"]
}
```

**`user → get`**
```json
{
  "ok": true,
  "user": {
    "id": "U08XXXXXXXX",
    "name": "alice",
    "real_name": "Alice Dupont",
    "profile": {
      "email": "alice@example.com",
      "title": "Engineering Manager"
    }
  }
}
```

**`reaction → add`**
```json
{
  "ok": true
}
```

**`file → upload`**
```json
{
  "ok": true,
  "file": {
    "id": "F08XXXXXXXX",
    "name": "report.pdf",
    "permalink": "https://files.slack.com/files/..."
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Send a Message

Post a notification to a Slack channel.

**Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "operation": "message",
  "messageOperation": "send",
  "messageChannelId": "C08XXXXXXXX",
  "text": "✅ Deployment to production completed successfully."
}
```

---

### Example: Reply in a Thread

Post a reply to an existing message thread.

**Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "operation": "message",
  "messageOperation": "reply",
  "messageChannelId": "C08XXXXXXXX",
  "threadTs": "{{input.ts}}",
  "text": "All {{input.testCount}} tests passed."
}
```

---

### Example: Create a Channel and Invite Users

Provision a new private channel and add team members.

**Step 1 — Create the channel:**
```json
{
  "token": "xoxb-your-bot-token",
  "operation": "channel",
  "channelOperation": "create",
  "channelName": "project-{{input.projectId}}",
  "isPrivate": true
}
```

**Step 2 — Invite users:**
```json
{
  "token": "xoxb-your-bot-token",
  "operation": "channel",
  "channelOperation": "invite",
  "channelId": "{{input.channel.id}}",
  "inviteUserIds": "U08XXXXXXXX,U09YYYYYYYY"
}
```

---

### Example: Add a Reaction to Signal Status

React to a message to indicate workflow completion.

**Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "operation": "reaction",
  "reactionOperation": "add",
  "reactionChannelId": "C08XXXXXXXX",
  "reactionMessageTs": "{{input.ts}}",
  "reaction": "white_check_mark"
}
```

---

### Example: Upload a Report File

Share a generated PDF to a channel.

**Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "operation": "file",
  "fileOperation": "upload",
  "fileUrl": "{{input.reportUrl}}",
  "fileName": "weekly-report-{{input.week}}.pdf",
  "fileChannelId": "C08XXXXXXXX"
}
```

---

### Example: Get User Status

Look up the current status of a Slack user.

**Configuration:**
```json
{
  "token": "xoxb-your-bot-token",
  "operation": "user",
  "userOperation": "getStatus",
  "userUserId": "U08XXXXXXXX"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: CI/CD Deployment Notification

Notify the team on deployment start, then update the message with the result.

```json
{
  "nodes": [
    {
      "id": "trigger-deploy",
      "type": "webhook-trigger",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "notify-start",
      "type": "slack-action",
      "position": { "x": 300, "y": 100 },
      "config": {
        "token": "xoxb-your-bot-token",
        "operation": "message",
        "messageOperation": "send",
        "messageChannelId": "C08XXXXXXXX",
        "text": "🚀 Deploying *{{input.service}}* to production..."
      }
    },
    {
      "id": "run-deploy",
      "type": "jenkins-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "operation": "triggerBuild",
        "jobName": "{{input.service}}"
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
        "threadTs": "{{input.ts}}",
        "text": "✅ Build #{{input.buildNumber}} completed."
      }
    }
  ]
}
```

### Common Patterns

- **Alert on Error:** Workflow error output → Slack send (post alert to #incidents)
- **Daily Digest:** Cron → Fetch data → Slack send (formatted report)
- **Approval Loop:** Slack send → [Slack Trigger](./slack-trigger.md) (wait for reaction) → Continue workflow
- **Onboarding:** New user webhook → Slack channel create → Slack invite
- **Build Status:** Jenkins trigger build → Slack reply with result in thread

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `not_authed` or `invalid_auth`

**Cause:** The `token` is invalid, expired, or missing required scopes.

**Solution:** Verify the token starts with `xoxb-`. In **api.slack.com → OAuth & Permissions**, add the required scopes for your operations (see table below) and reinstall the app.

#### `channel_not_found`

**Cause:** The `channelId` or `messageChannelId` is incorrect, or the bot is not a member of the channel.

**Solution:** Invite the bot to the channel: `/invite @YourBot`. Use `channel → list` to retrieve the correct channel ID.

#### `message_not_found`

**Cause:** The `messageMessageTs` or `threadTs` does not match any message in the channel.

**Solution:** Use `message → list` to retrieve recent message timestamps, or capture the `ts` from a prior `message → send` output.

#### `cant_invite_self`

**Cause:** The bot token's user was included in `inviteUserIds`.

**Solution:** Remove the bot's own user ID from the invite list.

### Required Bot Scopes by Operation

| Operation | Required Scopes |
|-----------|----------------|
| `message → send` | `chat:write` |
| `message → update` / `delete` | `chat:write` |
| `channel → list` | `channels:read` |
| `channel → create` | `channels:manage` |
| `channel → invite` | `channels:manage` |
| `channel → members` | `channels:read` |
| `file → upload` | `files:write` |
| `file → list` / `info` / `delete` | `files:read`, `files:write` |
| `reaction → add` / `remove` | `reactions:write` |
| `reaction → list` | `reactions:read` |
| `user → get` / `list` / `getStatus` | `users:read`, `users:read.email` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Slack Trigger](./slack-trigger.md) - Trigger workflows on incoming Slack events
- [Jenkins Action](./jenkins.md) - Trigger builds and post results to Slack
- [Jira Action](./jira.md) - Create Jira issues and notify teams on Slack

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
