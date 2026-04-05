---
node_id: "postgres-action"
title: "PostgreSQL Action"
description: "Execute PostgreSQL operations — select, insert, update, upsert, delete, and raw SQL queries with schema support"
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - postgres
  - postgresql
  - database
  - sql
  - relational
  - action
related_nodes:
  - mysql-action
  - mongodb-action
  - function
---

<!-- SECTION: header -->
# PostgreSQL Action

> **Category:** Data | **Type:** Action Node

Connect to a PostgreSQL database and execute structured operations — query rows, insert records, update data, perform upserts, delete or truncate tables, or run raw SQL statements. Supports schema-qualified table access and parameterized queries.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **PostgreSQL Action** node connects to a PostgreSQL server using a connection pool and executes one of six available operations. It uses parameterized queries (`$1`, `$2`, ...) to prevent SQL injection and fully supports PostgreSQL-specific features such as named schemas, `ON CONFLICT ... DO UPDATE`, and `CASCADE` on destructive operations.

### Key Features

- **Six Operations:** Select, Insert, Update, Insert or Update (upsert), Delete, Execute SQL query
- **Schema Support:** Target any schema with `schemaName` (defaults to `public`)
- **Parameterized Queries:** All values are passed as bound parameters — no SQL injection risk
- **Upsert via `ON CONFLICT`:** Uses `EXCLUDED` pseudo-table for conflict resolution
- **Cascade Option:** `TRUNCATE` and `DROP` support the `CASCADE` flag
- **Null Handling:** Optionally replace empty strings with `NULL` on write operations
- **Connection Pooling:** Uses `pg.Pool` for efficient, reusable connections

### Use Cases

- Query analytics data from a PostgreSQL data warehouse
- Insert event logs or webhook payloads into a structured table
- Update user records based on incoming workflow data
- Upsert product entries using a unique SKU or email as conflict key
- Clean up expired records with DELETE or TRUNCATE on a schedule
- Run complex reports or migrations with raw SQL

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `host` | `string` | ✅ Yes | — | PostgreSQL server hostname or IP |
| `port` | `number` | ✅ Yes | `5432` | PostgreSQL server port |
| `user` | `string` | ✅ Yes | — | PostgreSQL username |
| `password` | `string` | ✅ Yes | — | PostgreSQL password |
| `database` | `string` | ✅ Yes | — | Target database name |
| `schemaName` | `string` | ❌ No | `public` | Schema containing the target table |
| `connectionTimeoutMillis` | `number` | ❌ No | `5000` | Connection timeout in milliseconds |

### Common Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `tableName` | `string` | ✅ Yes | — | Target table name |
| `operation` | `enum` | ✅ Yes | — | Operation to perform (see below) |
| `replaceEmptyStringWithNull` | `boolean` | ❌ No | `false` | Convert empty string values to `NULL` on Insert / Update / Upsert |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `Select` | Query rows from a table |
| `Insert` | Insert a new row |
| `Update` | Update rows matching a WHERE condition |
| `Insert or Update` | Upsert using `ON CONFLICT (...) DO UPDATE SET` |
| `Delete` | Delete rows, truncate, or drop a table |
| `Execute SQL query` | Run a raw SQL statement |

---

### Operation: Select

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `selectParams.returnAll` | `boolean` | ❌ No | Return all rows (ignores `limit`) |
| `selectParams.limit` | `number` | ❌ No | Maximum rows to return |
| `selectParams.where` | `string` | ❌ No | SQL WHERE clause (e.g. `status = 'active'`) |
| `selectParams.columns` | `string[]` | ❌ No | Columns to return — defaults to `*` |
| `selectParams.sort` | `array` | ❌ No | Sort rules: `[{ column, order: "asc" \| "desc" }]` |

### Operation: Insert

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `insertParams.rows` | `array` | ✅ Yes | Fields: `[{ rowName, rowValue }]` |
| `insertParams.skipOnConflict` | `boolean` | ❌ No | Use `ON CONFLICT DO NOTHING` to skip duplicates |

### Operation: Update

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `updateParams.where` | `string` | ✅ Yes | SQL WHERE clause to identify rows |
| `updateParams.rows` | `array` | ✅ Yes | Fields to update: `[{ rowName, rowValue }]` |

### Operation: Insert or Update

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `insertOrUpdateParams.rows` | `array` | ✅ Yes | Fields: `[{ rowName, rowValue }]` |
| `insertOrUpdateParams.conflictColumns` | `string[]` | ✅ Yes | Columns defining the conflict constraint (e.g. `["email"]`) |

> On conflict, all non-conflict columns are updated using `EXCLUDED.<column>` values.

### Operation: Delete

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `deleteParams.operation` | `enum` | ✅ Yes | `Delete`, `Truncate`, or `Drop` |
| `deleteParams.where` | `string` | Conditional | SQL WHERE clause — required for `Delete` |
| `deleteParams.cascade` | `boolean` | ❌ No | Apply `CASCADE` for `Truncate` or `Drop` |

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
    { "id": 1, "name": "Alice", "email": "alice@example.com" },
    { "id": 2, "name": "Bob", "email": "bob@example.com" }
  ]
}
```

**`Insert`**
```json
{
  "operation": "inserted",
  "rowCount": 1
}
```

**`Update`**
```json
{
  "operation": "updated",
  "rowCount": 3
}
```

**`Insert or Update`**
```json
{
  "operation": "inserted or updated",
  "rowCount": 1
}
```

**`Delete` (row-level)**
```json
{
  "operation": "delete",
  "rowCount": 5
}
```

**`Delete` (truncate)**
```json
{
  "operation": "truncate",
  "rowCount": null
}
```

**`Execute SQL query` (SELECT)**
```json
{
  "rows": [{ "total": 1280 }],
  "rowCount": 1,
  "command": "SELECT"
}
```

**`Execute SQL query` (DML)**
```json
{
  "rows": [],
  "rowCount": 3,
  "command": "UPDATE"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Select with Filter and Sort

Query up to 50 active users from the `analytics` schema, sorted by creation date.

**Configuration:**
```json
{
  "host": "db.example.com",
  "port": 5432,
  "user": "app_user",
  "password": "secret",
  "database": "myapp",
  "schemaName": "analytics",
  "tableName": "users",
  "operation": "Select",
  "selectParams": {
    "returnAll": false,
    "limit": 50,
    "where": "status = 'active'",
    "columns": ["id", "name", "email", "created_at"],
    "sort": [{ "column": "created_at", "order": "desc" }]
  }
}
```

**Generated SQL:**
```sql
SELECT "id", "name", "email", "created_at"
FROM "analytics"."users"
WHERE status = 'active'
ORDER BY "created_at" DESC
LIMIT 50;
```

---

### Example: Insert a Row

Insert a new lead into the `public.leads` table.

**Configuration:**
```json
{
  "tableName": "leads",
  "schemaName": "public",
  "operation": "Insert",
  "insertParams": {
    "rows": [
      { "rowName": "email", "rowValue": "{{input.email}}" },
      { "rowName": "name", "rowValue": "{{input.name}}" },
      { "rowName": "source", "rowValue": "webhook" }
    ],
    "skipOnConflict": false
  }
}
```

**Generated SQL:**
```sql
INSERT INTO "public"."leads" ("email", "name", "source")
VALUES ($1, $2, $3);
```

---

### Example: Update Rows

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

**Generated SQL:**
```sql
UPDATE "public"."users"
SET "status" = $1, "updated_at" = $2
WHERE id = 42;
```

---

### Example: Upsert a Product

Insert or update a product record based on its SKU.

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

**Generated SQL:**
```sql
INSERT INTO "public"."products" ("sku", "name", "price")
VALUES ($1, $2, $3)
ON CONFLICT ("sku")
DO UPDATE SET "sku" = EXCLUDED."sku", "name" = EXCLUDED."name", "price" = EXCLUDED."price";
```

---

### Example: Truncate with Cascade

Clear a staging table and all dependent rows.

**Configuration:**
```json
{
  "tableName": "staging_events",
  "operation": "Delete",
  "deleteParams": {
    "operation": "Truncate",
    "cascade": true
  }
}
```

**Generated SQL:**
```sql
TRUNCATE TABLE "public"."staging_events" CASCADE;
```

---

### Example: Raw SQL Query

Run a custom aggregation.

**Configuration:**
```json
{
  "tableName": "orders",
  "operation": "Execute SQL query",
  "executeQueryParams": {
    "query": "SELECT date_trunc('month', created_at) AS month, COUNT(*) AS total FROM public.orders GROUP BY 1 ORDER BY 1 DESC"
  }
}
```

**Output:**
```json
{
  "rows": [
    { "month": "2026-03-01T00:00:00.000Z", "total": "142" },
    { "month": "2026-02-01T00:00:00.000Z", "total": "118" }
  ],
  "rowCount": 2,
  "command": "SELECT"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Daily Cleanup Job

Run every night to delete expired sessions from the database.

```json
{
  "nodes": [
    {
      "id": "schedule",
      "type": "cron",
      "position": { "x": 100, "y": 100 },
      "config": {
        "cronExpression": "0 2 * * *"
      }
    },
    {
      "id": "delete-expired",
      "type": "postgres-action",
      "position": { "x": 300, "y": 100 },
      "config": {
        "host": "db.example.com",
        "port": 5432,
        "user": "app_user",
        "password": "secret",
        "database": "myapp",
        "schemaName": "public",
        "tableName": "sessions",
        "operation": "Delete",
        "deleteParams": {
          "operation": "Delete",
          "where": "expires_at < NOW()"
        }
      }
    },
    {
      "id": "log-result",
      "type": "function",
      "position": { "x": 500, "y": 100 },
      "config": {
        "code": "console.log(`Deleted ${input.rowCount} expired sessions`); return input;"
      }
    }
  ]
}
```

### Common Patterns

- **ETL:** Fetch external data → Function (transform) → Postgres Insert
- **Sync:** Cron trigger → Postgres Select → HTTP push to external API
- **Upsert Pipeline:** Incoming webhook → Postgres Insert or Update (idempotent)
- **Cleanup Job:** Cron trigger → Postgres Delete (WHERE expiry) → Log result
- **Report:** Cron trigger → Postgres Execute SQL query → Email result

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Connection timeout

**Cause:** The PostgreSQL server is unreachable or overloaded.

**Solution:** Verify `host`, `port`, and network access. Increase `connectionTimeoutMillis` if needed. Ensure the PostgreSQL server is running and accepting connections from the workflow engine's IP.

#### `password authentication failed`

**Cause:** Wrong `user` or `password`.

**Solution:** Double-check credentials. Ensure the PostgreSQL user exists and has access to the target database. Use `\du` in `psql` to verify user privileges.

#### `relation "schema"."table" does not exist`

**Cause:** Wrong `schemaName` or `tableName`.

**Solution:** Confirm that the table exists in the specified schema. The default schema is `public`. Use `\dt schema.*` in `psql` to list tables.

#### `where required` on Delete

**Cause:** `deleteParams.operation` is `Delete` but no `where` clause was provided.

**Solution:** Provide a WHERE condition. Use `Truncate` to clear all rows, or `Drop` to remove the entire table.

#### `conflictColumns is required` on Upsert

**Cause:** `insertOrUpdateParams.conflictColumns` is empty or missing.

**Solution:** Provide at least one column that has a UNIQUE or PRIMARY KEY constraint in PostgreSQL. Example: `["email"]` or `["id"]`.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `ECONNREFUSED` | Connection refused | Check host, port, and firewall |
| `28P01` | Password authentication failed | Verify credentials |
| `42P01` | Relation does not exist | Check `schemaName` and `tableName` |
| `23505` | Unique violation | Use `Insert or Update` or enable `skipOnConflict` |
| `42601` | SQL syntax error | Check the raw SQL in `executeQueryParams.query` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [MySQL Action](./mysql.md) - MySQL relational database operations
- [MongoDB Action](./mongodb.md) - Document-based NoSQL operations
- [Function](./function.md) - Transform data before inserting

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
