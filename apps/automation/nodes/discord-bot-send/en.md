---
node_id: "discord-bot-send"

title: "Discord Bot Send Message"

description: "Send messages to a Discord channel through the Discord Bot API."

category: "Communication / Discord"

version: "1.0.0"

language: "en"

last_updated: "2026-09-15"

author: "Fusion Team"

tags:

- discord
- discord-bot
- messaging
- channel
- bot-api

related_nodes:

- discord-api
- function
- if

---

**# Discord Bot Send Message**

> **\*\*Category:\*\*** communication-nodes | **\*\*Type:\*\*** Action Node

Send messages to a **\*\*Discord channel\*\*** using the Discord Bot API.

The node sends a `POST` request to Discord API v10. Message content can be configured directly or generated from incoming workflow data.

**### Supported Features**

\- Send a message to a Discord channel

\- Authenticate using a Discord bot token

\- Target a channel using `channelId`

\- Use configured `content`

\- Fall back to incoming string data

\- Serialize non-string incoming data with `JSON.stringify()`

\- Return the created Discord message response

\- Surface Discord HTTP errors

**### Use Cases**

\- Send workflow notifications to Discord

\- Send alerts to a Discord channel

\- Forward workflow text to Discord

\- Post automation results to Discord

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `botToken` | `string` | ✅ Yes | — | Discord bot token. Minimum length: 1. |
| `channelId` | `string` | ✅ Yes | — | Target Discord channel ID. Minimum length: 1. |
| `content` | `string` | ❌ No | — | Message content. If omitted, incoming workflow data is used. |

All parameters support expressions.

**---**

**## Operations**

The node exposes one fixed operation.

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| Send Message | `/api/v10/channels/{channelId}/messages` | `POST` | Send a message to the configured Discord channel. |

Endpoint:

```text
https://discord.com/api/v10/channels/<channelId>/messages
```

There is no `operation` parameter. Every execution sends a message.

**---**

**## Request Body Construction**

If `content` is configured:

```json
{
  "content": "Hello from Fusion"
}
```

If `content` is `null` or `undefined` and incoming data is a string, the incoming string is used.

If incoming data is not a string, the node uses:

```ts
JSON.stringify(data)
```

The exact logic is:

```ts
content ?? (typeof data === "string" ? data : JSON.stringify(data))
```

**### Request Headers**

```text
Authorization: Bot <botToken>
Content-Type: application/json
```

**---**

**## Inputs & Outputs**

**### Inputs**

`handleTick()` receives incoming workflow `data`.

Configured `content` takes priority. Otherwise:

\- String input is used directly.

\- Non-string input is serialized with `JSON.stringify()`.

**### Outputs**

On success:

```json
{
  "status": "sent",
  "body": {
    "content": "Hello from Fusion"
  },
  "message": {}
}
```

| Output | Description |
| ------ | ----------- |
| `status` | Fixed value `"sent"`. |
| `body` | The request body sent to Discord. |
| `message` | Parsed JSON response returned by Discord. |

**---**

**## Configuration Examples**

**### Send Configured Message**

```json
{
  "botToken": "YOUR_DISCORD_BOT_TOKEN",
  "channelId": "YOUR_CHANNEL_ID",
  "content": "Hello from Fusion"
}
```

**### Send Incoming Workflow Data**

```json
{
  "botToken": "YOUR_DISCORD_BOT_TOKEN",
  "channelId": "YOUR_CHANNEL_ID"
}
```

When `content` is omitted, incoming workflow data is used.

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Discord Bot Send Message

\- API Node → Discord Bot Send Message

\- Function → Discord Bot Send Message

\- If → Discord Bot Send Message

\- Monitoring Node → Discord Bot Send Message

**---**

**## Error Handling**

**### Discord Bot API Error**

For a non-success HTTP response, the node throws:

```text
Discord Bot API error: <status> - <response text>
```

The response body is read using:

```ts
await response.text()
```

**### Invalid Configuration**

`botToken` and `channelId` are required strings with `.min(1)` and can fail schema validation before execution.

**### Serialization Error**

When `content` is omitted, non-string incoming data is passed to `JSON.stringify(data)`. Data that cannot be serialized can fail before the API request.

**---**

**## Troubleshooting**

**### Discord API Returns 401**

Verify that `botToken` contains the intended Discord bot token.

The node sends:

```text
Authorization: Bot <botToken>
```

---

**### Discord API Returns 403**

Check that the configured bot can access the target channel and send messages there.

---

**### Message Contains JSON**

No explicit `content` was configured and incoming data was not a string, so the node serialized it with:

```ts
JSON.stringify(data)
```

Configure `content` explicitly if you want a custom message.

---

**### Incoming Data Is Ignored**

Configured `content` takes priority because the node uses:

```ts
content ?? fallback
```

**---**

**## Security**

Authentication uses:

```text
Authorization: Bot <botToken>
```

For production workflows:

\- Store `botToken` using protected secrets or expressions

\- Never commit a real Discord bot token to Git

\- Do not place real tokens in workflow example JSON files

\- Avoid exposing Authorization headers in logs

\- Rotate the token if it is exposed

**---**

**## Notes**

Metadata label:

```text
Discord Bot Send Message
```

Metadata description:

```text
Sends a message to a Discord channel through the Bot API.
```

The node uses:

```text
POST https://discord.com/api/v10/channels/${channelId}/messages
```

Content priority is:

```text
configured content
→ incoming string
→ JSON.stringify(incoming data)
```

The node returns:

```text
status
body
message
```

The node does not:

\- Retrieve messages

\- Edit messages

\- Delete messages

\- Send embeds

\- Add reactions

\- Send direct messages

\- Connect to the Discord Gateway

\- Generate bot tokens

\- Retry failed requests

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-15 | Initial release |
