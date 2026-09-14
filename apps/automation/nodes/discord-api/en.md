---
node_id: "discord-api"

title: "Discord API"

description: "Get Discord bot user information using the Discord API."

category: "Communication / Discord"

version: "1.0.0"

language: "en"

last_updated: "2026-09-14"

author: "Fusion Team"

tags:

- discord
- discord-api
- bot
- user
- profile
- communication
- api

related_nodes:

- http-request
- function
- if

---

**# Discord API**

> **\*\*Category:\*\*** communication-nodes | **\*\*Type:\*\*** Action Node

Retrieve information about the authenticated **\*\*Discord bot user\*\*** using the Discord API.

The **\*\*Discord API\*\*** node sends a request to Discord API v10 using the configured bot token and retrieves the user associated with that token.

The node uses the Discord `/users/@me` endpoint and returns a normalized object containing selected properties from the Discord response.

**### Supported Features**

\- Authenticate using a Discord bot token

\- Retrieve the authenticated Discord bot user

\- Return user ID and username

\- Return discriminator and global display name

\- Return avatar and banner identifiers

\- Return bot and system flags

\- Return MFA status

\- Return accent color and locale

\- Return verification and email values when present

\- Return Discord flags and premium type

\- Normalize selected Discord response fields

\- Surface Discord HTTP errors

**### Use Cases**

\- Validate a Discord bot token

\- Retrieve the identity of the configured Discord bot

\- Display Discord bot profile information

\- Confirm which bot account is connected to a workflow

\- Retrieve bot metadata for downstream processing

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `botToken` | `string` | ❌ No* | `""` | Discord bot token used for authentication. Required at runtime. |

Although `botToken` is optional in the schema, the implementation requires a non-empty value when executed.

**---**

**## Operations**

The node exposes one fixed operation.

| Operation | Endpoint | Method | Description |
| --------- | -------- | ------ | ----------- |
| Get User Info | `/api/v10/users/@me` | `GET` | Retrieve information about the authenticated Discord bot user. |

The complete endpoint is:

```text
https://discord.com/api/v10/users/@me
```

The node does not expose an `operation` field. Every execution performs this lookup.

**---**

**## Request Body Construction**

The node performs a `GET` request and does not send a request body.

```text
GET https://discord.com/api/v10/users/@me
```

The authentication header is:

```text
Authorization: Bot <botToken>
```

No `Content-Type` or `Accept` header is explicitly configured.

**---**

**## Inputs & Outputs**

**### Inputs**

The node receives `data` and `_session` in `handleTick()`, but neither is used.

The bot token is read from node configuration.

**### Outputs**

The node returns:

| Output | Description |
| ------ | ----------- |
| `success` | `true` after a successful request. |
| `id` | Discord user ID. |
| `username` | Discord username. |
| `discriminator` | Discord discriminator. |
| `global_name` | Global display name. |
| `avatar` | Avatar identifier. |
| `bot` | Bot flag. |
| `system` | System-user flag. |
| `mfa_enabled` | MFA-enabled value. |
| `banner` | Banner identifier. |
| `accent_color` | Accent color. |
| `locale` | Locale. |
| `verified` | Verification value. |
| `email` | Email value. |
| `flags` | Discord user flags. |
| `premium_type` | Discord premium type. |
| `public_flags` | Discord public flags. |

Properties are copied directly from the parsed Discord response.

**### Output Example**

```json
{
  "success": true,
  "id": "123456789012345678",
  "username": "fusion-bot",
  "discriminator": "0",
  "global_name": "Fusion Bot",
  "avatar": null,
  "bot": true,
  "system": false,
  "mfa_enabled": false,
  "banner": null,
  "accent_color": null,
  "locale": "en-US",
  "verified": true,
  "email": null,
  "flags": 0,
  "premium_type": 0,
  "public_flags": 0
}
```

The exact values depend on the Discord API response.

**---**

**## Configuration Examples**

**### Get Discord Bot User Information**

```json
{
  "botToken": "YOUR_DISCORD_BOT_TOKEN"
}
```

No additional operation configuration is required.

**---**

**## Workflow Integration**

**### Common Patterns**

\- Trigger → Discord API — retrieve the configured bot identity

\- Discord API → Function — process returned user information

\- Discord API → If — branch based on returned metadata

\- Discord API → Logging — record the configured bot identity

**---**

**## Error Handling**

**### Missing Bot Token**

The helper throws:

```text
Discord bot token is required
```

Because the helper is called inside the `handleTick()` `try` block, the final error becomes:

```text
Discord API request failed: Discord bot token is required
```

**### Discord API Error**

For a non-success HTTP status:

```text
Discord API error: <status>
```

The final wrapped error becomes:

```text
Discord API request failed: Discord API error: <status>
```

**### General Request Error**

Other errors are wrapped as:

```text
Discord API request failed: <error message>
```

**---**

**## Troubleshooting**

**### "Discord API request failed: Discord bot token is required"**

**\*\*Cause\*\***

`botToken` is empty or missing.

**\*\*Solution\*\***

Provide the Discord bot token.

---

**### "Discord API request failed: Discord API error: 401"**

**\*\*Cause\*\***

Discord returned HTTP status `401`.

**\*\*Solution\*\***

Verify the configured bot token.

---

**### Returned Properties Are Missing**

The node copies selected properties directly from the API response without fallback values.

Downstream workflow logic should account for properties that are not present.

**---**

**## Security**

The bot token is sent using:

```text
Authorization: Bot <botToken>
```

For production workflows:

\- Store the bot token using protected secrets or expressions

\- Do not hardcode bot tokens in reusable workflows

\- Avoid exposing Authorization headers in logs

\- Rotate the token if it is exposed

The request uses:

```text
https://discord.com/api/v10/users/@me
```

**---**

**## Notes**

The node metadata label is:

```text
Discord API
```

The node uses the fixed endpoint:

```text
https://discord.com/api/v10/users/@me
```

The schema defines `botToken` as optional with an empty-string default, but execution requires a non-empty token.

The node returns a custom object containing selected Discord user properties rather than returning the complete raw API response.

The node does not:

\- Connect to the Discord Gateway

\- Use `discord.js`

\- Send messages

\- Retrieve guilds or channels

\- Retrieve messages

\- Manage members

\- Create webhooks

\- Generate bot tokens

\- Retry failed requests

\- Use incoming workflow data

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-09-14 | Initial release |
