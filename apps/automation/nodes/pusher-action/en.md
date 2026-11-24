---
node_id: "pusher-action"
title: "Pusher Send"
description: "Send real-time event messages to Pusher Channels with support for single and multi-channel broadcast, dynamic payload expressions, and TLS encryption."
category: "communication"
subcategory: "chat-and-collaboration"
version: "1.0.0"
language: "en"
last_updated: "2026-11-24"
author: "Fusion Team"
tags:
  - pusher
  - websocket
  - real-time
  - messaging
  - broadcast
  - pub-sub
  - notifications
related_nodes:
  - function
  - manual-trigger
  - webhook
---

<!-- SECTION: overview -->
# Pusher Send

> **Category:** Communication & Collaboration&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Pusher Send** node triggers real-time events across Pusher Channels. It enables workflows to push live updates, UI notifications, chat messages, and telemetry payloads directly to connected web, mobile, and desktop clients via WebSockets.

The node supports single-channel publishing, simultaneous multi-channel broadcasting, dynamic template pattern replacement (`data.xxx`), sender socket exclusion (`socketId`), and TLS-secured HTTP API communication.

### Use Cases

- **Live UI Updates:** Broadcast database changes or background job completions directly to frontend clients without polling.
- **Collaborative Apps:** Push live document edits, presence markers, or canvas updates to connected users.
- **Multi-Tenant Alerts:** Broadcast security alerts or announcements across multiple tenant channels simultaneously.
- **IoT & Sensor Streaming:** Publish telemetry alerts or machine threshold warnings to operator monitoring consoles.
- **Chat & Activity Feeds:** Deliver real-time chat messages and activity feed updates to active subscribers.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `appId` | `string` | ✅ Yes | — | Pusher Application ID obtained from your Pusher account dashboard. |
| `key` | `string` | ✅ Yes | — | Pusher Application Key used for client and API authentication. |
| `secret` | `string` | ✅ Yes | — | Pusher Application Secret used to compute HMAC-SHA256 request signatures. |
| `cluster` | `string` | ✅ Yes | — | Geographical cluster hosting the Pusher app (e.g. `eu`, `mt1`, `us2`, `ap1`). |
| `channel` | `string` | ✅ Yes | — | Default target channel name. Must be <= 164 characters and contain only alphanumeric characters, `-`, and `_`. Supports dynamic patterns (e.g. `orders-data.orderId`). |
| `event` | `string` | ✅ Yes | — | Event name to trigger. Cannot exceed 200 characters, contain newlines, or start with reserved prefixes `pusher:` or `pusher_internal:`. |
| `data` | `any` | ❌ No | — | Payload data to send. Can be a JSON Object, Array, String, Number, Boolean, or an expression referencing upstream node outputs. If omitted, uses incoming node input data. |
| `channels` | `array<string>` | ❌ No | — | List of multiple channels to trigger simultaneously (up to 100 channels). If provided and non-empty, overrides the single `channel` field. |
| `socketId` | `string` | ❌ No | — | Socket ID of the sender to exclude from receiving the event (format `<socket_id>.<number>`, e.g. `1234.5678`). Prevents duplicate notifications on emitting clients. |
| `useTLS` | `boolean` | ❌ No | `true` | When `true`, communicates with Pusher via HTTPS/TLS. Strongly recommended for production security. |

---

### Dynamic Data Patterns (`data.xxx`)

The node can dynamically extract values from the incoming trigger or action payload and interpolate them into `channel`, `event`, `data`, `channels`, and `socketId` strings using the `data.<path>` syntax:

- **Channel interpolation:** `tenant-data.tenantId` becomes `tenant-acme_corp` if input data has `{ tenantId: "acme_corp" }`.
- **Nested paths:** `user-data.user.profile.id` traverses nested objects.
- **Object payloads:** If `data` is set to an expression (e.g. `outputs.Function.success`), the resolved JavaScript object is sent directly as structured JSON.

---

### Multi-Channel Broadcast

To broadcast a single event across several channels in a single API call, use the `channels` array parameter:

```json
{
  "channels": [
    "notifications_web",
    "notifications_mobile",
    "audit_log"
  ]
}
```

The node deduplicates channel names, ensures all names comply with Pusher format standards, and validates that the list does not exceed the maximum safety threshold of 100 channels.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data from the preceding node. Can be accessed inside string fields via `data.xxx` or passed as default payload if `data` parameter is left empty. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the event is successfully dispatched to Pusher. |
| `error` | `object` | Emitted when validation fails or Pusher rejects the API request. |

### Output Schema (`success`)

When an event triggers successfully, the `success` output emits an object with execution metadata:

```json
{
  "success": true,
  "channels": [
    "test_fusion",
    "test_canal_2"
  ],
  "event": "user-registered",
  "data": {
    "userId": "user_10",
    "action": "user-registered",
    "name": "Alex",
    "role": "admin"
  },
  "result": {
    "eventIds": {},
    "numberOfSubscribers": 1
  },
  "sentAt": "2026-11-24T14:13:42.813Z"
}
```

### Output Schema (`error`)

If an error occurs (such as invalid credentials or format violations), the node returns a structured error object without throwing uncaught exceptions:

```json
{
  "success": false,
  "error": {
    "type": "pusher_error",
    "message": "Unexpected status code 401",
    "channels": [
      "test_fusion"
    ],
    "event": "user-registered",
    "timestamp": "2026-11-24T14:14:45.752Z"
  }
}
```

> [!IMPORTANT]
> **Secret Redaction:** Any application secret, key, or token detected in error messages or logs is automatically replaced with `[REDACTED]` to prevent credential leaks.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
### Example Workflow

The following workflow demonstrates triggering a real-time event when a user registers. A **Function** node prepares structured user data, and **Pusher Send** broadcasts the event across multiple channels.

```fusion-workflow
src: example.workflow.json
title: Trigger Real-time Event to Pusher Channels
```

### Step-by-Step Walkthrough

1. **Manual Trigger:** Triggers the workflow execution manually or on demand.
2. **Function Node:** Prepares the user object:
   ```javascript
   return {
     userId: "user_10",
     action: "user-registered",
     name: "Alex",
     role: "admin"
   };
   ```
3. **Pusher Send Node:**
   - Evaluates dynamic payload from `outputs.Function.success`.
   - Sends the `user-registered` event to channels `notifications` and `audit-log`.
   - Connected WebSocket subscribers receive the message instantly with sub-second latency.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Unexpected status code 401`

**Cause:** Invalid `appId`, `key`, or `secret`. The HMAC-SHA256 signature generated does not match the credentials on Pusher servers.

**Solution:** Verify your Pusher application credentials in the Pusher Dashboard. Ensure no extra whitespace or trailing newlines exist in the fields.

#### `Invalid event name`

**Cause:** The event name starts with `pusher:` or `pusher_internal:`, exceeds 200 characters, or contains newline characters (`\r`, `\n`).

**Solution:** Use standard event names such as `order-created`, `notification`, or `user-updated`. Prefixes beginning with `pusher:` are reserved for client protocol events.

#### `Invalid channel name`

**Cause:** The channel name contains unsupported punctuation (e.g. spaces, `#`, `!`) or exceeds 164 characters.

**Solution:** Ensure channel names only contain alphanumeric characters, `-`, and `_`. For private or presence channels, ensure correct prefixes (`private-` or `presence-`).

#### `Cannot trigger more than 100 channels simultaneously`

**Cause:** The `channels` array contains more than 100 entries.

**Solution:** Pusher batch triggers limit single requests to 100 target channels. Split your recipients into batches of 100 or less.

#### `Payload size exceeds safety limit`

**Cause:** The serialized JSON payload exceeds 100 KB.

**Solution:** Pusher is designed for lightweight messaging (standard limit 10 KB, max 100 KB). For large datasets, send an event with a resource ID or URL pointing to the full data in a database or object storage.

### Error Codes

| Error Type | Cause | Resolution |
|------------|-------|------------|
| `invalid_channel_name` | Empty channel name, illegal characters, or length > 164 | Provide a valid alphanumeric channel name |
| `invalid_event_name` | Event exceeds 200 chars or uses reserved `pusher:` prefix | Rename event to a custom application event |
| `invalid_socket_id` | Socket ID does not match `<id>.<number>` format | Use exact socket ID from client connection event |
| `invalid_data_type` | Payload is a function, symbol, or bigint | Pass JSON-serializable objects, arrays, or strings |
| `too_many_channels` | Channels array has more than 100 items | Reduce target channel count to <= 100 |
| `pusher_error` | Pusher API rejected request (401, 403, 404, 500) | Check credentials, cluster location, and account quota |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](/docs/function) – Prepare or transform dynamic event payloads
- [Manual Trigger](/docs/manual-trigger) – Manually trigger workflows for testing and automation
- [Webhook](/docs/webhook) – Trigger workflows from incoming HTTP webhooks

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-24 | Initial release of Pusher Send action node documentation |

<!-- /SECTION: changelog -->
