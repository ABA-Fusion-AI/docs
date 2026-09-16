---
node_id: "discord-webhook-send"

title: "Discord Webhook Send"

description: "Send messages to a Discord channel using an incoming webhook URL."

category: "Communication / Discord"

version: "1.0.0"

language: "en"

last_updated: "2026-09-15"

author: "Fusion Team"

tags:

- discord

- webhook

- messaging

- embeds

- avatar

- notification

related_nodes:

- discord-api

- discord-bot-send

- function

- if

---

**# Discord Webhook Send**

> **\*\*Category:\*\*** communication-nodes | **\*\*Type:\*\*** Action Node

Send messages to a **\*\*Discord channel\*\*** using an incoming Discord webhook URL.

The **\*\*Discord Webhook Send\*\*** node supports text content, custom usernames, preset or custom avatar URLs, Discord embeds, and automatic fallback to incoming workflow data when explicit content is not configured.

**### Supported Features**

\- Send messages through a Discord incoming webhook

\- Send configured text content

\- Use incoming workflow data as fallback content

\- Set a custom webhook username

\- Select from predefined avatar presets

\- Use a custom avatar URL

\- Send Discord embeds

\- Serialize non-string incoming data with `JSON.stringify()`

\- Return the exact request body after a successful send

\- Surface Discord webhook HTTP errors

**### Use Cases**

\- Send workflow notifications to Discord

\- Send success or error alerts with different avatars

\- Send AI-generated results to Discord

\- Post structured Discord embeds

\- Customize webhook identity per workflow

\- Forward incoming workflow data to a Discord channel

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `webhookUrl` | `string` | ✅ Yes | — | Discord incoming webhook URL. Minimum length: 1. |
| `content` | `string` | ❌ No | — | Message content. Falls back to incoming workflow data when omitted. |
| `username` | `string` | ❌ No | — | Optional username used by the webhook message. |
| `avatarPreset` | `enum` | ❌ No | `"BOT"` | Avatar preset: `BOT`, `AI`, `WORKFLOW`, `NOTIFICATION`, `SUCCESS`, `ERROR`, or `CUSTOM`. |
| `customAvatarUrl` | `string` | ❌ No | — | Custom avatar URL used when `avatarPreset` is `CUSTOM`. |
| `embeds` | `array<any>` | ❌ No | — | Optional array of Discord embed objects. |

`webhookUrl`, `content`, `username`, and `customAvatarUrl` support expressions.

**### Avatar Presets**

| Preset | Behavior |
| ------ | -------- |
| `BOT` | Uses the predefined BOT avatar URL. |
| `AI` | Uses the predefined AI avatar URL. |
| `WORKFLOW` | Uses the predefined WORKFLOW avatar URL. |
| `NOTIFICATION` | Uses the predefined NOTIFICATION avatar URL. |
| `SUCCESS` | Uses the predefined SUCCESS avatar URL. |
| `ERROR` | Uses the predefined ERROR avatar URL. |
| `CUSTOM` | Uses the value configured in `customAvatarUrl`. |

The default preset is:

```text
BOT
```

**---**

**## Operations**

The node exposes one fixed operation.

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| Send Webhook Message | `webhookUrl` | `POST` | Send content, username, avatar and embeds to the configured Discord webhook. |

Unlike the Discord Bot API node, the endpoint is not hardcoded. The request is sent directly to the configured `webhookUrl`.

**---**

**## Request Body Construction**

The node determines the avatar URL first.

For `CUSTOM`:

```ts
avatarUrl = customAvatarUrl
```

For any predefined preset:

```ts
avatarUrl = avatarUrls[avatarPreset]
```

The request body is then constructed as:

```json
{
  "content": "Hello from Fusion",
  "username": "Fusion Bot",
  "avatar_url": "https://example.com/avatar.png",
  "embeds": []
}
```

The implementation uses:

```ts
const body = {
  content: content ?? (typeof data === "string" ? data : JSON.stringify(data)),
  username,
  avatar_url: avatarUrl,
  embeds,
};
```

**### Content Fallback**

Content priority is:

```text
configured content
→ incoming string
→ JSON.stringify(incoming data)
```

The fallback occurs only when configured `content` is `null` or `undefined`.

**### Request Headers**

```text
Content-Type: application/json
```

No Authorization header is added because authentication information is contained in the configured webhook URL.

**---**

**## Inputs & Outputs**

**### Inputs**

The node receives incoming workflow data through:

```text
data
```

If `content` is configured, incoming data is not used for the message content.

If `content` is not configured:

\- String data is used directly.

\- Non-string data is serialized using `JSON.stringify(data)`.

**### Outputs**

After a successful request, the node returns:

```json
{
  "status": "sent",
  "body": {
    "content": "Hello from Fusion",
    "username": "Fusion Bot",
    "avatar_url": "https://example.com/avatar.png",
    "embeds": []
  }
}
```

| Output | Description |
| ------ | ----------- |
| `status` | Fixed value `"sent"`. |
| `body` | The exact body object constructed by the node. |

The node does not parse or return the successful Discord response body.

**---**

**## Configuration Examples**

**### Basic Webhook Message**

```json
{
  "webhookUrl": "YOUR_DISCORD_WEBHOOK_URL",
  "content": "Hello from Fusion"
}
```

**### Custom Username**

```json
{
  "webhookUrl": "YOUR_DISCORD_WEBHOOK_URL",
  "content": "Workflow completed",
  "username": "Fusion Workflow"
}
```

**### Success Avatar**

```json
{
  "webhookUrl": "YOUR_DISCORD_WEBHOOK_URL",
  "content": "Operation completed successfully",
  "avatarPreset": "SUCCESS"
}
```

**### Error Avatar**

```json
{
  "webhookUrl": "YOUR_DISCORD_WEBHOOK_URL",
  "content": "Workflow execution failed",
  "avatarPreset": "ERROR"
}
```

**### Custom Avatar**

```json
{
  "webhookUrl": "YOUR_DISCORD_WEBHOOK_URL",
  "content": "Custom notification",
  "avatarPreset": "CUSTOM",
  "customAvatarUrl": "https://example.com/avatar.png"
}
```

**### Message With Embeds**

```json
{
  "webhookUrl": "YOUR_DISCORD_WEBHOOK_URL",
  "content": "Workflow result",
  "avatarPreset": "WORKFLOW",
  "embeds": [
    {
      "title": "Workflow Status",
      "description": "Execution completed."
    }
  ]
}
```

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Discord Webhook Send

\- Function → Discord Webhook Send

\- If Success → Discord Webhook Send (`SUCCESS`)

\- If Error → Discord Webhook Send (`ERROR`)

\- AI Node → Discord Webhook Send (`AI`)

\- Workflow Result → Discord Webhook Send (`WORKFLOW`)

**---**

**## Error Handling**

**### Discord Webhook Error**

If Discord returns a non-success HTTP status, the node reads the response body and throws:

```text
Discord Webhook error: <status> - <response text>
```

The implementation uses:

```ts
await response.text()
```

to include the Discord response text in the error.

**### Invalid Webhook URL Configuration**

`webhookUrl` is a required string with `.min(1)`.

An empty value can therefore fail schema validation before the HTTP request.

The schema validates that the value is a non-empty string; it does not use a URL-specific validator.

**### Content Serialization Error**

If `content` is omitted and incoming data is not a string, the node calls:

```ts
JSON.stringify(data)
```

Values that cannot be serialized can cause execution to fail before the request is sent.

**---**

**## Troubleshooting**

**### Discord Webhook Returns an HTTP Error**

Verify that the configured `webhookUrl` is the intended Discord incoming webhook URL.

The node sends a `POST` request directly to this value.

---

**### Custom Avatar Is Missing**

When:

```text
avatarPreset = CUSTOM
```

the node uses:

```text
customAvatarUrl
```

If `customAvatarUrl` is undefined, `avatar_url` is also undefined in the body object.

---

**### Wrong Avatar Is Displayed**

Check the configured `avatarPreset`.

Supported values are:

```text
BOT
AI
WORKFLOW
NOTIFICATION
SUCCESS
ERROR
CUSTOM
```

---

**### Incoming Data Is Ignored**

Configured `content` takes priority because the node uses:

```ts
content ?? fallback
```

Remove `content` if you want the node to use incoming workflow data.

---

**### Message Contains Serialized JSON**

No explicit `content` was configured and incoming data was not a string.

The node therefore uses:

```ts
JSON.stringify(data)
```

**---**

**## Security**

The Discord webhook URL acts as a credential because it identifies and authorizes use of the webhook.

For production workflows:

\- Store `webhookUrl` using protected secrets or expressions

\- Never commit a real Discord webhook URL to Git

\- Do not include real webhook URLs in workflow example files

\- Avoid exposing webhook URLs in logs

\- Replace the webhook if its URL is exposed

The node sends the webhook request using the URL supplied in configuration.

**---**

**## Notes**

The node metadata label is:

```text
Discord Webhook Send
```

The node description is:

```text
Sends a message to a Discord channel using an incoming webhook URL.
```

The default avatar preset is:

```text
BOT
```

Supported avatar presets are:

```text
BOT
AI
WORKFLOW
NOTIFICATION
SUCCESS
ERROR
CUSTOM
```

Preset avatar URLs are stored in the node's `avatarUrls` constant.

When `CUSTOM` is selected:

```text
customAvatarUrl
```

is used instead.

The request body fields are:

```text
content
username
avatar_url
embeds
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
```

The fixed success status is:

```text
sent
```

The node does not:

\- Use a Discord bot token

\- Add an Authorization header

\- Retrieve Discord messages

\- Edit or delete messages

\- Parse the successful Discord response body

\- Validate the structure of individual embed objects

\- Validate `customAvatarUrl` as a URL

\- Retry failed webhook requests

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-15 | Initial release |
