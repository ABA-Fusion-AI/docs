---
node_id: "rocketchat"
title: "Rocket.Chat"
description: "Rocket.Chat API. Send messages, manage channels, and retrieve users."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-08-07"
author: "Fusion Team"
tags:
  - rocketchat
  - messaging
  - integration
  - chat
  - peer-only
related_nodes:
  - slack
  - http-request
  - function
---

<!-- SECTION: overview -->
# Rocket.Chat

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Interact with your [Rocket.Chat](https://www.rocket.chat) instance via its REST API. Send messages to channels, list channels, retrieve message history, create new channels, and fetch user lists — all from within a Fusion workflow.

### Use Cases

- **Automated Notifications:** Post alerts, reports, or system events to a Rocket.Chat channel automatically.
- **Team Communication Bots:** Build bots that send dynamic messages based on workflow triggers (new orders, errors, approvals).
- **Channel Management:** Programmatically create channels and organize teams as part of an onboarding or provisioning workflow.
- **Audit & Monitoring:** Retrieve message history from channels to audit activity or feed it into an analysis pipeline.
- **User Lookup:** Fetch the user list to validate Rocket.Chat accounts before sending targeted notifications.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | Yes | `getChannels` | The action to perform against the Rocket.Chat API. See operations below. |
| `host` | `string` | No | — | The base URL of your Rocket.Chat instance (e.g., `https://chat.mycompany.com`). |
| `userId` | `string` | No | — | The Rocket.Chat User ID used for authentication. Found in your account settings. |
| `authToken` | `string` | No | — | The Rocket.Chat Auth Token for the authenticated user. Used as the `X-Auth-Token` header. |
| `channelId` | `string` | No | — | The Rocket.Chat channel ID. Required for `sendMessage`, `getMessages`. |
| `message` | `string` | No | — | The message text to send. Only used with `sendMessage`. |
| `channelName` | `string` | No | — | The channel name (without `#`). Used with `createChannel`. |

### Available Operations

| Operation | Description | Required Parameters |
|-----------|-------------|---------------------|
| `sendMessage` | Post a text message to a channel. | `channelId`, `message` |
| `getChannels` | List all public channels in the Rocket.Chat instance. | — |
| `getMessages` | Retrieve recent messages from a specific channel. | `channelId` |
| `createChannel` | Create a new public channel. | `channelName` |
| `getUsers` | Retrieve the list of all users in the instance. | — |

### Parameter Visibility by Operation

| Parameter | `sendMessage` | `getChannels` | `getMessages` | `createChannel` | `getUsers` |
|-----------|:---:|:---:|:---:|:---:|:---:|
| `host` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `userId` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `authToken` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `channelId` | ✅ | — | ✅ | — | — |
| `message` | ✅ | — | — | — | — |
| `channelName` | — | — | — | ✅ | — |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Data and configuration supplied by the preceding workflow node. Parameters can be passed via expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The Rocket.Chat API response for the selected operation. |
| `error` | `Error` | Emitted on validation errors, authentication failures, network issues, or API errors. |

### Output Examples

#### `sendMessage`

```json
{
  "success": true,
  "message": {
    "_id": "messageid123",
    "rid": "GENERAL",
    "msg": "Hello from Fusion!",
    "ts": "2026-08-07T09:00:00.000Z",
    "u": {
      "_id": "userId123",
      "username": "fusion-bot"
    }
  }
}
```

#### `getChannels`

```json
{
  "success": true,
  "channels": [
    { "_id": "GENERAL", "name": "general", "usersCount": 42 },
    { "_id": "ch_dev", "name": "dev-team", "usersCount": 10 }
  ],
  "total": 2
}
```

#### `getUsers`

```json
{
  "success": true,
  "users": [
    { "_id": "u1", "username": "abdelkhalek", "name": "Abdelkhalek", "status": "online" },
    { "_id": "u2", "username": "fusion-bot", "name": "Fusion Bot", "status": "online" }
  ],
  "total": 2
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Rocket.Chat in a Workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **Rocket.Chat Node:** Executes the configured operation (e.g., `sendMessage`) using the provided `host`, `userId`, `authToken`, and channel parameters.
3. **Log Node:** Displays the API response.

### Common Patterns

- **Alert on Error:** Connect the `error` output of any node to a Rocket.Chat node with `sendMessage` to notify a channel when a workflow fails.
- **Daily Digest:** Use a Cron trigger to run `getMessages` every morning and summarize overnight activity with an AI Chat node before posting back to the channel.
- **Onboarding Automation:** When a new user is created in your system, use `createChannel` to set up a dedicated channel and send a welcome `sendMessage` automatically.
- **Channel Roster Sync:** Periodically call `getUsers` and cross-reference with an internal directory to detect inactive accounts.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `authToken` and `userId` in Fusion's **Secrets** system. Do not paste credentials directly into workflow parameters or commit them to version control.

- Rocket.Chat auth tokens do not expire by default but can be revoked from the Rocket.Chat admin panel under **Administration → Users**.
- Always use a **dedicated bot user** for automation — avoid using personal credentials.
- Restrict the bot user's permissions to only the channels and actions required by your workflows.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Unauthorized` — Authentication failed
- **Cause:** The `userId` or `authToken` is missing, incorrect, or the token has been revoked.
- **Solution:** Verify both values in your Rocket.Chat account settings under **My Account → Personal Access Tokens**. Regenerate the token if needed.

#### `Channel not found`
- **Cause:** The `channelId` does not match an existing channel, or the bot user does not have access to it.
- **Solution:** Use `getChannels` first to retrieve the correct `_id` for the target channel. Ensure the bot user is a member of the channel.

#### `sendMessage` posts nothing / no error
- **Cause:** The `message` field is empty.
- **Solution:** Provide a non-empty string in the `message` parameter. You can use an expression to build the message dynamically from upstream data.

#### `createChannel` fails — channel already exists
- **Cause:** A channel with the same `channelName` already exists in the instance.
- **Solution:** Use `getChannels` to check if the channel exists before attempting to create it. Add an If/Else node to skip creation if the channel is already present.

#### Connection refused / network error
- **Cause:** The `host` URL is incorrect, uses HTTP instead of HTTPS, or the Rocket.Chat instance is unreachable.
- **Solution:** Verify the `host` URL (e.g., `https://chat.mycompany.com`) and confirm the instance is running and accessible from the workflow execution environment.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Slack](./slack.md) – Send messages and manage Slack workspaces
- [HTTP Request](./http-request.md) – Call Rocket.Chat REST API endpoints not covered by this node
- [Function](./function.md) – Dynamically build message text or channel names from workflow data
- [Cron](./cron.md) – Schedule periodic Rocket.Chat operations

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-07 | Initial documentation |

<!-- /SECTION: changelog -->
