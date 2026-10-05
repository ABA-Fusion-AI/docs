---
node_id: "seatable"
title: "SeaTable"
description: "Interact with SeaTable databases. Supports creating, reading, updating, and deleting rows in tables."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-05"
author: "Fusion Team"
tags:
  - seatable
  - database
  - no-code
  - spreadsheet
  - integration
  - peer-only
related_nodes:
  - airtable
  - http-request
  - function
---

<!-- SECTION: overview -->
# SeaTable

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read and write rows in a [SeaTable](https://seatable.com) base. The node authenticates with a **base API token**, exchanges it for a short-lived base access token, and then calls the SeaTable rows API of that base. It works with SeaTable Cloud (`https://cloud.seatable.io`) and self-hosted SeaTable servers.

### Use Cases

- **Store workflow results:** Insert a row for each order, form submission, or event produced earlier in the workflow.
- **Read a table:** Load all rows of a table and process them in downstream nodes (Function, If/Else, notifications).
- **Look up a record:** Fetch a single row by its `_id`.
- **Keep records in sync:** Update or delete a row when the matching record changes in another system.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `server` | `url` | No | `https://cloud.seatable.io` | SeaTable server URL. Change it only for a self-hosted SeaTable. |
| `apiToken` | `string` | Yes | — | **Base** API token (not the Account-Token). See [Getting an API Token](#getting-an-api-token). |
| `operation` | `enum` | No | `Get All Rows` | `Create Row`, `Delete Row`, `Get Row`, `Get All Rows`, or `Update Row`. |
| `tableId` | `string` | Yes | — | **Name** of the table in the base, e.g. `Table1` (sent to SeaTable as `table_name`). |
| `rowId` | `string` | No | — | Row `_id`. Required for `Get Row`, `Update Row`, and `Delete Row` (can also come from the incoming data). |

### Available Operations

| Operation | Description | Row data | Required |
|-----------|-------------|----------|----------|
| `Get All Rows` | Returns the rows of the table. | — | `tableId` |
| `Get Row` | Returns one row by its `_id`. | — | `tableId`, `rowId` |
| `Create Row` | Inserts one row per incoming object (or one per item if the input is an array). | **Incoming data** | `tableId` |
| `Update Row` | Updates one row with the fields of the incoming object. | **Incoming data** | `tableId`, `rowId` |
| `Delete Row` | Deletes one row. | — | `tableId`, `rowId` |

### How Row Data Is Provided

The node has no field for row values. `Create Row` and `Update Row` use the **data coming from the previous node** as the row:

- The keys of the incoming object must match **column names** of the table (e.g. `{ "Name": "Alice", "Amount": 12 }`).
- Keys that do not match a column are ignored by SeaTable. If no key matches, no row is inserted (`inserted_row_count: 0`), but the node still returns `success: true`.
- For `Create Row`, an incoming **array** of objects inserts several rows at once.

Use a Function node (or any node that outputs an object) before SeaTable to build the row.

### Dynamic Values from the Incoming Data

The incoming data can override the configuration for one execution:

| Incoming key | Overrides |
|--------------|-----------|
| `operation` | `operation` |
| `tableId` | `tableId` |
| `rowId` or `_id` | `rowId` |

### Getting an API Token

1. Sign in to SeaTable and open the **home page**.
2. Hover over the base, click its **options menu (•••)** → **Advanced** → **API Tokens**.
3. Enter a name, choose **Read-Write** permission, and generate the token.
4. Paste it into `apiToken`.

> The **Account-Token** shown under *Personal settings* does **not** work with this node — it needs a base API token.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` \| `array` | Row data for `Create Row` / `Update Row`, and optional overrides (`operation`, `tableId`, `rowId` / `_id`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The result of the operation. **Errors are also returned here** with `success: false` (see [Error Handling](#error-handling)). |
| `error` | `Error` | Not used for operation errors (the node catches them and returns them on `success`). |

### Output Examples

#### `Get All Rows`

```json
{
  "success": true,
  "rows": {
    "rows": [
      {
        "_id": "JHn4y4zRRcutPhFKmbX1ew",
        "Name": "Alice",
        "tick": 1,
        "_ctime": "2026-10-05T18:20:32.713+02:00",
        "_mtime": "2026-10-05T18:20:32.713+02:00",
        "_creator": "user@auth.local",
        "_archived": false
      }
    ]
  }
}
```

#### `Get Row`

```json
{
  "success": true,
  "row": {
    "_id": "F_x12W-vQgOLPTyMNbk0gQ",
    "Name": null,
    "tick": 1,
    "_ctime": "2026-10-05T18:30:36.101+02:00",
    "_mtime": "2026-10-05T18:30:36.101+02:00"
  }
}
```

#### `Create Row`

```json
{
  "success": true,
  "records": {
    "inserted_row_count": 1,
    "row_ids": [{ "_id": "F_x12W-vQgOLPTyMNbk0gQ" }],
    "first_row": { "_id": "F_x12W-vQgOLPTyMNbk0gQ", "_ctime": "2026-10-05T16:30:36.101+00:00" }
  },
  "count": 1
}
```

`count` is the number of objects sent; check `records.inserted_row_count` for the number of rows actually inserted.

#### `Update Row`

```json
{ "success": true, "rowId": "F_x12W-vQgOLPTyMNbk0gQ", "result": { "success": true } }
```

#### `Delete Row`

```json
{ "success": true, "rowId": "F_x12W-vQgOLPTyMNbk0gQ", "result": { "deleted_rows": 1 } }
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use SeaTable in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **SeaTable Node:** Runs `Get All Rows` on `Table1`.
3. **Log Node:** Displays the rows.

### Example Configurations

#### Read all rows

```json
{
  "server": "https://cloud.seatable.io",
  "apiToken": "{{ secrets.SEATABLE_TOKEN }}",
  "operation": "Get All Rows",
  "tableId": "Table1"
}
```

#### Insert a row built by a previous node

Previous node output:

```json
{ "Name": "Alice", "Amount": 12 }
```

SeaTable configuration:

```json
{
  "apiToken": "{{ secrets.SEATABLE_TOKEN }}",
  "operation": "Create Row",
  "tableId": "Table1"
}
```

#### Delete a row

```json
{
  "apiToken": "{{ secrets.SEATABLE_TOKEN }}",
  "operation": "Delete Row",
  "tableId": "Table1",
  "rowId": "F_x12W-vQgOLPTyMNbk0gQ"
}
```

### Common Patterns

- **Webhook → Function → SeaTable (`Create Row`):** Map an incoming request to column names and store it.
- **Cron → SeaTable (`Get All Rows`) → Function → Notification:** Send a daily summary of a table.
- **SeaTable (`Get Row`) → Function → SeaTable (`Update Row`):** Read a record, change some fields, write it back.
- **SeaTable → If/Else on `success`:** Because errors arrive on the success output, branch on `success === false` to handle failures.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `apiToken` in Fusion's **Secrets** system. Never paste tokens directly into workflow parameters or commit them to version control.

- A base API token has **no expiration date** — treat it like a password and delete it in SeaTable when it is no longer needed.
- Give the token the lowest permission that works (**Read-Only** for workflows that only read).
- Use one token per base and per workflow so it can be revoked individually.
- The node exchanges the API token for a base access token valid for 3 days and keeps it while the workflow runs.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Error Handling

The node catches all operation errors and returns them on the **success** output:

```json
{
  "success": false,
  "error": "Seatable API error: 404 {\"error_message\":\"table name [NoSuchTable] or id [] not exists in base ...\"}",
  "operation": "Get All Rows",
  "tableId": "NoSuchTable"
}
```

Check `success` in the next node to detect failures.

### Common Issues

#### `Failed to authenticate: 401 …` / `403 …`
- **Cause:** The token is wrong, deleted, or is an **Account-Token** instead of a base API token.
- **Solution:** Create a base API token from the base's **Advanced → API Tokens** menu.

#### `table name [X] or id [] not exists in base …`
- **Cause:** `tableId` does not match a table name (names are case-sensitive).
- **Solution:** Use the exact table name shown in the tab, e.g. `Table1`.

#### `Row ID is required for … operation`
- **Cause:** `Get Row`, `Update Row`, or `Delete Row` without `rowId` (and no `_id` / `rowId` in the incoming data).
- **Solution:** Set `rowId`, or pass the row's `_id` from a previous `Get All Rows` / `Create Row`.

#### `Create Row` returns `success: true` but no row appears
- **Cause:** None of the incoming keys matches a column name (`records.inserted_row_count` is `0`).
- **Solution:** Make the previous node output an object whose keys are the table's column names.

#### `Data must be a non-array object for Update Row operation`
- **Cause:** The incoming data for `Update Row` is an array or empty.
- **Solution:** Send a single object with the fields to change.

#### Too many requests / quota exceeded
- **Cause:** SeaTable plans limit API calls (the Free plan allows 3,000 calls per month).
- **Solution:** Reduce how often the workflow runs or upgrade the SeaTable plan.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-05 | Initial documentation |

<!-- /SECTION: changelog -->