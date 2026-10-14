---
node_id: "filemaker-action"
title: "FileMaker"
description: "Interact with Claris FileMaker databases via the FileMaker Data API v2. Supports CRUD operations on records, find queries, script execution, and layout metadata."
category: "data"
subcategory: "databases"
version: "1.0.0"
language: "en"
last_updated: "2026-10-14"
author: "Fusion Team"
tags:
  - filemaker
  - claris
  - database
  - data-api
  - crm
  - action
related_nodes:
  - airtable-action
  - mysql-action
  - function
---

<!-- SECTION: header -->
# FileMaker

> **Category:** Data&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Connect to a Claris FileMaker Server or FileMaker Cloud instance and automate operations using the official FileMaker Data API (v2). Supports full record lifecycle management (Find, Get, Create, Edit, Duplicate, Delete), automated script execution, and layout schema inspection.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **FileMaker** node integrates your workflows with Claris FileMaker databases. It communicates over HTTPS using the FileMaker Data API v2, managing session creation, authentication tokens, and automatic session closure.

All record operations in FileMaker target a specific **Layout** rather than raw database tables, ensuring data validation, security privileges, and business calculations defined in FileMaker Pro are strictly respected.

### Key Capabilities

- **Find Records (`findRecords`)**: Perform multi-criteria searches with FileMaker operators (`*`, `>`, `<`, `=`, `!`), pagination, and field sorting.
- **Get Records (`listRecords`)**: Retrieve paginated lists of records from any accessible layout with `limit` and `offset`.
- **Get Record by ID (`getRecord`)**: Fetch complete field data for a single record by its internal FileMaker Record ID.
- **Create Record (`createRecord`)**: Insert new records by passing field key-value pairs (`fieldData`).
- **Edit Record (`updateRecord`)**: Modify existing records by ID with partial or full field updates.
- **Duplicate Record (`duplicateRecord`)**: Clone an existing record with its relations and calculations preserved.
- **Delete Record (`deleteRecord`)**: Remove a record permanently from the target layout by ID.
- **Perform Script (`executeScript`)**: Trigger server-side FileMaker scripts with optional parameters and capture the script result.
- **Layout Discovery (`getLayouts`, `getLayout`)**: Dynamically list all layouts or inspect field metadata for form building.

### Common Use Cases

- **CRM Synchronization**: Sync new leads, customers, or form submissions from webhooks into a FileMaker CRM.
- **Inventory & Order Management**: Read inventory levels from FileMaker and update stock status during e-commerce fulfillment.
- **Automated Reporting & Scripts**: Trigger periodic FileMaker report generation scripts and capture status codes.
- **Cross-Platform ETL**: Query records matching specific criteria in FileMaker and replicate them into PostgreSQL, Airtable, or Snowflake.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

The following parameters are required to establish a secure session with FileMaker Server:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `host` | `string` | ✅ Yes | Full URL of the FileMaker Server (e.g., `https://filemaker.example.com`). Must include protocol, no trailing slash. |
| `database` | `string` | ✅ Yes | Exact name of the FileMaker database file (e.g., `Customers` or `Sales`). |
| `username` | `string` | ✅ Yes | FileMaker account name with the `fmrest` extended privilege enabled. |
| `password` | `string` | ✅ Yes | Password for the FileMaker account. |

### Operation Selector

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `listRecords` | Selects the operation to perform across records, scripts, or layouts. |

### Available Operations

| Operation | Target | Description |
|-----------|--------|-------------|
| `listRecords` | Layout | Retrieve a paginated list of records from the layout |
| `getRecord` | Layout + Record ID | Retrieve a specific record by its internal numeric ID |
| `createRecord` | Layout + Record Data | Insert a new record with specified field values |
| `updateRecord` | Layout + Record ID + Record Data | Update field values of an existing record |
| `duplicateRecord` | Layout + Record ID | Duplicate an existing record |
| `deleteRecord` | Layout + Record ID | Delete a record by ID |
| `findRecords` | Layout + Find Parameters | Search records using criteria JSON, pagination, and sorting |
| `executeScript` | Layout + Script Name | Execute a FileMaker script on the server |
| `getLayouts` | Database | Retrieve all layout names available in the database |
| `getLayout` | Layout | Retrieve metadata and field definitions for a layout |

### Operation-Specific Parameters

#### Shared Parameters

- **`layout`** (`string`, Conditional): The layout name. Required for all operations except `getLayouts`.
- **`recordId`** (`string`, Conditional): Numeric record ID. Required for `getRecord`, `updateRecord`, `duplicateRecord`, and `deleteRecord`.

#### Record Data (`recordData`)
Required for `createRecord` and `updateRecord`. Defined as an object with a `fields` map:
```json
{
  "fields": {
    "FirstName": "Alice",
    "LastName": "Dupont",
    "Email": "alice@example.com",
    "Phone": "+33612345678"
  }
}
```

#### Find Parameters (`findParams`)
Required for `findRecords`. Contains the search query, limit, offset, and sort rules:
- **`query`** (`string`, Required): JSON array of criteria objects. Supports wildcards (`*`), comparisons (`>`, `<`), and exact matches (`=`).
  ```json
  [{"LastName": "Dup*", "City": "Paris"}]
  ```
- **`limit`** (`number`, Optional): Maximum matching records to return (1–1000).
- **`offset`** (`number`, Optional): Starting position in the found set (1-based).
- **`sort`** (`string`, Optional): JSON array of sort rules:
  ```json
  [{"fieldName": "LastName", "sortOrder": "ascend"}]
  ```

#### Script Parameters (`scriptParams`)
Required for `executeScript`:
- **`scriptName`** (`string`, Required): Name of the script to execute as defined in FileMaker Pro.
- **`scriptParam`** (`string`, Optional): String parameter passed to `Get(ScriptParameter)` inside the script.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data used dynamically in expressions (e.g., `{{input.recordId}}` or `{{input.email}}`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `output` | `object` | JSON response payload returned by the FileMaker Data API. |
| `error` | `Error` | Emitted when authentication fails, records are not found, or script errors occur. |

### Output Schemas by Operation

#### `listRecords` & `findRecords`
```json
{
  "response": {
    "dataInfo": {
      "database": "Customers",
      "layout": "Contacts",
      "table": "Contacts",
      "totalRecordCount": 150,
      "foundCount": 2,
      "returnedCount": 2
    },
    "data": [
      {
        "recordId": "1",
        "modId": "0",
        "fieldData": {
          "FirstName": "Alice",
          "LastName": "Dupont",
          "Email": "alice@example.com"
        }
      },
      {
        "recordId": "2",
        "modId": "0",
        "fieldData": {
          "FirstName": "Bob",
          "LastName": "Martin",
          "Email": "bob@example.com"
        }
      }
    ]
  },
  "messages": [
    {
      "code": "0",
      "message": "OK"
    }
  ]
}
```

#### `getRecord`
```json
{
  "response": {
    "data": [
      {
        "recordId": "1",
        "modId": "0",
        "fieldData": {
          "FirstName": "Alice",
          "LastName": "Dupont",
          "Email": "alice@example.com"
        }
      }
    ]
  },
  "messages": [
    {
      "code": "0",
      "message": "OK"
    }
  ]
}
```

#### `createRecord`
```json
{
  "response": {
    "recordId": "5089",
    "modId": "0"
  },
  "messages": [
    {
      "code": "0",
      "message": "OK"
    }
  ]
}
```

#### `updateRecord`
```json
{
  "response": {
    "modId": "1"
  },
  "messages": [
    {
      "code": "0",
      "message": "OK"
    }
  ]
}
```

#### `deleteRecord`
```json
{
  "success": true,
  "recordId": "1"
}
```

#### `executeScript`
```json
{
  "response": {
    "scriptResult": "0",
    "scriptError": "0"
  },
  "messages": [
    {
      "code": "0",
      "message": "OK"
    }
  ]
}
```

#### `getLayouts`
```json
{
  "response": {
    "layouts": [
      { "name": "Contacts" },
      { "name": "Invoices" },
      { "name": "Reports" }
    ]
  },
  "messages": [
    {
      "code": "0",
      "message": "OK"
    }
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Example Workflow

### Query and Process FileMaker Records

This example queries active customer records from FileMaker, logs the results, and prepares them for downstream automation.

```fusion-workflow
src: example.workflow.json
title: Query and inspect FileMaker records
```

### Typical Workflow Pattern

1. **Trigger**: An automated scheduler (Cron) or Webhook fires.
2. **FileMaker Node**: Runs `findRecords` on layout `Contacts` with query `[{"Status": "Active"}]`.
3. **Downstream Action**: A Function or CRM node iterates through the `response.data` array to process each record.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `FileMaker login failed (401): Unauthorized`
- **Cause:** Invalid username or password, or the account does not have the `fmrest` extended privilege enabled.
- **Solution:** Open the database file in FileMaker Pro, navigate to **File > Manage > Security**, select the Privilege Set assigned to the account, and ensure **Access via FileMaker Data API (fmrest)** is checked.

#### `FileMaker API error (404): Layout not found`
- **Cause:** The layout name in `layout` does not match any layout in the database or is misspelled (names are case-sensitive).
- **Solution:** Run the `getLayouts` operation to view all exact layout names available to your account.

#### `Layout Name is required` or `Record ID is required`
- **Cause:** The selected operation requires a layout or record ID that was omitted.
- **Solution:** Ensure `layout` is filled for all record/find/script operations, and provide `recordId` for `getRecord`, `updateRecord`, `duplicateRecord`, or `deleteRecord`.

#### `FileMaker operation failed: FetchError / ECONNREFUSED`
- **Cause:** FileMaker Server is offline, ports `80`/`443` are blocked by a firewall, or the Data API is disabled in Admin Console.
- **Solution:** Verify FileMaker Server Admin Console at `https://<host>/admin-console/`, navigate to **Connectors > FileMaker Data API**, and verify the toggle is **Enabled (ON)**.

### Error Codes Reference

| Error Code | Meaning | Resolution |
|------------|---------|------------|
| `0` | OK | Operation succeeded |
| `102` | Field missing | Verify field names match the layout fields |
| `105` | Layout missing | Verify the layout name is published and exact |
| `212` | Invalid user/password | Check credentials in FileMaker Security |
| `401` | No records match request | Find query returned empty set; check search criteria |
| `802` | Unable to open file | Check database status in FileMaker Server Admin Console |
| `952` | Data API token expired | Node automatically re-authenticates on next call |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Airtable Action](../airtable-action/en.md) - Cloud-based relational spreadsheet and database actions
- [MySQL Action](../mysql-action/en.md) - Execute queries against relational MySQL databases
- [Function](../function/en.md) - Transform and manipulate FileMaker record data in JavaScript

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-14 | Initial release with full FileMaker Data API v2 CRUD, Find, Script, and Layout support |

<!-- /SECTION: changelog -->
