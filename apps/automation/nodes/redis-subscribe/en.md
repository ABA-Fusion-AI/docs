---
node_id: "redis-subscribe"
title: "Redis Subscribe"
description: "Triggers workflow execution when messages are published to Redis Pub/Sub channels"
category: "triggers"
subcategory: "messaging"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - redis
  - trigger
  - pubsub
  - messaging
  - real-time
related_nodes:
  - redis-action
  - interval
  - cron
---

<!-- SECTION: header -->
# Redis Subscribe

> **Category:** Triggers | **Type:** Trigger Node

Listen to one or more Redis Pub/Sub channels and trigger a workflow execution each time a message is received.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Redis Subscribe** node connects to a Redis server and subscribes to one or more Pub/Sub channels. Every time a message is published to a subscribed channel, the node emits an output and triggers downstream nodes — making it ideal for real-time, event-driven workflows.

### Key Features

- **Multi-Channel:** Subscribe to multiple channels in a single node
- **Real-Time Triggering:** Fires immediately when a message arrives
- **Pause / Resume:** Unsubscribes and resubscribes without losing configuration
- **Timestamped Output:** Each trigger includes the message, channel, and reception timestamp

### Use Cases

- React to events published by other services or the [Redis Action](./redis-action.md) node
- Dispatch notifications when a background job completes
- Trigger data pipeline workflows from external producers
- Build real-time dashboards fed by Redis message streams

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | ✅ Yes | `localhost` | Redis server hostname or IP address |
| `port` | `number` | ✅ Yes | `6379` | Redis server port (1–65535) |
| `username` | `string` | ❌ No | — | Redis ACL username (Redis 6+) |
| `password` | `string` | ❌ No | — | Redis password |
| `db` | `number` | ❌ No | `0` | Redis database index |

### Subscription Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `channels` | `string[]` | ✅ Yes | — | List of channel names to subscribe to |

> **Note:** Each string in `channels` must be a non-empty channel name. At least one channel is required.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

This is a trigger node — it does not accept inputs from other nodes. It activates automatically when a message arrives on a subscribed channel.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted on each received message |
| `error` | `Error` | Emitted if a connection or subscription error occurs |

### Output Schema (`success`)

```json
{
  "channel": "notifications",
  "message": "Order #4521 has been shipped",
  "timestamp": 1741600000000
}
```

| Field | Type | Description |
|-------|------|-------------|
| `channel` | `string` | The channel on which the message was received |
| `message` | `string` | The raw message payload |
| `timestamp` | `number` | Unix timestamp (milliseconds) when the message was received |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Single Channel

Trigger a workflow whenever a message is published to the `notifications` channel.

**Configuration:**
```json
{
  "host": "localhost",
  "port": 6379,
  "channels": ["notifications"]
}
```

**Output (on message received):**
```json
{
  "channel": "notifications",
  "message": "New user registered: user@example.com",
  "timestamp": 1741600000000
}
```

---

### Advanced Example: Multiple Channels

Subscribe to both `orders` and `alerts` channels in a single node.

**Configuration:**
```json
{
  "host": "localhost",
  "port": 6379,
  "channels": ["orders", "alerts"]
}
```

**Output (when a message arrives on `orders`):**
```json
{
  "channel": "orders",
  "message": "{\"orderId\": \"ORD-789\", \"status\": \"shipped\"}",
  "timestamp": 1741600010000
}
```

> **Tip:** Use a Function node after this node to parse `JSON.parse(input.message)` if the message is a JSON string.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Real-Time Order Notification

Send a Slack message every time a new order event is published to Redis.

```json
{
  "nodes": [
    {
      "id": "redis-trigger",
      "type": "redis-subscribe",
      "position": { "x": 100, "y": 100 },
      "config": {
        "host": "localhost",
        "port": 6379,
        "channels": ["orders"]
      }
    },
    {
      "id": "parse-message",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "return { order: JSON.parse(input.message), channel: input.channel };"
      }
    },
    {
      "id": "send-slack",
      "type": "slack-send-message",
      "position": { "x": 500, "y": 100 },
      "config": {
        "channel": "#orders",
        "message": "New order received: {{input.order.orderId}}"
      }
    }
  ],
  "connections": [
    {
      "source": "redis-trigger",
      "sourceOutput": "success",
      "target": "parse-message",
      "targetInput": "input"
    },
    {
      "source": "parse-message",
      "sourceOutput": "output",
      "target": "send-slack",
      "targetInput": "input"
    }
  ]
}
```

### Common Patterns

- **Event Fan-Out:** Redis Subscribe → Function (parse) → Multiple downstream nodes
- **Job Completion Hook:** Worker publishes to Redis → Subscribe node → Post-processing workflow
- **Real-Time Alert:** Service publishes error → Subscribe node → Send email/Slack

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Node subscribes but never triggers

**Cause:** No messages are being published to the configured channels.

**Solution:** Use the [Redis Action](./redis-action.md) node with the `publish` operation to test that messages arrive. Verify that the publisher is targeting the exact same channel name (case-sensitive).

#### Connection drops and node stops receiving messages

**Cause:** Network interruption or Redis server restart.

**Solution:** The node will emit an error output on connection loss. Use error handling in the workflow to restart or alert. Consider deploying Redis with persistence and high availability.

#### `channels` field validation error

**Cause:** An empty string was provided in the channels array.

**Solution:** Each entry in `channels` must be a non-empty string. Remove any blank entries.

#### Messages received but `timestamp` seems incorrect

**Cause:** `timestamp` is generated on the workflow engine side using `Date.now()`, not from the Redis server.

**Solution:** If you need a precise server-side timestamp, include it in the message payload on the publisher side.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Verify Redis host, port, and firewall |
| `WRONGPASS` | Invalid password | Check the `password` field |
| `NOAUTH` | Authentication required | Provide `username` and/or `password` |
| `Subscribe failed` | Subscription error | Check channel names and Redis ACL permissions |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Redis Action](./redis-action.md) - Perform key operations and publish messages
- [Cron](./cron.md) - Time-based workflow trigger
- [Interval](./interval.md) - Fixed-interval workflow trigger

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
