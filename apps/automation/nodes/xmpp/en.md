---
node_id: "xmpp"
title: "XMPP"
description: "Send XMPP (Jabber) chat messages over WebSocket or TCP. Recipient and message can come from the parameters or from the incoming payload."
category: "Communication / Messaging"
subcategory: "Chat"
version: "1.0.0"
language: "en"
last_updated: "2026-11-17"
author: "Fusion Team"
tags:
  - xmpp
  - jabber
  - chat
  - messaging
  - websocket
  - notifications
related_nodes:
  - smtp
  - discord-api
  - pender-sms
  - infobip-sms
---

<!-- SECTION: overview -->
# XMPP

> **Category:** Communication / Messaging &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Send **XMPP (Jabber)** chat messages from a Fusion workflow. The node logs in to an XMPP server with an existing account, then sends a `chat` message to a recipient JID (e.g. `bob@example.com`). It supports **WebSocket** (`wss://`) and **TCP** (`xmpp://`) connections.

The recipient and the message can be set in the node parameters or sent by the previous node as `{ "to": "...", "message": "..." }`.

### Use Cases

- **Alerts & notifications:** Send monitoring alerts or workflow results to a team or an on-call engineer on an XMPP server.
- **Self-hosted messaging:** Notify users on a private, on-premise chat server (Prosody, ejabberd, Openfire).
- **Bots & integrations:** Forward events from other systems (forms, CRM, tickets) to XMPP users.
- **IoT messaging:** Push device or sensor events to XMPP accounts.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `service` | `string` | Yes | — | Server URL: `wss://host/xmpp-websocket` (WebSocket) or `xmpp://host:5222` (TCP). |
| `domain` | `string` | No | — | XMPP domain of the account (e.g. `example.com`). Needed when it differs from the `service` host (proxy, tunnel, localhost). |
| `resource` | `string` | No | — | XMPP resource of the session (e.g. `fusion`). |
| `username` | `string` | Yes | — | Account username, without the domain (e.g. `alice`). |
| `password` | `string` | Yes | — | Account password. Supports expressions. |
| `to` | `string` | No* | — | Recipient JID (e.g. `bob@example.com`). |
| `message` | `string` | No | — | Text of the message. |

\* `to` must be provided either in the parameters or in the incoming payload.

### Recipient & Message Resolution

| Value | Priority order |
|-------|----------------|
| Recipient | 1. `to` field of the incoming payload → 2. `to` parameter |
| Message | 1. `message` field of the incoming payload → 2. `message` parameter → 3. the input itself (a string as-is, an object as JSON text) |

> Values from the payload take priority over the parameters.

### Service URL

| Connection | Format | Example |
|------------|--------|---------|
| WebSocket | `wss://<host>/<path>` | `wss://chat.example.com/xmpp-websocket` |
| TCP | `xmpp://<host>:5222` | `xmpp://chat.example.com:5222` |

The WebSocket URL of a server is usually listed at `https://<host>/.well-known/host-meta`. Use WebSocket when the server is behind HTTPS or a reverse proxy.

### Authentication

The node uses an existing XMPP account (username + password) on the target server. It does not create accounts. Create the sending account on your server (e.g. `prosodyctl register alice example.com <password>` on Prosody) or with any XMPP client.

### Connection Reuse

The connection is kept open between runs and reused as long as `service`, `domain`, `resource`, `username` and `password` do not change. Changing one of them opens a new connection. A connection attempt that takes more than **10 seconds** fails.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` / `string` | Triggers the node. Can carry `to` and `message`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Confirmation of the sent message. |
| `error` | `Error` | Emitted on missing recipient, connection failure, authentication failure or send failure. |

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `true` when the message was sent. |
| `to` | `string` | Recipient JID. |
| `message` | `string` | Text that was sent. |
| `sent` | `boolean` | `true` when the message was sent. |

### Output Example

```json
{
  "success": true,
  "to": "bob@localhost",
  "message": "Test Fusion XMPP",
  "sent": true
}
```

> `sent: true` means the server accepted the message. If the recipient is offline, delivery depends on the server (most servers store offline messages).

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Send an XMPP message
```

### How it flows

1. **Manual Trigger:** Starts the workflow (optionally with `{ "to": "...", "message": "..." }`).
2. **XMPP Node:** Connects to the XMPP server and sends the chat message.
3. **Log Node:** Displays the send confirmation.

### Example Configuration

```json
{
  "service": "wss://chat.example.com/xmpp-websocket",
  "domain": "example.com",
  "resource": "fusion",
  "username": "alerts",
  "password": "{{ secrets.XMPP_PASSWORD }}",
  "to": "oncall@example.com",
  "message": "Workflow {{ input.name }} finished with status {{ input.status }}"
}
```

### Example Payload

Leave `to` and `message` empty in the node and send them from the previous node:

```json
{
  "to": "bob@example.com",
  "message": "Hello from Fusion"
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the XMPP `password` in Fusion's **Secrets** vault. Do not commit hardcoded passwords into version control.

- Use a dedicated account for automations, not a personal account.
- Prefer encrypted connections (`wss://`, or TCP with STARTTLS enabled on the server).
- Restrict which workflows can send messages to avoid spam from compromised inputs.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Recipient (to) is required`
- **Cause:** No `to` in the parameters nor in the incoming payload.
- **Solution:** Fill in `to`, or send `{ "to": "user@domain" }` from the previous node.

#### `Failed to connect to XMPP server: timed out after 10s (service: ...)`
- **Cause:** The server cannot be reached at the `service` URL (wrong URL or path, firewall, tunnel closed).
- **Solution:** Check the URL; for WebSocket, open it with `https://` in a browser to see if the server answers.

#### `Failed to connect to XMPP server: not-authorized`
- **Cause:** Wrong username or password, or the account does not exist on this domain.
- **Solution:** Check the credentials, and set `domain` if the server's XMPP domain differs from the `service` host.

#### `Failed to connect to XMPP server: ... host-unknown`
- **Cause:** The `domain` does not match a virtual host of the server.
- **Solution:** Set `domain` to the XMPP domain configured on the server.

#### `Failed to send XMPP message: ...`
- **Cause:** The connection dropped while sending.
- **Solution:** Run the workflow again; the node reconnects automatically.

#### Message sent but not received
- **Cause:** The recipient JID is wrong, or the server blocks messages from contacts not in the recipient's roster.
- **Solution:** Check the JID and add the sender to the recipient's contacts.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-17 | Initial documentation: parameter reference, payload resolution, connection reuse, output schema, and workflow integration |

<!-- /SECTION: changelog -->
