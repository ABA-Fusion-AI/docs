---
node_id: "mysql-action"
title: "MySQL Action"
description: "Execute MySQL operations — select, insert, update, upsert, delete, and raw SQL queries"
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - mysql
  - database
  - sql
  - relational
  - action
related_nodes:
  - mongodb-action
  - redis-action
  - function
---

<!-- SECTION: header -->
# MySQL Action

> **Category:** Data | **Type:** Action Node

Connect to a MySQL database and execute structured operations — query rows, insert records, update data, perform upserts, delete entries, or run raw SQL statements.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **MySQL Action** node establishes a connection to a MySQL server and executes one of six available operations against a target table. It supports parameterized queries to prevent SQL injection and provides structured, predictable output for each operation type.

### Key Features

- **Six Operations:** Select, Insert, Update, Insert or Update (upsert), Delete, Execute SQL query
- **Flexible Select:** Filter with WHERE clauses, sort by columns, limit results, and select specific columns
- **Upsert Support:** `INSERT ... ON DUPLICATE KEY UPDATE` using configurable conflict columns
- **Null Handling:** Optionally replace empty strings with `NULL` on write operations
- **Raw SQL Mode:** Execute any SQL statement directly for advanced use cases
- **Big Number Support:** Handles `BIGINT` values safely as strings

### Use Cases

- Query customer records and feed them into downstream nodes
- Insert form submissions or API responses into a database table
- Update user profiles based on incoming webhook data
- Upsert product catalog entries using SKU as the conflict key
- Delete stale records as part of a scheduled cleanup workflow
- Run custom reports or migrations with raw SQL

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | ✅ Yes | `localhost` | MySQL server hostname or IP address |
| `port` | `number` | ✅ Yes | `3306` | MySQL server port |
| `user` | `string` | ✅ Yes | — | MySQL username |
| `password` | `string` | ✅ Yes | — | MySQL password |
| `database` | `string` | ✅ Yes | — | Database name to connect to |
| `connectionTimeout` | `number` | ❌ No | `10000` | Connection timeout in milliseconds |

### Common Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `tableName` | `string` | ✅ Yes | — | Target table name |
| `operation` | `enum` | ✅ Yes | — | Operation to perform (see below) |
| `replaceEmptyStringWithNull` | `boolean` | ❌ No | `false` | Replace `""` values with `NULL` on Insert / Update / Upsert |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `Select` | Query rows from a table |
| `Insert` | Insert a new row |
| `Update` | Update existing rows matching a WHERE condition |
| `Insert or Update` | Insert a row or update it on duplicate key conflict |
| `Delete` | Delete rows, truncate, or drop a table |
| `Execute SQL query` | Run a raw SQL statement |

---

### Operation: Select

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `selectParams.returnAll` | `boolean` | ❌ No | Return all rows (ignores `limit`) |
| `selectParams.limit` | `number` | ❌ No | Maximum number of rows to return |
| `selectParams.where` | `string` | ❌ No | SQL WHERE clause (e.g. `status = 'active'`) |
| `selectParams.columns` | `string[]` | ❌ No | Columns to return — defaults to `*` |
| `selectParams.sort` | `array` | ❌ No | Sort rules: `[{ column, order: "asc" \| "desc" }]` |

### Operation: Insert

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `insertParams.rows` | `array` | ✅ Yes | Fields to insert: `[{ rowName, rowValue }]` |
| `insertParams.skipOnConflict` | `boolean` | ❌ No | Use `INSERT IGNORE` to skip duplicate key errors |

### Operation: Update

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `updateParams.where` | `string` | ✅ Yes | SQL WHERE clause to identify rows to update |
| `updateParams.rows` | `array` | ✅ Yes | Fields to update: `[{ rowName, rowValue }]` |

### Operation: Insert or Update

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `insertOrUpdateParams.rows` | `array` | ✅ Yes | Fields: `[{ rowName, rowValue }]` |
| `insertOrUpdateParams.conflictColumns` | `string[]` | ✅ Yes | Columns that define a duplicate (e.g. `["email"]`) |

### Operation: Delete

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deleteParams.operation` | `enum` | ✅ Yes | `Delete`, `Truncate`, or `Drop` |
| `deleteParams.where` | `string` | Conditional | SQL WHERE clause — required for `Delete` |

### Operation: Execute SQL query

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `executeQueryParams.query` | `string` | ✅ Yes | Raw SQL statement to execute |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data (usable via expressions in field values) |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the executed operation |
| `error` | `Error` | Emitted if the operation or connection fails |

### Output Schemas by Operation

**`Select`**
```json
{
  "count": 2,
  "rows": [
    { "id": 1, "name": "Alice", "status": "active" },
    { "id": 2, "name": "Bob", "status": "active" }
  ]
}
```

**`Insert`**
```json
{
  "operation": "inserted",
  "affectedRows": 1,
  "insertId": 42
}
```

**`Update`**
```json
{
  "operation": "updated",
  "affectedRows": 3
}
```

**`Insert or Update`**
```json
{
  "operation": "inserted or updated",
  "affectedRows": 2,
  "insertId": 0
}
```

**`Delete`**
```json
{
  "operation": "delete",
  "affectedRows": 5
}
```

**`Execute SQL query` (SELECT)**
```json
{
  "rows": [{ "total": "1280" }],
  "count": 1
}
```

**`Execute SQL query` (DML)**
```json
{
  "affectedRows": 3,
  "insertId": 0,
  "message": ""
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Select Active Users

Query up to 100 active users, sorted by creation date.

**Configuration:**
```json
{
  "host": "db.example.com",
  "port": 3306,
  "user": "app_user",
  "password": "secret",
  "database": "myapp",
  "tableName": "users",
  "operation": "Select",
  "selectParams": {
    "returnAll": false,
    "limit": 100,
    "where": "status = 'active'",
    "columns": ["id", "name", "email", "created_at"],
    "sort": [{ "column": "created_at", "order": "desc" }]
  }
}
```

**Output:**
```json
{
  "count": 2,
  "rows": [
    { "id": 5, "name": "Carol", "email": "carol@example.com", "created_at": "2026-02-01" },
    { "id": 3, "name": "Alice", "email": "alice@example.com", "created_at": "2026-01-15" }
  ]
}
```

---

### Example: Insert a New Record

Insert a new order into the `orders` table.

**Configuration:**
```json
{
  "tableName": "orders",
  "operation": "Insert",
  "insertParams": {
    "rows": [
      { "rowName": "customer_id", "rowValue": "{{input.customerId}}" },
      { "rowName": "product_sku", "rowValue": "{{input.sku}}" },
      { "rowName": "quantity", "rowValue": "{{input.qty}}" },
      { "rowName": "status", "rowValue": "pending" }
    ],
    "skipOnConflict": false
  }
}
```

**Output:**
```json
{
  "operation": "inserted",
  "affectedRows": 1,
  "insertId": 1024
}
```

---

### Example: Update a User's Status

Update the status of a specific user.

**Configuration:**
```json
{
  "tableName": "users",
  "operation": "Update",
  "updateParams": {
    "where": "id = 42",
    "rows": [
      { "rowName": "status", "rowValue": "inactive" },
      { "rowName": "updated_at", "rowValue": "2026-03-10" }
    ]
  }
}
```

**Output:**
```json
{
  "operation": "updated",
  "affectedRows": 1
}
```

---

### Example: Upsert a Product

Insert a product by SKU, or update its price if it already exists.

**Configuration:**
```json
{
  "tableName": "products",
  "operation": "Insert or Update",
  "insertOrUpdateParams": {
    "conflictColumns": ["sku"],
    "rows": [
      { "rowName": "sku", "rowValue": "WIDGET-XL" },
      { "rowName": "name", "rowValue": "Widget XL" },
      { "rowName": "price", "rowValue": "29.99" }
    ]
  }
}
```

---

### Example: Delete Old Records

Delete orders older than 90 days.

**Configuration:**
```json
{
  "tableName": "orders",
  "operation": "Delete",
  "deleteParams": {
    "operation": "Delete",
    "where": "created_at < DATE_SUB(NOW(), INTERVAL 90 DAY)"
  }
}
```

---

### Example: Raw SQL Query

Run a custom aggregation query.

**Configuration:**
```json
{
  "tableName": "orders",
  "operation": "Execute SQL query",
  "executeQueryParams": {
    "query": "SELECT status, COUNT(*) as total FROM orders GROUP BY status"
  }
}
```

**Output:**
```json
{
  "rows": [
    { "status": "pending", "total": "12" },
    { "status": "shipped", "total": "45" },
    { "status": "cancelled", "total": "3" }
  ],
  "count": 3
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Webhook to Database

Receive an incoming webhook, validate the payload, and insert it into MySQL.

```json
{
  "nodes": [
    {
      "id": "receive-webhook",
      "type": "webhook-trigger",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "validate",
      "type": "function",
      "position": { "x": 300, "y": 100 },
      "config": {
        "code": "if (!input.email) throw new Error('email is required'); return input;"
      }
    },
    {
      "id": "save-to-db",
      "type": "mysql-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "host": "db.example.com",
        "port": 3306,
        "user": "app_user",
        "password": "secret",
        "database": "myapp",
        "tableName": "leads",
        "operation": "Insert",
        "insertParams": {
          "rows": [
            { "rowName": "email", "rowValue": "{{input.email}}" },
            { "rowName": "name", "rowValue": "{{input.name}}" },
            { "rowName": "source", "rowValue": "webhook" }
          ]
        }
      }
    }
  ]
}
```

### Common Patterns

- **ETL:** Fetch external data → Function (transform) → MySQL Insert
- **Sync:** Cron trigger → MySQL Select → HTTP Request (push to external API)
- **Upsert Pipeline:** Incoming data → MySQL Insert or Update (idempotent writes)
- **Cleanup Job:** Cron trigger → MySQL Delete (WHERE date condition) → Log result

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Connection timeout

**Cause:** The MySQL server is unreachable or the `connectionTimeout` is too low.

**Solution:** Verify host, port, and network access. Increase `connectionTimeout` if the server is slow to respond (e.g. cold start or high load).

#### `Access denied for user` error

**Cause:** Invalid credentials or the user lacks privileges on the target database.

**Solution:** Check `user` and `password`. Ensure the MySQL user has the necessary `SELECT`, `INSERT`, `UPDATE`, or `DELETE` privileges on the target database and table.

#### `tableName is required` error

**Cause:** The `tableName` field was left empty.

**Solution:** Provide a valid table name. For `Execute SQL query`, `tableName` is still required by the node configuration even if not used in the query itself.

#### `where` clause missing for Delete operation

**Cause:** `deleteParams.operation` is set to `Delete` but no `where` condition was provided.

**Solution:** Always provide a WHERE clause for row-level deletion. Use `Truncate` to clear all rows, or `Drop` to remove the table entirely.

#### Empty string values stored instead of NULL

**Cause:** `replaceEmptyStringWithNull` is `false` (default).

**Solution:** Set `replaceEmptyStringWithNull` to `true` to automatically convert empty string field values to `NULL` during Insert, Update, or Upsert operations.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Check host, port, and firewall |
| `ER_ACCESS_DENIED_ERROR` | Access denied | Verify MySQL credentials |
| `ER_NO_SUCH_TABLE` | Table doesn't exist | Check `tableName` spelling |
| `ER_DUP_ENTRY` | Duplicate entry | Use `Insert or Update` or enable `skipOnConflict` |
| `ER_PARSE_ERROR` | SQL syntax error | Check the raw SQL in `executeQueryParams.query` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [MongoDB Action](./mongodb.md) - Document-based NoSQL operations
- [Redis Action](./redis-action.md) - Key-value cache operations
- [Function](./function.md) - Transform data before inserting

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
