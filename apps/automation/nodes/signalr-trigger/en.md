---
node_id: "signalr-trigger"
title: "SignalR Trigger"
description: "Listen for real-time messages from Azure SignalR Service — triggers when registered events are received on a hub connection"
category: "integrations"
subcategory: "messaging"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - signalr
  - azure
  - real-time
  - websocket
  - trigger
related_nodes:
  - signalr-action
  - event-hub-receive
  - webhook-trigger
---

<!-- SECTION: header -->
# SignalR Trigger

> **Category:** Integrations | **Type:** Trigger Node

Connect to an Azure SignalR hub as a client and trigger a workflow whenever a registered event is received. Supports automatic reconnection with exponential backoff and graceful pause/resume.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **SignalR Trigger** node establishes a persistent WebSocket connection to an Azure SignalR hub using the `@microsoft/signalr` client SDK. It registers one or more named event handlers and emits a workflow trigger payload whenever any of those events fires.

Authentication uses the **AccessKey** from the connection string to generate a short-lived HS256 JWT (1-hour TTL) for the client connection. This is distinct from the [SignalR Action](./signalr-action.md) node, which uses Entra ID app credentials for REST management calls.

### Key Features

- **Event Subscription:** Listen to any number of named hub events simultaneously
- **Auto-Reconnect:** Exponential backoff — 0ms → 2s → 10s → 30s between retries
- **Connection Events:** Emits on initial connect, reconnect, and disconnect
- **Pause / Resume:** Deregisters listeners on pause; re-registers on resume without dropping the connection
- **10-Second Timeout:** Connection setup fails fast if the hub is unreachable
- **Payload Normalisation:** Single-argument events expose `payload` directly; multi-argument events expose the full `arguments` array

### Use Cases

- Trigger downstream automation whenever a frontend client broadcasts an event (e.g. a form submission, a button click)
- React to IoT device telemetry pushed through SignalR from a backend aggregator
- Fan out processing logic when a collaborative document is saved by any user
- Feed real-time chat messages into a moderation or sentiment-analysis pipeline

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `connectionString` | `string` | ✅ Yes | — | Azure SignalR connection string — must contain `Endpoint` and `AccessKey` |
| `hub` | `string` | ✅ Yes | `default` | Hub name to connect to (e.g. `chat`, `orders`, `dashboard`) |
| `events` | `string[]` | ✅ Yes | — | One or more event names to listen for (e.g. `["orderUpdate", "priceChange"]`) |
| `userId` | `string` | ✅ Yes | — | User ID embedded in the JWT — used to identify this client connection |

### Authentication Details

The node extracts `Endpoint` and `AccessKey` from the connection string and generates a JWT with the following claims:

| Claim | Value |
|-------|-------|
| `aud` | `https://<endpoint>/client/?hub=<hub>` |
| `nameid` | `userId` parameter |
| `exp` | Current time + 3600 seconds |

The token is signed with HMAC-SHA256 using the `AccessKey`. It is refreshed on each reconnect.

### Reconnect Strategy

| Retry | Delay |
|-------|-------|
| 1st | 0ms (immediate) |
| 2nd | 2 000ms |
| 3rd | 10 000ms |
| 4th+ | 30 000ms |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when an event fires or a connection lifecycle event occurs |
| `error` | `object` | Emitted when the connection closes unexpectedly |

### Output Schema — Event Received

Emitted every time a registered event fires:

```json
{
  "session": {
    "event": "orderUpdate",
    "connectionId": "conn_abc123",
    "userId": "workflow-bot",
    "hub": "orders"
  },
  "data": {
    "event": "orderUpdate",
    "arguments": [{ "orderId": "ORD-999", "status": "shipped" }],
    "receivedAt": "2026-03-10T15:00:00.000Z",
    "payload": { "orderId": "ORD-999", "status": "shipped" }
  }
}
```

> `payload` is a convenience field: equal to `arguments[0]` when the event has a single argument, or the full `arguments` array when there are multiple.

### Output Schema — Client Connected

Emitted once after the initial connection succeeds:

```json
{
  "session": {
    "connectionId": "conn_abc123",
    "userId": "workflow-bot",
    "state": "Connected"
  },
  "data": {
    "type": "client_connected",
    "clientInfo": {
      "connectionId": "conn_abc123",
      "userId": "workflow-bot",
      "state": "Connected",
      "connectedAt": "2026-03-10T14:59:55.000Z"
    },
    "message": "SignalR client successfully connected and ready to receive events",
    "timestamp": "2026-03-10T14:59:55.100Z"
  }
}
```

### Output Schema — Reconnected

Emitted after an automatic reconnect succeeds:

```json
{
  "session": {
    "connectionId": "conn_xyz789",
    "userId": "workflow-bot",
    "hub": "orders"
  },
  "data": {
    "type": "reconnected",
    "connectionId": "conn_xyz789",
    "message": "SignalR client successfully reconnected",
    "timestamp": "2026-03-10T15:05:00.000Z"
  }
}
```

### Output Schema — Connection Closed (Error)

Emitted to the `error` output if the connection closes with an error:

```json
{
  "session": {},
  "data": {
    "type": "connection_closed",
    "message": "The connection was stopped during negotiation.",
    "timestamp": "2026-03-10T15:10:00.000Z"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Listen for Order Events

Subscribe to `orderCreated` and `orderUpdated` events from the `orders` hub.

**Configuration:**
```json
{
  "connectionString": "Endpoint=https://my-signalr.service.signalr.net;AccessKey=abc123;Version=1.0;",
  "hub": "orders",
  "events": ["orderCreated", "orderUpdated"],
  "userId": "fusion-workflow-bot"
}
```

**Triggered output when `orderCreated` fires:**
```json
{
  "data": {
    "event": "orderCreated",
    "arguments": [{ "orderId": "ORD-1001", "total": 249.99, "customer": "john@example.com" }],
    "payload": { "orderId": "ORD-1001", "total": 249.99, "customer": "john@example.com" },
    "receivedAt": "2026-03-10T15:00:00.000Z"
  }
}
```

---

### Example: Monitor a Dashboard Hub

Listen to a single `dashboardRefresh` event and fan out into multiple downstream processors.

**Configuration:**
```json
{
  "connectionString": "Endpoint=https://my-signalr.service.signalr.net;AccessKey=abc123;Version=1.0;",
  "hub": "dashboard",
  "events": ["dashboardRefresh"],
  "userId": "processor-01"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: SignalR Event → Jira Issue → Slack Notification

When a client emits a `reportBug` event on the `support` hub, automatically create a Jira issue and notify the team on Slack.

```json
{
  "nodes": [
    {
      "id": "signalr-listener",
      "type": "signalr-trigger",
      "config": {
        "connectionString": "Endpoint=https://my-signalr.service.signalr.net;AccessKey=...;Version=1.0;",
        "hub": "support",
        "events": ["reportBug"],
        "userId": "triage-bot"
      }
    },
    {
      "id": "create-jira-issue",
      "type": "jira-action",
      "config": {
        "operation": "createIssue",
        "projectKey": "BUGS",
        "issueData": {
          "summary": "{{input.data.payload.title}}",
          "description": "{{input.data.payload.description}}",
          "issuetype": { "name": "Bug" },
          "priority": { "name": "High" }
        }
      }
    },
    {
      "id": "notify-slack",
      "type": "slack-action",
      "config": {
        "operation": "message",
        "messageOperation": "send",
        "messageChannelId": "C08XXXXXXXX",
        "text": "🐛 New bug reported via SignalR: *{{input.data.payload.title}}* → {{input.key}}"
      }
    }
  ]
}
```

### Common Patterns

- **Event → Transform → Store:** SignalR event → Function (normalize) → MongoDB (persist)
- **Multi-event Router:** Listen to 3+ events → Function (switch on `data.event`) → different branches
- **Real-time to Async:** SignalR receive → Kafka publish (bridge real-time to event streaming)
- **Monitoring Loop:** Reconnected event → Slack notify (alert when SignalR drops and recovers)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Invalid SignalR connection string: missing Endpoint or AccessKey`

**Cause:** The connection string is malformed or only contains the `Endpoint`.

**Solution:** Copy the full connection string from **Azure Portal → SignalR Service → Keys**. It must contain both `Endpoint=` and `AccessKey=`.

#### `Failed to connect to SignalR: Connection timeout`

**Cause:** The hub endpoint is unreachable or the hub name is wrong. Connection times out after 10 seconds.

**Solution:** Verify the hub name matches the one used by your SignalR server. Check network access and firewall rules.

#### No events received after connecting

**Cause:** The event names in `events` don't match what the server is calling `Clients.All.SendAsync("eventName", ...)` with.

**Solution:** Confirm event names are case-sensitive exact matches. Check your SignalR server code for the exact string used in `SendAsync`.

#### `SignalR connection not initialized`

**Cause:** `run()` was called before `setup()` completed.

**Solution:** This is handled internally by the framework. If it occurs, check for errors in the `setup()` phase (e.g. invalid connection string).

### Error Codes

| Source | Error | Solution |
|--------|-------|----------|
| JWT generation | `Invalid AccessKey` | Verify the `AccessKey` in the connection string |
| Hub connection | `Connection timeout` | Check hub name and network access |
| Event handling | `TypeError in listener` | Validate event argument types in downstream function nodes |
| Auto-reconnect | Repeated reconnect warnings | Check if SignalR service is under load or restarting |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [SignalR Action](./signalr-action.md) - Send messages and manage connections/groups via the REST API
- [Event Hub Receive](./eventhub-receive.md) - Receive high-throughput events from Azure Event Hubs
- [Kafka Trigger](./kafka-trigger.md) - Consume events from Apache Kafka topics

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
