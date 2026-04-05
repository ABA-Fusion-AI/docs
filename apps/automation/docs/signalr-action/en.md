---
node_id: "signalr-action"
title: "SignalR Action"
description: "Manage Azure SignalR connections, groups, users, and broadcast real-time messages via the SignalR REST API with Entra authentication"
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
  - messaging
  - action
related_nodes:
  - signalr-trigger
  - event-hub-send
  - slack-action
  - webhook-trigger
---

<!-- SECTION: header -->
# SignalR Action

> **Category:** Integrations | **Type:** Action Node

Perform REST management operations on Azure SignalR Service — check and close connections, manage group memberships, and send real-time messages to specific connections, groups, users, or all connected clients.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **SignalR Action** node authenticates to Azure SignalR using **Azure Entra ID** (formerly Azure Active Directory) via a client credentials flow (`ClientSecretCredential`). It acquires a Bearer token scoped to `https://signalr.azure.com/.default` and calls the SignalR REST Management API v1.

This node is the **server-side push** counterpart to the [SignalR Trigger](./signalr-trigger.md) receiver. Use it to send events from your workflow into SignalR clients in real time.

### Key Features

- **Entra Auth:** Secure app-registration-based authentication — no shared keys on the action side
- **13 Operations:** Full coverage of connections, groups, users, and broadcast
- **Payload Targeting:** Send a named method invocation (`target` + `arguments`) to any scope
- **Expression Support:** `connectionId`, `group`, and `user` accept dynamic expressions from upstream nodes
- **Conditional Fields:** Parameters appear only when required by the selected operation

### Use Cases

- Broadcast a live dashboard update to all connected clients after a data pipeline completes
- Push a personalised notification to a specific user's connection
- Add or remove a user from a SignalR group based on a role change event
- Close a stale or unauthorized connection from an automation workflow
- Check whether a connection is still alive before sending a critical message

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `connectionString` | `string` | ✅ Yes | Azure SignalR connection string (e.g. `Endpoint=https://...;AccessKey=...;Version=1.0;`) |
| `tenantId` | `string` | ✅ Yes | Azure Active Directory tenant ID |
| `clientId` | `string` | ✅ Yes | App registration client ID |
| `clientSecret` | `string` | ✅ Yes | App registration client secret |
| `hub` | `string` | ✅ Yes | SignalR hub name (e.g. `chat`, `notifications`) |

> The `Endpoint` value is extracted from the connection string automatically. `tenantId`, `clientId`, and `clientSecret` are used to generate a short-lived Entra Bearer token.

### Operation Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Operation to execute (see below) |
| `connectionId` | `string` | Conditional | Target connection ID — required for connection-specific operations |
| `group` | `string` | Conditional | Target group name — required for group operations |
| `user` | `string` | Conditional | Target user ID — required for user operations |
| `payload` | `object` | Conditional | Message payload for send operations (see structure below) |

### `payload` Structure

Used by `sendToConnection`, `sendToGroup`, `sendToUser`, and `broadcast`:

```json
{
  "target": "orderUpdate",
  "arguments": ["ORD-999", "shipped"]
}
```

- `target`: The client-side method name to invoke on connected clients
- `arguments`: Array of string arguments passed to the client method

### Operations Overview

#### Connections

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `checkConnection` | `connectionId` | Check whether a connection is currently active |
| `closeConnection` | `connectionId` | Forcefully close a specific connection |
| `sendToConnection` | `connectionId`, `payload` | Send a message to a single connection |

#### Groups

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `checkGroupConnections` | `group` | Check whether any connections exist in a group |
| `addConnectionToGroup` | `connectionId`, `group` | Add a connection to a group |
| `removeConnectionFromGroup` | `connectionId`, `group` | Remove a connection from a group |

#### Users

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `addUserToGroup` | `user`, `group` | Add a user to a group |
| `removeUserFromGroup` | `user`, `group` | Remove a user from a specific group |
| `removeUserFromAllGroups` | `user` | Remove a user from every group |
| `checkUserInGroup` | `user`, `group` | Check whether a user belongs to a group |
| `sendToUser` | `user`, `payload` | Send a message to all connections of a user |

#### Messaging

| Operation | Required Params | Description |
|-----------|-----------------|-------------|
| `sendToGroup` | `group`, `payload` | Send a message to all connections in a group |
| `broadcast` | `payload` | Send a message to all connected clients in the hub |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data — `connectionId`, `group`, and `user` can be resolved from this via expressions |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the operation |
| `error` | `Error` | Emitted if the operation or authentication fails |

### Output Schema

All operations return the same envelope:

```json
{
  "success": true,
  "operation": "broadcast",
  "status": 202,
  "statusText": "Accepted",
  "data": null,
  "timestamp": "2026-03-10T15:00:00.000Z",
  "context": {
    "hub": "notifications",
    "connectionId": null,
    "group": null,
    "user": null
  }
}
```

> Most SignalR Management API responses return HTTP 200 (check) or 202 (send/mutate) with an empty body. `data` will be `null` for these operations.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Broadcast to All Clients

Push a live price update to every client connected to the `market` hub.

**Configuration:**
```json
{
  "connectionString": "Endpoint=https://my-signalr.service.signalr.net;AccessKey=...;Version=1.0;",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientId": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
  "clientSecret": "your-client-secret",
  "hub": "market",
  "operation": "broadcast",
  "payload": {
    "target": "priceUpdate",
    "arguments": ["BTC", "85000"]
  }
}
```

---

### Example: Send to a Specific User

Send an order status notification to a specific user.

**Configuration:**
```json
{
  "connectionString": "Endpoint=https://my-signalr.service.signalr.net;AccessKey=...;Version=1.0;",
  "tenantId": "...",
  "clientId": "...",
  "clientSecret": "...",
  "hub": "orders",
  "operation": "sendToUser",
  "user": "{{input.userId}}",
  "payload": {
    "target": "orderStatusChanged",
    "arguments": ["{{input.orderId}}", "{{input.status}}"]
  }
}
```

---

### Example: Add Connection to Group

Move a connection into a broadcast group when a user subscribes.

**Configuration:**
```json
{
  "hub": "dashboard",
  "operation": "addConnectionToGroup",
  "connectionId": "{{input.connectionId}}",
  "group": "admins"
}
```

---

### Example: Check if Connection is Active

Verify a connection before sending a critical message.

**Configuration:**
```json
{
  "hub": "alerts",
  "operation": "checkConnection",
  "connectionId": "{{input.connectionId}}"
}
```

**Output:**
```json
{
  "success": true,
  "operation": "checkConnection",
  "status": 200,
  "data": null,
  "context": { "hub": "alerts", "connectionId": "abc123", "group": null, "user": null }
}
```

> HTTP 200 means the connection exists; HTTP 404 means it does not.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Data Pipeline Completion → Real-Time Dashboard Broadcast

After a batch data processing workflow finishes, broadcast the result to all dashboard clients.

```json
{
  "nodes": [
    {
      "id": "etl-complete",
      "type": "cron",
      "config": { "expression": "0 * * * *" }
    },
    {
      "id": "run-etl",
      "type": "function",
      "config": {
        "code": "return { summary: computeSummary(), updatedAt: new Date().toISOString() };"
      }
    },
    {
      "id": "broadcast-update",
      "type": "signalr-action",
      "config": {
        "hub": "dashboard",
        "operation": "broadcast",
        "payload": {
          "target": "dashboardRefresh",
          "arguments": ["{{input.summary}}", "{{input.updatedAt}}"]
        }
      }
    }
  ]
}
```

### Common Patterns

- **Role Change → Group Membership:** User role update → `addUserToGroup` / `removeUserFromGroup`
- **Alert Fan-out:** Critical event → `broadcast` to all clients in a monitoring hub
- **Personalised Push:** Order event → `sendToUser` with dynamic `user` from upstream data
- **Session Cleanup:** Inactivity timer → `checkConnection` → `closeConnection` if still open
- **Scoped Notification:** Slack message received → `sendToGroup` for a team-specific SignalR group

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Failed to acquire Entra access token`

**Cause:** `tenantId`, `clientId`, or `clientSecret` is incorrect, or the app registration lacks the `Azure SignalR Service` API permission.

**Solution:** Verify app registration credentials in the Azure portal. Ensure the app has the `Azure SignalR Service > SignalR App Server` role assignment on the SignalR resource.

#### `Invalid SignalR connection string: missing Endpoint`

**Cause:** The connection string is malformed or truncated.

**Solution:** Copy the full connection string from **Azure Portal → SignalR Service → Keys**. It must include `Endpoint=`, `AccessKey=`, and `Version=`.

#### `group and connectionId are required for addConnectionToGroup`

**Cause:** Either `group` or `connectionId` is empty or not resolved from an expression.

**Solution:** Check that upstream nodes are emitting the expected fields and that expressions like `{{input.connectionId}}` resolve to non-empty values.

#### HTTP 404 on `checkConnection` or `sendToConnection`

**Cause:** The connection ID no longer exists (client disconnected).

**Solution:** Use `checkConnection` before sending; handle 404 gracefully in a downstream function node.

### Error Codes

| Code | Meaning | Solution |
|------|---------|----------|
| `401` | Entra token rejected | Verify `clientId`, `clientSecret`, `tenantId` |
| `403` | Insufficient permissions | Assign the `SignalR App Server` role to the app registration |
| `404` | Connection / group / user not found | Verify IDs; connection may have dropped |
| `413` | Payload too large | Reduce `arguments` size |
| `0` (Network Error) | Cannot reach SignalR endpoint | Check `connectionString` and network access |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [SignalR Trigger](./signalr-trigger.md) - Receive real-time events from SignalR clients
- [Event Hub Send](./eventhub-send.md) - Send events to Azure Event Hubs for large-scale ingestion
- [Slack Action](./slack-action.md) - Alternative messaging channel for human-readable notifications

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
