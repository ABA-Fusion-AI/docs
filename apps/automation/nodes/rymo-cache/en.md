---
node_id: "rymo-cache"
title: "Rymo Cache"
description: "Manage query and data caching in Redis — store, retrieve, invalidate table queries, clear by pattern, and inspect cache performance."
category: "data"
subcategory: "cache"
version: "1.0.0"
language: "en"
last_updated: "2026-09-03"
author: "Fusion Team"
tags:
  - rymo
  - cache
  - redis
  - database
  - query-cache
  - performance
related_nodes:
  - rymo-query
  - rymo-create
  - rymo-delete
  - redis-action
---

<!-- SECTION: header -->
# Rymo Cache

> **Category:** Data & Storage&nbsp;&nbsp;|&nbsp;&nbsp;**Subcategory:** Cache & Memory&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage high-performance query caching and key-value storage in Redis. Invalidate database table caches, purge key patterns, set temporary results with TTL, and monitor cache memory.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Rymo Cache** node provides direct, automated cache management for database workflows backed by Redis. When used alongside database query nodes like **Rymo Query**, it drastically reduces database load by caching repeated queries and invalidating cached datasets when tables are updated or records change.

It also functions as a standalone Redis key-value cache supporting time-to-live (TTL) expiration, wildcard pattern purging, and real-time keyspace statistics.

### Key Features

- **Table Invalidation (`invalidateTable`):** Invalidate all cached queries belonging to a specific database table (e.g. purging `rymo:users:*` when user records are updated).
- **Pattern Invalidation (`invalidatePattern`):** Delete keys matching flexible glob-style wildcards (e.g. `user_*`, `session:*`).
- **Full Cache Purge (`clearAll`):** Safely wipe all cached keys sharing the designated namespace prefix without flushing unrelated database keys.
- **Cache Statistics (`getStats`):** Inspect total key count and human-readable memory usage in real time.
- **Direct Key-Value Operations (`set`, `get`, `delete`):** Store arbitrary strings or serialized JSON payloads with custom expiration (TTL).
- **Key Namespacing (`keyPrefix`):** Automatically isolates application keys under a configurable prefix (`rymo` by default) to avoid key collisions.

### Use Cases

- **Post-Mutation Cache Invalidation:** Automatically clear cached `users` or `orders` queries immediately after an `INSERT`, `UPDATE`, or `DELETE` step in your workflow.
- **API Response Caching:** Store expensive third-party API payloads with a 5-minute TTL to reduce rate-limit consumption and speed up workflow execution.
- **Session & Token Management:** Store temporary verification tokens, OTPs, or authentication tokens that automatically expire after a few minutes.
- **Cache Observability:** Regularly query cache statistics (`getStats`) and forward memory usage metrics to Slack or monitoring systems.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Basic Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `redisUrl` | `string` | ✅ Yes | — | Redis connection URL. Supports standard `redis://` and encrypted `rediss://` protocols. |
| `operation` | `enum` | ✅ Yes | `invalidateTable` | The cache operation to execute (see [Operations](#operations)). |
| `keyPrefix` | `string` | ❌ No | `rymo` | Namespace prefix prepended to keys. Helps isolate cached datasets. |

### Operations & Conditional Parameters

Depending on the selected `operation`, additional conditional fields become available:

| Operation | Description | Required Parameters | Optional Parameters |
|-----------|-------------|---------------------|---------------------|
| `invalidateTable` | Invalidate all cached queries for a database table | `tableName` | `keyPrefix` |
| `invalidatePattern` | Delete keys matching a glob wildcard pattern | `pattern` | `keyPrefix` |
| `clearAll` | Delete all keys matching the namespace prefix | — | `keyPrefix` |
| `getStats` | Retrieve key count and memory statistics | — | `keyPrefix` |
| `set` | Store a key-value pair with an expiration time | `cacheKey`, `value` | `ttl`, `keyPrefix` |
| `get` | Retrieve the value of a specific key | `cacheKey` | `keyPrefix` |
| `delete` | Remove a specific key from the cache | `cacheKey` | `keyPrefix` |

### Parameter Details

#### `redisUrl`
The Redis connection string.
- **Local instance:** `redis://localhost:6379` or `redis://127.0.0.1:6379`
- **Authenticated instance:** `redis://:yourpassword@127.0.0.1:6379`
- **Cloud / Managed Redis (Upstash, Redis Cloud, Aiven):** `rediss://default:password@endpoint:6379`

#### `tableName`
*(Available for `invalidateTable`)*  
The database table name whose query cache should be cleared. The node purges all keys matching `${keyPrefix}:${tableName}:*`.

#### `pattern`
*(Available for `invalidatePattern`)*  
A glob pattern to match keys against. If the pattern contains a colon `:`, it is evaluated as-is; otherwise, `${keyPrefix}:` is automatically prepended.
- Example: `user_*` resolves to `rymo:user_*`
- Example: `custom:namespace:*` resolves to `custom:namespace:*`

#### `cacheKey`
*(Available for `set`, `get`, and `delete`)*  
The identifier for the cached entry. If the key contains a colon `:`, it is preserved as-is; otherwise, it is prefixed with `${keyPrefix}:`.

#### `value`
*(Available for `set`)*  
The string or serialized JSON data to store. Upstream objects can be passed dynamically using expressions such as `{{JSON.stringify(input.data)}}`.

#### `ttl`
*(Available for `set`)*  
Time-To-Live in seconds. Minimum is `1`, maximum is `86400` (24 hours). Default is `300` (5 minutes). Once the TTL elapses, Redis expires and removes the key automatically.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data from previous nodes, accessible via expressions in `cacheKey`, `value`, `tableName`, etc. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the cache operation succeeds, containing operation-specific details. |
| `error` | `Error` | Emitted if connection fails or invalid parameters are provided. |

### Output Schemas by Operation

#### 1. `set`
Emitted after writing a key to Redis:
```json
{
  "success": true,
  "operation": "set",
  "key": "rymo:user_101",
  "ttl": 300
}
```

#### 2. `get`
Emitted when retrieving a key. If the key does not exist, `found` is `false` and `value` is `null`:
```json
{
  "success": true,
  "operation": "get",
  "key": "rymo:user_101",
  "value": "{\"id\": 101, \"name\": \"Abdel\", \"role\": \"admin\"}",
  "found": true
}
```

#### 3. `getStats`
Returns active key metrics for the current prefix:
```json
{
  "success": true,
  "operation": "getStats",
  "stats": {
    "totalKeys": 12,
    "memoryUsed": "1.42M"
  }
}
```

#### 4. `invalidateTable`
Returns the count of query cache entries removed for the given table:
```json
{
  "success": true,
  "operation": "invalidateTable",
  "table": "users",
  "keysInvalidated": 3
}
```

#### 5. `invalidatePattern`
Returns the count of keys deleted matching the glob pattern:
```json
{
  "success": true,
  "operation": "invalidatePattern",
  "pattern": "rymo:user_*",
  "keysInvalidated": 5
}
```

#### 6. `delete`
Confirms deletion of a single key:
```json
{
  "success": true,
  "operation": "delete",
  "key": "rymo:user_101"
}
```

#### 7. `clearAll`
Confirms removal of all keys matching the namespace prefix:
```json
{
  "success": true,
  "operation": "clearAll",
  "keysCleared": 18
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Manage Rymo Query Cache Operations
```

### Typical Workflow Patterns

#### Pattern A: Cache-Aside Database Read
1. Check **Rymo Cache** with operation `get` and `cacheKey: "products:featured"`.
2. Use a **Condition (If)** node:
   - If `input.found === true`, proceed with cached data.
   - If `input.found === false`, query PostgreSQL via **Rymo Query**, format results, and call **Rymo Cache** (`set`) with a 600s TTL.

#### Pattern B: Invalidate Table on Database Update
1. A webhook triggers an update on table `orders` via **Rymo Update**.
2. On `success`, wire the flow to **Rymo Cache**:
   - `operation`: `invalidateTable`
   - `tableName`: `orders`
3. Subsequent read queries automatically fetch fresh data from the database.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Cache operation failed: Failed to connect to Redis`
- **Cause:** The Redis instance is unreachable, wrong host/port, firewall blockage, or incorrect password.
- **Solution:** Verify the `redisUrl` connection string. For cloud providers (such as Upstash or Redis Cloud), ensure you use `rediss://` (with two 's' characters for TLS encryption) and verify that your IP is allowed.

#### `Table name is required for invalidateTable operation`
- **Cause:** The `tableName` field was left empty when running `invalidateTable`.
- **Solution:** Provide the database table name (e.g. `users`, `orders`, `products`).

#### `invalidateTable returns keysInvalidated: 0`
- **Cause:** The cached keys do not follow the `${keyPrefix}:${tableName}:*` naming convention, or no keys are currently cached for this table.
- **Solution:** Ensure your query caching writes keys under `${keyPrefix}:${tableName}:<hash>`. If setting keys manually, use `rymo:tableName:keyName`.

#### `Cache key is required for set operation`
- **Cause:** `cacheKey` parameter is missing or empty.
- **Solution:** Provide a non-empty string for `cacheKey`.

### Error Reference

| Error Message | Cause | Resolution |
|---------------|-------|------------|
| `Redis Client Error: WRONGPASS` | Authentication failure | Check the password in your `redisUrl` |
| `Cache key is required for ...` | Missing required `cacheKey` | Supply a valid key name |
| `Pattern is required for invalidatePattern operation` | Missing `pattern` parameter | Provide a wildcard string (e.g. `user_*`) |
| `Unsupported operation: ...` | Unknown operation string passed | Select one of the 7 supported operations |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Rymo Query](../rymo-query/en.md) – Execute safe queries against PostgreSQL with automatic query caching
- [Rymo Create](../rymo-create/en.md) – Insert records into PostgreSQL tables
- [Rymo Delete](../rymo-delete/en.md) – Delete database records and trigger cache invalidation
- [Redis Action](../redis-action/en.md) – Generic Redis operations and Pub/Sub messaging

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-03 | Initial release with table invalidation, pattern invalidation, key-value storage, and cache statistics |

<!-- /SECTION: changelog -->
