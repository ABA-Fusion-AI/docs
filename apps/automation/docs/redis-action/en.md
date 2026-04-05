---
node_id: "redis-action"
title: "Redis Action"
description: "Perform Redis operations — key management, pub/sub messaging, and server info"
category: "data"
subcategory: "cache"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - redis
  - cache
  - key-value
  - pubsub
  - action
related_nodes:
  - redis-subscribe
  - mongodb-action
  - mysql-action
---

<!-- SECTION: header -->
# Redis Action

> **Category:** Data | **Type:** Action Node

Execute Redis operations on a connected Redis instance — read/write keys, manage counters, list keys, and publish messages to channels.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Redis Action** node connects to a Redis server and performs a selected operation. It supports common key-value operations as well as Pub/Sub publishing, making it ideal for caching, session management, counters, and real-time messaging workflows.

### Key Features

- **Key Management:** Get, set, delete, and increment keys
- **Pattern Matching:** List keys using glob-style patterns
- **Pub/Sub Publishing:** Publish messages to named channels
- **Server Info:** Retrieve structured Redis server diagnostics
- **Expression Support:** Key, value, and pattern fields accept dynamic expressions

### Use Cases

- Cache API responses with `setKey` and retrieve them with `getKey`
- Track page views or event counts with `incrementKey`
- Invalidate cache entries using `deleteKey`
- Publish notifications to real-time subscribers via `publish`
- Audit the Redis keyspace with `listKeys`
- Monitor server health with `info`

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | ✅ Yes | `localhost` | Redis server hostname or IP address |
| `port` | `number` | ✅ Yes | `6379` | Redis server port |
| `username` | `string` | ❌ No | — | Redis ACL username (Redis 6+) |
| `password` | `string` | ❌ No | — | Redis password |
| `db` | `number` | ❌ No | `0` | Redis database index (0–15) |

### Operation Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | — | Operation to perform (see below) |
| `key` | `string` | Conditional | — | Key name — required for `getKey`, `setKey`, `deleteKey`, `incrementKey` |
| `value` | `string` | Conditional | — | Value to store — required for `setKey` |
| `pattern` | `string` | Conditional | `*` | Glob pattern — used for `listKeys` |
| `channel` | `string` | Conditional | — | Channel name — required for `publish` |
| `message` | `string` | Conditional | — | Message payload — required for `publish` |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `getKey` | Retrieve the value of a key |
| `setKey` | Set a key to a given value |
| `deleteKey` | Delete one or more keys |
| `incrementKey` | Atomically increment a numeric key by 1 |
| `listKeys` | List all keys matching a glob pattern |
| `publish` | Publish a message to a Pub/Sub channel |
| `info` | Retrieve structured server diagnostics |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data (available via expressions in `key`, `value`, etc.) |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the executed operation |
| `error` | `Error` | Emitted if the operation fails |

### Output Schemas by Operation

**`getKey`**
```json
{
  "key": "user:123",
  "value": "John Doe",
  "exists": true
}
```

**`setKey`**
```json
{
  "success": true,
  "status": "OK"
}
```

**`deleteKey`**
```json
{
  "success": true,
  "deleted_count": 1
}
```

**`incrementKey`**
```json
{
  "success": true,
  "new_value": 42
}
```

**`listKeys`**
```json
{
  "count": 3,
  "keys": ["user:1", "user:2", "user:3"]
}
```

**`publish`**
```json
{
  "success": true,
  "subscribers_received": 2,
  "message": "Message delivered to 2 subscriber(s)"
}
```

**`info`**
```json
{
  "server": {
    "version": "7.0.5",
    "mode": "standalone",
    "os": "Linux",
    "uptime_days": "12"
  },
  "clients": {
    "connected": "4",
    "blocked": "0"
  },
  "memory": {
    "used": "1.23M",
    "peak": "1.50M",
    "rss": "2.10M"
  },
  "stats": {
    "total_connections": "1042",
    "total_commands": "98432",
    "keyspace_hits": "8210",
    "keyspace_misses": "132"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Cache a Value

Store an API response in Redis for 1 hour.

**Configuration:**
```json
{
  "host": "localhost",
  "port": 6379,
  "operation": "setKey",
  "key": "api:weather:paris",
  "value": "{{JSON.stringify(input.response)}}"
}
```

**Output:**
```json
{
  "success": true,
  "status": "OK"
}
```

---

### Example: Read a Cached Value

Retrieve a previously stored value.

**Configuration:**
```json
{
  "host": "localhost",
  "port": 6379,
  "operation": "getKey",
  "key": "api:weather:paris"
}
```

**Output:**
```json
{
  "key": "api:weather:paris",
  "value": "{\"temp\":22,\"condition\":\"Sunny\"}",
  "exists": true
}
```

---

### Example: Increment a Counter

Track the number of visits to a page.

**Configuration:**
```json
{
  "host": "localhost",
  "port": 6379,
  "operation": "incrementKey",
  "key": "visits:homepage"
}
```

**Output:**
```json
{
  "success": true,
  "new_value": 1053
}
```

---

### Example: List Keys by Pattern

Find all session keys in the database.

**Configuration:**
```json
{
  "host": "localhost",
  "port": 6379,
  "operation": "listKeys",
  "pattern": "session:*"
}
```

**Output:**
```json
{
  "count": 2,
  "keys": ["session:abc123", "session:xyz789"]
}
```

---

### Example: Publish a Notification

Push a real-time event to subscribers.

**Configuration:**
```json
{
  "host": "localhost",
  "port": 6379,
  "operation": "publish",
  "channel": "notifications",
  "message": "Order #4521 has been shipped"
}
```

**Output:**
```json
{
  "success": true,
  "subscribers_received": 3,
  "message": "Message delivered to 3 subscriber(s)"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Cache-Aside Pattern

Check the cache before calling an external API. Store the result if it was a cache miss.

```json
{
  "nodes": [
    {
      "id": "check-cache",
      "type": "redis-action",
      "position": { "x": 100, "y": 100 },
      "config": {
        "host": "localhost",
        "port": 6379,
        "operation": "getKey",
        "key": "product:{{input.productId}}"
      }
    },
    {
      "id": "fetch-from-api",
      "type": "http-request",
      "position": { "x": 300, "y": 200 },
      "config": {
        "url": "https://api.example.com/products/{{input.productId}}",
        "method": "GET"
      }
    },
    {
      "id": "store-in-cache",
      "type": "redis-action",
      "position": { "x": 500, "y": 200 },
      "config": {
        "host": "localhost",
        "port": 6379,
        "operation": "setKey",
        "key": "product:{{input.productId}}",
        "value": "{{JSON.stringify(input)}}"
      }
    }
  ]
}
```

### Common Patterns

- **Cache-Aside:** Check Redis → (miss) Fetch API → Store in Redis → Return
- **Rate Limiting:** `incrementKey` on `ratelimit:userId` → Check value → Block or allow
- **Invalidation:** On data update → `deleteKey` matching cache entries
- **Real-Time Events:** Action completes → `publish` to channel → Subscribers react

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Connection refused

**Cause:** Redis server is not running or the host/port is incorrect.

**Solution:** Verify that Redis is running and accessible from the workflow engine. Check firewall rules and confirm the `host` and `port` values.

#### `key is required` error

**Cause:** A `key`-dependent operation was configured without providing a key value.

**Solution:** Ensure the `key` field is filled in for `getKey`, `setKey`, `deleteKey`, and `incrementKey` operations.

#### `publish` returns `subscribers_received: 0`

**Cause:** No clients are currently subscribed to the target channel.

**Solution:** This is not an error — the message was published successfully. Use the [Redis Subscribe](./redis-subscribe.md) node to create active subscribers before publishing.

#### Key value is `null` in `getKey` output

**Cause:** The key does not exist in Redis.

**Solution:** Check the `exists` field in the output (`false` means the key was not found). Consider setting a default in a following Function node.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Check Redis host, port, and firewall |
| `WRONGPASS` | Invalid password | Verify the `password` field |
| `NOAUTH` | Authentication required | Provide `username` and/or `password` |
| `ERR` | Invalid operation arguments | Check that required fields are filled in |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Redis Subscribe](./redis-subscribe.md) - Trigger workflows from Redis Pub/Sub channels
- [MongoDB Action](./mongodb.md) - Document-based NoSQL operations
- [MySQL Action](./mysql.md) - Relational database operations

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
