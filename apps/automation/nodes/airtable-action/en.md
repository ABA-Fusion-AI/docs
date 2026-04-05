---
node_id: "airtable-action"
title: "Airtable Action"
description: "Create, read, update, delete, and list records in Airtable tables using the official Airtable API"
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-03-10"
author: "Fusion Team"
tags:
  - airtable
  - database
  - no-code
  - spreadsheet
  - crm
  - action
related_nodes:
  - airtable-trigger
  - baserow-action
  - function
---

<!-- SECTION: header -->
# Airtable Action

> **Category:** Data | **Type:** Action Node

Connect to an Airtable base and perform CRUD operations on a table — create, read, update, delete, and list records. Uses the official Airtable API with `typecast` enabled for flexible field value handling.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Airtable Action** node connects to the Airtable API using a personal access token and performs one of five operations on a named table within a specified base. It uses Airtable's `typecast` option, which automatically converts string values to the correct field type (numbers, dates, select options, etc.) without requiring manual type casting.

### Key Features

- **Five Operations:** Append, Update, Delete, Read, List
- **Typecast Support:** String values are automatically cast to the correct Airtable field type
- **Record ID Targeting:** Update, Delete, and Read operations target records by their Airtable `recordId`
- **Field-Pair Input:** Append and Update fields are defined as `[{ fieldName, fieldValue }]` pairs
- **Full Record Output:** All outputs include the `recordId` alongside all field values

### Use Cases

- Log workflow events or form submissions to an Airtable base
- Sync external API data into an Airtable CRM or project tracker
- Update records in Airtable based on incoming webhook events
- Read a specific record to drive conditional logic in a workflow
- List all records for batch processing or reporting

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `string` | ✅ Yes | Airtable personal access token — generate from **airtable.com/create/tokens** |
| `baseId` | `string` | ✅ Yes | Airtable Base ID — found in the base URL: `airtable.com/{baseId}/{tableId}` |
| `tableName` | `string` | ✅ Yes | Name of the table to operate on (must match exactly as shown in the UI) |

### Operation Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | ✅ Yes | Operation to perform (see below) |
| `recordId` | `string` | Conditional | Airtable record ID (`recXXXXXXXXXXXXXX`) — required for Update, Delete, and Read |
| `recordData` | `array` | Conditional | Field values as `[{ fieldName, fieldValue }]` — required for Append and Update |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `Append data to table` | Create a new record in the table |
| `Update data in table` | Update an existing record by `recordId` |
| `Delete data from table` | Delete a record by `recordId` |
| `Read data from table` | Read a specific record by `recordId`, or all records if `recordId` is omitted |
| `List data from table` | Retrieve all records with a total count |

> **Append vs. List vs. Read:** `Append data to table` creates records. `Read data from table` returns one or all records as a flat array. `List data from table` returns all records wrapped in a `{ count, results }` object.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data — available via expressions in `recordId` and field values |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object \| array` | Result of the executed operation |
| `error` | `Error` | Emitted if the Airtable API call fails |

### Output Schemas by Operation

**`Append data to table`**

Returns an array with the created record (Airtable supports batch create, so the result is always an array):
```json
[
  {
    "recordId": "recABC123XYZ",
    "Name": "Alice",
    "Status": "Active",
    "Score": 42
  }
]
```

**`Update data in table`**

Returns the updated record as a flat object:
```json
{
  "recordId": "recABC123XYZ",
  "Name": "Alice",
  "Status": "Inactive",
  "Score": 42
}
```

**`Delete data from table`**
```json
{
  "recordId": "recABC123XYZ",
  "deleted": true
}
```

**`Read data from table` (single record)**
```json
{
  "recordId": "recABC123XYZ",
  "Name": "Alice",
  "Status": "Active",
  "Score": 42
}
```

**`Read data from table` (all records — when `recordId` is omitted)**
```json
[
  { "recordId": "recABC123XYZ", "Name": "Alice", "Status": "Active" },
  { "recordId": "recDEF456UVW", "Name": "Bob", "Status": "Inactive" }
]
```

**`List data from table`**
```json
{
  "count": 2,
  "results": [
    { "recordId": "recABC123XYZ", "Name": "Alice", "Status": "Active" },
    { "recordId": "recDEF456UVW", "Name": "Bob", "Status": "Inactive" }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Append a New Record

Create a new lead in an Airtable CRM table from a webhook payload.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Leads",
  "operation": "Append data to table",
  "recordData": [
    { "fieldName": "Name", "fieldValue": "{{input.name}}" },
    { "fieldName": "Email", "fieldValue": "{{input.email}}" },
    { "fieldName": "Source", "fieldValue": "Webhook" },
    { "fieldName": "Score", "fieldValue": "{{input.score}}" }
  ]
}
```

**Output:**
```json
[
  {
    "recordId": "recABC123XYZ",
    "Name": "Alice",
    "Email": "alice@example.com",
    "Source": "Webhook",
    "Score": 85
  }
]
```

---

### Example: Update a Record

Mark a lead as converted after a sale.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Leads",
  "operation": "Update data in table",
  "recordId": "{{input.recordId}}",
  "recordData": [
    { "fieldName": "Status", "fieldValue": "Converted" },
    { "fieldName": "Converted Date", "fieldValue": "{{new Date().toISOString().slice(0,10)}}" }
  ]
}
```

---

### Example: Read a Record by ID

Retrieve a specific record to use in downstream nodes.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Leads",
  "operation": "Read data from table",
  "recordId": "recABC123XYZ"
}
```

**Output:**
```json
{
  "recordId": "recABC123XYZ",
  "Name": "Alice",
  "Email": "alice@example.com",
  "Status": "Active",
  "Score": 85
}
```

---

### Example: List All Records

Retrieve all records from the table for batch processing.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Leads",
  "operation": "List data from table"
}
```

**Output:**
```json
{
  "count": 3,
  "results": [
    { "recordId": "recABC123XYZ", "Name": "Alice", "Status": "Active" },
    { "recordId": "recDEF456UVW", "Name": "Bob", "Status": "Inactive" },
    { "recordId": "recGHI789RST", "Name": "Carol", "Status": "Active" }
  ]
}
```

---

### Example: Delete a Record

Remove a record from the table.

**Configuration:**
```json
{
  "apiKey": "patXXXXXXXXXXXXXX",
  "baseId": "appXXXXXXXXXXXXXX",
  "tableName": "Leads",
  "operation": "Delete data from table",
  "recordId": "recABC123XYZ"
}
```

**Output:**
```json
{
  "recordId": "recABC123XYZ",
  "deleted": true
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Sync External API Data to Airtable

Fetch product data from an API every hour and append new records to Airtable.

```json
{
  "nodes": [
    {
      "id": "hourly-trigger",
      "type": "cron",
      "position": { "x": 100, "y": 100 },
      "config": { "cronExpression": "0 * * * *" }
    },
    {
      "id": "fetch-products",
      "type": "http-request",
      "position": { "x": 300, "y": 100 },
      "config": {
        "url": "https://api.example.com/products/new",
        "method": "GET"
      }
    },
    {
      "id": "save-to-airtable",
      "type": "airtable-action",
      "position": { "x": 500, "y": 100 },
      "config": {
        "apiKey": "patXXXXXXXXXXXXXX",
        "baseId": "appXXXXXXXXXXXXXX",
        "tableName": "Products",
        "operation": "Append data to table",
        "recordData": [
          { "fieldName": "SKU", "fieldValue": "{{input.sku}}" },
          { "fieldName": "Name", "fieldValue": "{{input.name}}" },
          { "fieldName": "Price", "fieldValue": "{{input.price}}" }
        ]
      }
    }
  ]
}
```

### Common Patterns

- **Form → Airtable:** Webhook trigger → Airtable Append (log submissions)
- **Status Update:** [Airtable Trigger](./airtable-trigger.md) detects new record → Airtable Update (set status)
- **Read → Decision:** Airtable Read → Function (check field value) → Branch
- **Batch Processing:** Airtable List → Function (loop items) → External API per record
- **Cleanup:** Cron → Airtable List → Airtable Delete per old record

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Airtable API error: 401`

**Cause:** The `apiKey` is invalid, expired, or missing the required scopes.

**Solution:** Generate a new personal access token at **airtable.com/create/tokens**. Ensure the token includes the `data.records:read` and `data.records:write` scopes for the target base.

#### `Airtable API error: 404 — Table not found`

**Cause:** The `tableName` does not match any table in the base, or the `baseId` is wrong.

**Solution:** Verify both `baseId` and `tableName` exactly as they appear in Airtable. Both are case-sensitive. The `baseId` starts with `app` and the record IDs start with `rec`.

#### `recordId is required for update operation`

**Cause:** The `Update data in table` operation was selected but `recordId` was not provided.

**Solution:** Provide the `recordId` field. Use a prior `Read data from table` or `List data from table` operation to obtain it, then pass it via expression: `{{input.recordId}}`.

#### `At least one field is required` on Append / Update

**Cause:** The `recordData` array is empty.

**Solution:** Provide at least one `{ fieldName, fieldValue }` pair. Field names must match the exact column names in Airtable (case-sensitive).

#### String value stored instead of expected type (number, date, etc.)

**Cause:** Although `typecast` is enabled, some complex field types (linked records, attachments) may not cast correctly from plain strings.

**Solution:** For simple field types (number, date, single select), typecast handles the conversion automatically. For complex types, use the Airtable API's native format for those field values.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `401` | Unauthorized | Check API token and scopes |
| `404` | Not found | Verify `baseId` and `tableName` |
| `422` | Unprocessable entity | Invalid field name or value format |
| `429` | Rate limited | Reduce request frequency — Airtable limits to 5 req/sec per base |
| `ECONNREFUSED` | Connection refused | Check network access to `api.airtable.com` |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Airtable Trigger](./airtable-trigger.md) - Trigger workflows on Airtable record changes
- [Baserow Action](./baserow.md) - Similar operations on self-hosted Baserow instances
- [Function](./function.md) - Transform data before appending or updating records

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-03-10 | Initial release |

<!-- /SECTION: changelog -->
