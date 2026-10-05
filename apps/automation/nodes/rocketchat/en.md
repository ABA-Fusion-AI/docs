---
node_id: "rocketchat"
title: "Rocket.Chat"
description: "Rocket.Chat API. Send messages, manage channels, and retrieve users."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-05"
author: "Fusion Team"
tags:
  - rocketchat
  - messaging
  - integration
  - chat
  - peer-only
related_nodes:
  - slack-action
  - http-request
  - function
---

<!-- SECTION: overview -->
# Rocket.Chat

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Use the Rocket.Chat REST API from a Fusion workflow to send a channel message, list channels, retrieve channel history, create a channel, or list users. The node calls the `/api/v1` API on the configured Rocket.Chat server.

### Use Cases

- **Automated notifications:** Post workflow alerts or reports to a Rocket.Chat channel.
- **Channel management:** Create a channel as part of a provisioning workflow.
- **Message history:** Retrieve a channel's history for downstream processing.
- **User listing:** Retrieve the instance's user list for downstream workflow steps.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

The six string parameters are optional in the schema and support expressions. At runtime, every operation requires non-empty `host`, `userId`, and `authToken` values. The `operation` enum defaults to `getChannels` and has no expression metadata.

| Parameter | Type | Required at runtime | Default | Description |
|-----------|------|---------------------|---------|-------------|
| `operation` | `enum` | No | `getChannels` | Operation to perform: `sendMessage`, `getChannels`, `getMessages`, `createChannel`, or `getUsers`. |
| `host` | `string` | Yes | — | Rocket.Chat server URL, for example `https://chat.example.com`. |
| `userId` | `string` | Yes | — | Rocket.Chat user ID used in the `X-User-Id` authentication header. |
| `authToken` | `string` | Yes | — | Rocket.Chat auth token used in the `X-Auth-Token` authentication header. |
| `channelId` | `string` | For `sendMessage` and `getMessages` | — | Channel ID. |
| `message` | `string` | For `sendMessage` | — | Message text to send. Must not be empty. |
| `channelName` | `string` | For `createChannel` | — | Name of the channel to create. Must not be empty. |

### Operations

| Operation | Request | Required operation parameters |
|-----------|---------|--------------------------------|
| `getChannels` | `GET /api/v1/channels.list` | — |
| `sendMessage` | `POST /api/v1/chat.sendMessage` | `channelId`, `message` |
| `getMessages` | `GET /api/v1/channels.history?roomId=<channelId>` | `channelId` |
| `createChannel` | `POST /api/v1/channels.create` | `channelName` |
| `getUsers` | `GET /api/v1/users.list` | — |

The node sends `X-User-Id`, `X-Auth-Token`, and `Content-Type: application/json` headers. For `sendMessage`, the JSON body is `{ "message": { "rid": "<channelId>", "msg": "<message>" } }`. For `createChannel`, the body is `{ "name": "<channelName>" }`.

The node removes one trailing slash from `host` before appending `/api/v1`. Supply the server URL without the API path. For `getMessages`, `channelId` is URL-encoded in the `roomId` query parameter.

### Configuration Examples

Replace credential placeholders with protected values or expressions.

#### List Channels (Default Operation)

```json
{
  "host": "https://chat.example.com",
  "userId": "YOUR_USER_ID",
  "authToken": "YOUR_AUTH_TOKEN"
}
```

#### Send a Message

```json
{
  "operation": "sendMessage",
  "host": "https://chat.example.com",
  "userId": "YOUR_USER_ID",
  "authToken": "YOUR_AUTH_TOKEN",
  "channelId": "YOUR_CHANNEL_ID",
  "message": "Workflow completed successfully."
}
```

#### Retrieve Channel History

```json
{
  "operation": "getMessages",
  "host": "https://chat.example.com",
  "userId": "YOUR_USER_ID",
  "authToken": "YOUR_AUTH_TOKEN",
  "channelId": "YOUR_CHANNEL_ID"
}
```

#### Create a Channel

```json
{
  "operation": "createChannel",
  "host": "https://chat.example.com",
  "userId": "YOUR_USER_ID",
  "authToken": "YOUR_AUTH_TOKEN",
  "channelName": "workflow-alerts"
}
```

#### List Users

```json
{
  "operation": "getUsers",
  "host": "https://chat.example.com",
  "userId": "YOUR_USER_ID",
  "authToken": "YOUR_AUTH_TOKEN"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow data triggers execution but is not read directly by the handler. Operation parameters come from the node configuration. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The JSON response returned by the Rocket.Chat endpoint selected by `operation`. The response shape depends on the endpoint. |
| `error` | `Error` | An error is raised if required configuration is missing, an operation-specific parameter is missing, the request fails, or the API returns a non-success HTTP status. |

The handler returns parsed API JSON directly, without wrapping or transforming it. It checks the HTTP status with `res.ok` but does not inspect an API-level `success` field. A failure reported inside a successful HTTP response is therefore returned as JSON.

Network failures and JSON parsing failures propagate as errors. HTTP errors include the status and status text, but not the response body. Each execution makes one request; the implementation provides no retries, pagination controls, or automatic retrieval of additional pages.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Rocket.Chat in a Workflow
```

The embedded example contains manual-trigger paths for listing users, sending a message, retrieving channel history, and creating a channel, with results passed to Log nodes. Configure the server and credentials before running it.

### Common Patterns

- **Alert on error:** Connect an error-handling path to a Rocket.Chat node configured with `sendMessage`.
- **Channel history processing:** Use `getMessages` to pass a channel's returned history to downstream workflow steps.
- **Channel provisioning:** Use `createChannel` to create a channel, then use `sendMessage` to post an initial message.
- **User listing:** Use `getUsers` to pass the returned user list to downstream workflow steps.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

- Store `userId` and `authToken` in Fusion's **Secrets** system rather than directly in workflow parameters.
- Use a dedicated Rocket.Chat account for automation and grant it only the permissions needed by the workflows.
- Use HTTPS for the `host` URL when the Rocket.Chat server supports it.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `host, userId, and authToken are required`
- **Cause:** One or more authentication settings are missing.
- **Solution:** Provide the Rocket.Chat server URL, user ID, and auth token.

#### `channelId and message are required`
- **Cause:** `sendMessage` was selected without both required parameters, or `message` is empty.
- **Solution:** Set a channel ID and a non-empty message.

#### `channelId is required`
- **Cause:** `getMessages` was selected without a channel ID.
- **Solution:** Set `channelId` to the target channel's ID.

#### `channelName is required`
- **Cause:** `createChannel` was selected without a channel name.
- **Solution:** Set `channelName` to a non-empty value.

#### `Rocket.Chat Error: <status> <statusText>`
- **Cause:** The API request returned a non-success HTTP status. This can occur if the credentials are invalid, the account lacks permission, or the requested resource is unavailable.
- **Solution:** Check the Rocket.Chat server URL, credentials, account permissions, and operation parameters.

#### `Unknown operation: <operation>`
- **Cause:** An unsupported operation reached the handler. The schema normally restricts the operation to the five documented values.
- **Solution:** Select a supported operation.

#### A Request Fails or the Response Cannot Be Parsed
- **Cause:** The server cannot be reached, or its response is not valid JSON.
- **Solution:** Check connectivity and ensure `host` points to the Rocket.Chat server rather than a page or a URL already ending in `/api/v1`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Slack Action](../slack-action/en.md) – Send messages and manage Slack workspaces
- [HTTP Request](../http-request/en.md) – Call Rocket.Chat REST API endpoints not covered by this node
- [Function](../function/en.md) – Dynamically build message text or channel names from workflow data

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-05 | Regenerated documentation to match the implementation. |

<!-- /SECTION: changelog -->
