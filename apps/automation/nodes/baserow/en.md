---
node_id: "baserow-action"
title: "Baserow Action"
description: "Perform Baserow database operations — list fields, list rows, get a row, create, update, and delete rows"
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - baserow
  - database
  - no-code
  - rest-api
  - action
related_nodes:
  - airtable-action
  - mongodb-action
  - function
---

<!-- SECTION: header -->
# Baserow Action

> **Category:** Data | **Type:** Action Node

Connect to a Baserow instance and interact with its database tables — list fields and rows, retrieve, create, update, and delete records using the Baserow REST API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Baserow Action** node connects to any Baserow instance (self-hosted or cloud) using an API token and performs one of six CRUD operations on a target table. All operations use Baserow's `user_field_names` mode, meaning field names in inputs and outputs match the column names defined in the Baserow UI — no numeric field IDs required.

### Key Features

- **Six Operations:** listFields, listRows, getRow, createRow, updateRow, deleteRow
- **User Field Names:** All row operations use human-readable column names, not internal IDs
- **Self-Hosted Support:** Works with any Baserow instance via the `baseUrl` parameter
- **Expression Support:** `tableId` and `rowId` support dynamic expressions
- **Field-Pair Input:** Create and update fields are defined as `[{ name, value }]` pairs for easy UI mapping

### Use Cases

- Store workflow results or API responses in a Baserow table
- Query a Baserow table to feed data into downstream workflow nodes
- Update records in Baserow based on incoming webhook data
- Delete outdated entries as part of a scheduled cleanup workflow
- Inspect the schema of a Baserow table before mapping fields

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `baseUrl` | `string` | ✅ Yes | `http://localhost:8000` | Base URL of the Baserow instance (e.g. `https://baserow.example.com`) |
| `token` | `string` | ✅ Yes | — | Baserow API token — generate from your account settings |

> To generate an API token in Baserow: **Account Settings → API tokens → Create token**.

### Operation Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Operation to execute (see below) |
| `tableId` | `number` | ✅ Yes | Numeric ID of the target table (found in the Baserow URL) |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `listFields` | List all fields (columns) defined in a table |
| `listRows` | List rows from a table with optional filtering, sorting, and pagination |
| `getRow` | Retrieve a single row by its ID |
| `createRow` | Create a new row in a table |
| `updateRow` | Update an existing row by its ID |
| `deleteRow` | Delete a row by its ID |

---

### Operation: listFields

No additional parameters required. Returns all field definitions for the specified `tableId`.

### Operation: listRows

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filters` | `string` (JSON) | ❌ No | JSON object of Baserow filter parameters (e.g. `{"filter__Name__contains": "Alice"}`) |
| `orderBy` | `string` | ❌ No | Comma-separated column names to sort by (prefix with `-` for descending, e.g. `-Name`) |
| `size` | `number` | ❌ No | Number of rows per page |
| `page` | `number` | ❌ No | Page number for pagination |

### Operation: getRow

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `rowId` | `number` | ✅ Yes | Numeric ID of the row to retrieve |

### Operation: createRow / updateRow

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fields` | `array` | ✅ Yes | Field values as `[{ name: "Column Name", value: "..." }]` |
| `rowId` | `number` | Conditional | Row ID — required for `updateRow` only |

### Operation: deleteRow

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `rowId` | `number` | ✅ Yes | Numeric ID of the row to delete |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — available via expressions in `tableId`, `rowId`, and field values |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | Result of the executed operation |
| `error` | `Error` | Emitted if the operation or API call fails |

### Output Schemas by Operation

**`listFields`**
```json
{
  "success": true,
  "count": 3,
  "fields": [
    { "id": 1, "table_id": 42, "name": "Name", "type": "text", "primary": true },
    { "id": 2, "table_id": 42, "name": "Status", "type": "single_select", "primary": false },
    { "id": 3, "table_id": 42, "name": "Created At", "type": "date", "primary": false }
  ]
}
```

**`listRows`**
```json
{
  "success": true,
  "count": 2,
  "rows": [
    { "id": 1, "order": "1.00", "Name": "Alice", "Status": "Active" },
    { "id": 2, "order": "2.00", "Name": "Bob", "Status": "Inactive" }
  ]
}
```

**`getRow`**
```json
{
  "success": true,
  "row": {
    "id": 1,
    "order": "1.00",
    "Name": "Alice",
    "Status": "Active",
    "Created At": "2026-01-15"
  }
}
```

**`createRow`**
```json
{
  "success": true,
  "row": {
    "id": 3,
    "order": "3.00",
    "Name": "Carol",
    "Status": "Active"
  }
}
```

**`updateRow`**
```json
{
  "success": true,
  "row": {
    "id": 3,
    "order": "3.00",
    "Name": "Carol",
    "Status": "Inactive"
  }
}
```

**`deleteRow`**
```json
{
  "success": true,
  "deletedRowId": 3
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: List All Rows

Retrieve all rows from a Baserow table.

**Configuration:**
```json
{
  "baseUrl": "https://baserow.example.com",
  "token": "your-api-token",
  "operation": "listRows",
  "tableId": 42
}
```

---

### Example: List Rows with Filter and Sort

Retrieve only active users, sorted by name.

**Configuration:**
```json
{
  "baseUrl": "https://baserow.example.com",
  "token": "your-api-token",
  "operation": "listRows",
  "tableId": 42,
  "filters": "{\"filter__Status__equal\": \"Active\"}",
  "orderBy": "Name",
  "size": 50,
  "page": 1
}
```

**Output:**
```json
{
  "success": true,
  "count": 2,
  "rows": [
    { "id": 1, "Name": "Alice", "Status": "Active" },
    { "id": 3, "Name": "Carol", "Status": "Active" }
  ]
}
```

---

### Example: Get a Single Row

Retrieve the row with ID `1`.

**Configuration:**
```json
{
  "baseUrl": "https://baserow.example.com",
  "token": "your-api-token",
  "operation": "getRow",
  "tableId": 42,
  "rowId": 1
}
```

---

### Example: Create a Row

Add a new record from a webhook payload.

**Configuration:**
```json
{
  "baseUrl": "https://baserow.example.com",
  "token": "your-api-token",
  "operation": "createRow",
  "tableId": 42,
  "fields": [
    { "name": "Name", "value": "{{input.name}}" },
    { "name": "Email", "value": "{{input.email}}" },
    { "name": "Status", "value": "Active" }
  ]
}
```

**Output:**
```json
{
  "success": true,
  "row": { "id": 5, "Name": "David", "Email": "david@example.com", "Status": "Active" }
}
```

---

### Example: Update a Row

Change the status of a specific row.

**Configuration:**
```json
{
  "baseUrl": "https://baserow.example.com",
  "token": "your-api-token",
  "operation": "updateRow",
  "tableId": 42,
  "rowId": "{{input.rowId}}",
  "fields": [
    { "name": "Status", "value": "Inactive" },
    { "name": "Updated At", "value": "{{new Date().toISOString()}}" }
  ]
}
```

---

### Example: Delete a Row

Remove a record by ID.

**Configuration:**
```json
{
  "baseUrl": "https://baserow.example.com",
  "token": "your-api-token",
  "operation": "deleteRow",
  "tableId": 42,
  "rowId": 5
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Webhook to Baserow

Receive a form submission via webhook and save it directly to a Baserow table.

```json
{
  "nodes": [
    {
      "id": "webhook",
      "type": "webhook-trigger",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "save-to-baserow",
      "type": "baserow-action",
      "position": { "x": 300, "y": 100 },
      "config": {
        "baseUrl": "https://baserow.example.com",
        "token": "your-api-token",
        "operation": "createRow",
        "tableId": 42,
        "fields": [
          { "name": "Name", "value": "{{input.name}}" },
          { "name": "Email", "value": "{{input.email}}" },
          { "name": "Message", "value": "{{input.message}}" },
          { "name": "Submitted At", "value": "{{new Date().toISOString()}}" }
        ]
      }
    }
  ]
}
```

### Common Patterns

- **Form → Baserow:** Webhook trigger → Baserow createRow (log submissions)
- **Read → Process:** Baserow listRows → Function (transform) → External API
- **Sync:** Cron → External API fetch → Baserow createRow / updateRow per item
- **Cleanup:** Cron → Baserow listRows (filter old entries) → Baserow deleteRow per result

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `401 Unauthorized`

**Cause:** The API `token` is invalid or has expired.

**Solution:** Generate a new token in Baserow under **Account Settings → API tokens**. Ensure the token has permission to access the target table.

#### `Table ID is required`

**Cause:** The `tableId` field was left empty.

**Solution:** Provide the numeric table ID. You can find it in the Baserow URL when viewing a table: `.../database/{databaseId}/table/{tableId}/`.

#### `Row ID is required`

**Cause:** `getRow`, `updateRow`, or `deleteRow` was selected but `rowId` was not provided.

**Solution:** Provide the numeric row ID. If it comes from a previous node, use an expression like `{{input.row.id}}`.

#### `Fields are required` on createRow / updateRow

**Cause:** The `fields` array is empty or missing.

**Solution:** Provide at least one `{ name, value }` pair. Verify that field names exactly match the column names in the Baserow table (case-sensitive).

#### Rows returned but fields are empty

**Cause:** The field names in the response don't match what was expected.

**Solution:** Use `listFields` first to inspect the exact column names in the table. All row operations use `user_field_names: true`, so column names must match the Baserow UI names exactly.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `401` | Unauthorized | Check API token validity |
| `404` | Table or row not found | Verify `tableId` and `rowId` |
| `400` | Bad request | Check field names and value formats |
| `ECONNREFUSED` | Connection refused | Verify `baseUrl` and network access |
| `ENOTFOUND` | DNS resolution failed | Check `baseUrl` hostname |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Airtable Action](./airtable-action.md) - Similar operations on Airtable bases
- [MongoDB Action](./mongodb.md) - Document-based NoSQL operations
- [Function](./function.md) - Transform data before creating or updating rows

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
