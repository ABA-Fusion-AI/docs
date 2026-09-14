---
node_id: "twist"

title: "Twist"

description: "Manage Twist channels, threads, messages, and conversations using the Twist API."

category: "Communication / Team Messaging"

version: "1.0.0"

language: "en"

last_updated: "2026-09-14"

author: "Fusion Team"

tags:

- twist
- messaging
- channels
- threads
- conversations
- comments
- collaboration
- oauth2
- api

related_nodes:

- function
- if
- http-request

---

**# Twist**

> **\*\*Category:\*\*** communication-nodes | **\*\*Type:\*\*** Action Node

Integrate **\*\*Twist\*\*** team messaging into Fusion workflows using the Twist API v3.

The **\*\*Twist\*\*** node exposes four operations: `getChannels`, `postMessage`, `getThreads`, and `getConversations`.

All operations require a Twist OAuth2 access token. Workspace and channel identifiers are required depending on the selected operation.

**### Supported Features**

\- Retrieve channels from a Twist workspace

\- Post messages or comments to a Twist channel

\- Optionally associate a posted message with a thread

\- Retrieve threads from a channel

\- Retrieve workspace conversations

\- Authenticate using a Twist OAuth2 Bearer access token

\- Validate operation-specific required parameters

\- URL-encode workspace and channel identifiers used in GET requests

\- Return parsed Twist API JSON responses directly

**### Use Cases**

\- Retrieve available Twist channels

\- Post workflow notifications to Twist

\- Add content associated with a Twist thread

\- Retrieve channel threads for workflow processing

\- Retrieve workspace conversations

\- Integrate Twist collaboration data into automated workflows

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `operation` | `enum` | ❌ No | `"getChannels"` | `getChannels`, `postMessage`, `getThreads`, or `getConversations`. |
| `accessToken` | `string` | ✅ Yes | — | Twist OAuth2 access token required at runtime for every operation. |
| `workspaceId` | `string` | ❌ No | — | Required for `getChannels` and `getConversations`. |
| `channelId` | `string` | ❌ No | — | Required for `postMessage` and `getThreads`. |
| `content` | `string` | ❌ No | — | Required for `postMessage`. |
| `threadId` | `string` | ❌ No | — | Optional thread ID included when posting. |

**### Operation-Specific Parameters**

| Operation | Required Parameters | Optional Parameters |
| --------- | ------------------- | ------------------- |
| `getChannels` | `accessToken`, `workspaceId` | — |
| `postMessage` | `accessToken`, `channelId`, `content` | `threadId` |
| `getThreads` | `accessToken`, `channelId` | — |
| `getConversations` | `accessToken`, `workspaceId` | — |

**---**

**## Operations**

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| `getChannels` | `/channels/get?workspace_id={workspaceId}` | `GET` | Retrieve channels for a workspace. |
| `postMessage` | `/comments/add` | `POST` | Post content to a channel, optionally with a thread ID. |
| `getThreads` | `/threads/get?channel_id={channelId}` | `GET` | Retrieve threads for a channel. |
| `getConversations` | `/conversations/get?workspace_id={workspaceId}` | `GET` | Retrieve conversations for a workspace. |

The fixed base URL is:

```text
https://api.twist.com/api/v3
```

**---**

**## Request Body Construction**

**### Get Channels**

```text
GET https://api.twist.com/api/v3/channels/get?workspace_id=<workspaceId>
```

`workspaceId` is encoded using `encodeURIComponent()`.

**### Post Message**

```json
{
  "channel_id": "847582",
  "content": "Hello from Fusion"
}
```

If `threadId` is supplied:

```json
{
  "channel_id": "847582",
  "content": "Reply from Fusion",
  "thread_id": "8046649"
}
```

The node transforms:

```text
channelId → channel_id
threadId → thread_id
```

**### Get Threads**

```text
GET https://api.twist.com/api/v3/threads/get?channel_id=<channelId>
```

`channelId` is encoded using `encodeURIComponent()`.

**### Get Conversations**

```text
GET https://api.twist.com/api/v3/conversations/get?workspace_id=<workspaceId>
```

**### Request Headers**

Every request includes:

```text
Authorization: Bearer <accessToken>
Content-Type: application/json
```

**---**

**## Inputs & Outputs**

**### Inputs**

The node receives `incomingData`, but the implementation does not use it.

All operation parameters are read from node configuration.

**### Outputs**

Successful requests return:

```text
response.json()
```

directly.

The node does not wrap, normalize, rename, or filter successful Twist API responses.

**---**

**## Configuration Examples**

**### Get Channels**

```json
{
  "operation": "getChannels",
  "accessToken": "YOUR_TWIST_ACCESS_TOKEN",
  "workspaceId": "YOUR_WORKSPACE_ID"
}
```

**### Post Message**

```json
{
  "operation": "postMessage",
  "accessToken": "YOUR_TWIST_ACCESS_TOKEN",
  "channelId": "847582",
  "content": "Hello from Fusion"
}
```

**### Post Message With Thread ID**

```json
{
  "operation": "postMessage",
  "accessToken": "YOUR_TWIST_ACCESS_TOKEN",
  "channelId": "847582",
  "content": "Reply from Fusion",
  "threadId": "8046649"
}
```

**### Get Threads**

```json
{
  "operation": "getThreads",
  "accessToken": "YOUR_TWIST_ACCESS_TOKEN",
  "channelId": "847582"
}
```

**### Get Conversations**

```json
{
  "operation": "getConversations",
  "accessToken": "YOUR_TWIST_ACCESS_TOKEN",
  "workspaceId": "YOUR_WORKSPACE_ID"
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Twist (`postMessage`)

\- Scheduler → Twist (`getChannels`)

\- Twist (`getThreads`) → Function

\- Twist (`getConversations`) → Function

\- Twist → If

\- External Event → Function → Twist (`postMessage`)

**---**

**## Error Handling**

**### Missing Access Token**

```text
accessToken is required
```

**### Missing Workspace ID**

For `getChannels` and `getConversations`:

```text
workspaceId is required
```

**### Missing Channel ID and Content**

For `postMessage`:

```text
channelId and content are required
```

**### Missing Channel ID**

For `getThreads`:

```text
channelId is required
```

**### Twist API Error**

```text
Twist API Error: <status> <statusText> - <response body>
```

The response-body portion is only appended when error text exists.

**### Unknown Operation**

```text
Unknown operation: <operation>
```

**---**

**## Troubleshooting**

**### "accessToken is required"**

Provide a Twist OAuth2 access token in `accessToken`.

---

**### "workspaceId is required"**

Provide `workspaceId` when using `getChannels` or `getConversations`.

---

**### "channelId and content are required"**

Provide both values for `postMessage`:

```json
{
  "channelId": "847582",
  "content": "Hello from Fusion"
}
```

---

**### "channelId is required"**

Provide the target channel ID when using `getThreads`.

---

**### Thread ID Is Not Included**

`threadId` is optional and is added to the POST body only when its value is truthy.

**---**

**## Security**

Every request uses OAuth2 Bearer authentication:

```text
Authorization: Bearer <accessToken>
```

For production workflows:

\- Store `accessToken` using protected secrets or expressions

\- Avoid exposing Authorization headers in logs

\- Rotate or revoke compromised access tokens

\- Grant only the Twist permissions required by the workflow

Requests use the HTTPS API base URL:

```text
https://api.twist.com/api/v3
```

**---**

**## Notes**

The default operation is:

```text
getChannels
```

Supported operations are:

```text
getChannels
postMessage
getThreads
getConversations
```

`workspaceId` is required for:

```text
getChannels
getConversations
```

`channelId` is required for:

```text
postMessage
getThreads
```

`content` is required for:

```text
postMessage
```

`threadId` is optional.

The node transforms:

```text
channelId → channel_id
threadId → thread_id
workspaceId → workspace_id
```

GET identifier values are URL-encoded using `encodeURIComponent()`.

POST request bodies are serialized using `JSON.stringify()`.

The node does not:

\- Generate OAuth2 access tokens

\- Refresh expired access tokens

\- Use incoming workflow data directly

\- Retry failed requests

\- Paginate API responses

\- Cache API responses

\- Transform successful API responses

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-14 | Initial release |
