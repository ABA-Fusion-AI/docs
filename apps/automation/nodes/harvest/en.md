---
node_id: "harvest"

title: "Harvest"

description: "Track time, manage projects, clients, tasks, and invoices with Harvest."

category: "Productivity / Time Tracking"

version: "1.0.0"

language: "en"

last_updated: "2026-09-21"

author: "Fusion Team"

tags:

- harvest

- time-tracking

- projects

- tasks

- clients

- invoices

related_nodes:

- http-request

- function

- if

---

**# Harvest**

> **Category:** productivity-nodes | **Type:** Action Node

Connect workflows to Harvest for project management, time tracking, tasks, clients, and invoices.

The **Harvest** node supports retrieving Harvest resources and creating time entries through the Harvest API v2 using an access token and account ID.

**### Supported Features**

- List projects
- Retrieve a specific project
- List time entries
- Create time entries
- List tasks
- List clients
- Retrieve a specific client
- List invoices
- Retrieve a specific invoice
- Configure the number of returned results
- Authenticate using a Harvest access token and account ID
- Return parsed Harvest API responses directly

**### Use Cases**

- Retrieve projects inside workflows
- Track employee or project time
- Create Harvest time entries automatically
- Retrieve tasks and clients
- Retrieve invoices for downstream processing
- Connect Harvest data to other workflow nodes

**---**

**## Configuration**

**### Base Parameters**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `accessToken` | `string` | ✅ Yes | — | Harvest access token. Supports expressions. |
| `accountId` | `string` | ✅ Yes | — | Harvest account ID. Supports expressions. |
| `operation` | `enum` | ❌ No | `"listProjects"` | Operation to perform. |
| `id` | `string` | ❌ No | — | Resource ID used by get operations. |
| `limit` | `number` | ❌ No | `20` | Number of results requested per page. |
| `projectId` | `string` | ❌ No | — | Project ID for a time entry. |
| `taskId` | `string` | ❌ No | — | Task ID for a time entry. |
| `spentDate` | `string` | ❌ No | — | Date for the time entry. |
| `hours` | `number` | ❌ No | — | Hours for the time entry. Runtime fallback is `1`. |
| `notes` | `string` | ❌ No | — | Time-entry notes. Runtime fallback is an empty string. |

**### Supported Operations**

```text
listProjects
getProject
listTimeEntries
createTimeEntry
listTasks
listClients
getClient
listInvoices
getInvoice
```

Default operation:

```text
listProjects
```

**---**

**## Operations**

**### listProjects**

Request:

```text
GET /projects?per_page=<limit>
```

Default `limit`: `20`.

---

**### getProject**

Required:

```text
id
```

Request:

```text
GET /projects/<id>
```

---

**### listTimeEntries**

Request:

```text
GET /time_entries?per_page=<limit>
```

---

**### createTimeEntry**

Required:

```text
projectId
taskId
spentDate
```

Request:

```text
POST /time_entries
```

Body:

```json
{
  "project_id": "PROJECT_ID",
  "task_id": "TASK_ID",
  "spent_date": "2026-09-21",
  "hours": 1,
  "notes": ""
}
```

`hours` defaults to `1` and `notes` defaults to an empty string at runtime.

---

**### listTasks**

```text
GET /tasks?per_page=<limit>
```

---

**### listClients**

```text
GET /clients?per_page=<limit>
```

---

**### getClient**

Required: `id`

```text
GET /clients/<id>
```

---

**### listInvoices**

```text
GET /invoices?per_page=<limit>
```

---

**### getInvoice**

Required: `id`

```text
GET /invoices/<id>
```

**---**

**## Request Construction**

Base URL:

```text
https://api.harvestapp.com/v2
```

Headers:

```text
Authorization: Bearer <accessToken>
Harvest-Account-Id: <accountId>
Content-Type: application/json
User-Agent: Fusion Workflow
```

Request bodies are serialized using:

```ts
JSON.stringify(body)
```

List operations use `per_page=<limit>` with a default of `20`.

**---**

**## Inputs & Outputs**

**### Inputs**

`handleTick()` receives `_incomingData`, but the implementation does not use it. Request values come from node configuration.

**### Outputs**

The response is parsed with:

```ts
await res.json()
```

Successful responses are returned directly.

**### Output Example**

```json
{
  "...": "raw Harvest API response"
}
```

**---**

**## Configuration Examples**

**### List Projects**

```json
{
  "operation": "listProjects",
  "accessToken": "YOUR_ACCESS_TOKEN",
  "accountId": "YOUR_ACCOUNT_ID",
  "limit": 20
}
```

**### Create Time Entry**

```json
{
  "operation": "createTimeEntry",
  "accessToken": "YOUR_ACCESS_TOKEN",
  "accountId": "YOUR_ACCOUNT_ID",
  "projectId": "PROJECT_ID",
  "taskId": "TASK_ID",
  "spentDate": "2026-09-21",
  "hours": 2,
  "notes": "Development work"
}
```

**### Get Client**

```json
{
  "operation": "getClient",
  "accessToken": "YOUR_ACCESS_TOKEN",
  "accountId": "YOUR_ACCOUNT_ID",
  "id": "CLIENT_ID"
}
```

**### Get Invoice**

```json
{
  "operation": "getInvoice",
  "accessToken": "YOUR_ACCESS_TOKEN",
  "accountId": "YOUR_ACCOUNT_ID",
  "id": "INVOICE_ID"
}
```

<!-- SECTION: examples -->

**## Example Workflow**

```fusion-workflow
src: example.workflow.json
title: Use Harvest in a workflow
```

<!-- /SECTION: examples -->

**---**

**## Workflow Integration**

**### Common Patterns**

- Trigger → Harvest (`createTimeEntry`)
- Harvest (`listProjects`) → Function
- Harvest (`listTimeEntries`) → Data Processing
- Harvest (`listClients`) → CRM Processing
- Harvest (`listInvoices`) → Finance Processing
- Harvest (`getInvoice`) → Notification

**---**

**## Error Handling**

The node parses the response as JSON before checking the HTTP status.

For unsuccessful responses:

```text
Harvest error (<status>): <JSON response>
```

Validation errors:

```text
id is required
projectId is required
taskId is required
spentDate is required
Unknown operation: <operation>
```

`id is required` applies to `getProject`, `getClient`, and `getInvoice`.

**---**

**## Troubleshooting**

**### Authentication Fails**

Verify `accessToken` and `accountId`.

**### Time Entry Cannot Be Created**

Verify `projectId`, `taskId`, and `spentDate`.

**### Wrong Number of Results**

Verify `limit`. The node sends it as `per_page=<limit>`, with a default of `20`.

**### JSON Parsing Error**

The implementation calls `await res.json()` before checking `res.ok`. A non-JSON response can therefore fail during parsing before the custom Harvest error is thrown.

**### Incoming Data Is Ignored**

`_incomingData` is accepted by `handleTick()` but is not used.

**---**

**## Security**

The node uses a Harvest access token and account ID.

For production workflows:

- Store the access token securely
- Never commit real Harvest credentials to Git
- Do not include real credentials in workflow examples
- Avoid logging the `Authorization` header
- Rotate exposed access tokens

**---**

**## Notes**

Metadata label:

```text
Harvest
```

Metadata description:

```text
Track time, manage projects, and handle invoices with Harvest.
```

API base:

```text
https://api.harvestapp.com/v2
```

Default operation: `listProjects`

Default limit: `20`

The node does not implement retries, caching, response transformation, or pagination beyond setting `per_page`.

The `stop()` method performs no cleanup logic.

**---**

**## Changelog**

| Version | Date | Changes |
| --- | --- | --- |
| `1.0.0` | `2026-09-21` | Initial release |
