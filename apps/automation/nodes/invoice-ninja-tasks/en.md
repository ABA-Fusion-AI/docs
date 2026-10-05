---
node_id: "invoice-ninja-tasks"
title: "Invoice Ninja Tasks"
description: "Track project tasks, time logs, hourly billable work, and project progress in Invoice Ninja."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-03"
author: "Fusion Team"
tags:
  - invoice-ninja
  - tasks
  - time-tracking
  - projects
  - billing
  - peer-only
  - action
related_nodes:
  - invoice-ninja-invoices
  - invoice-ninja-quotes
  - invoice-ninja-clients
  - http-request
---

<!-- SECTION: overview -->
# Invoice Ninja Tasks

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Track project tasks, time logs, hourly billable work, and project progress in Invoice Ninja.

The **Invoice Ninja Tasks** node connects Fusion workflows directly to Invoice Ninja (both hosted SaaS and self-hosted instances). It provides production-ready operations with automated token authentication, URL normalization, robust input validation, and structured error handling.

### Key Features

- **Time & Duration Logging:** Track exact task duration in seconds with active running states
- **Hourly Billing Rates:** Specify custom billing rates per task or project
- **Project & Client Association:** Link tasks directly to clients and project IDs
- **Comprehensive Task Search:** Query tasks by client, project, status, or category

### Use Cases

- Sync task progress and billable time logs from Jira, GitHub, or Linear into Invoice Ninja
- Record billable project hours with custom hourly rates for automated time-and-materials invoicing
- Query all open tasks for a client or project to generate executive status reports
- Start or stop running task timers in response to external automation events
<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Connection Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `baseUrl` | `string` | ❌ No | `https://invoicing.co` | Root URL of your Invoice Ninja deployment. Supports SaaS (`https://invoicing.co`) or self-hosted servers (e.g. `https://ninja.example.com`). Trailing slashes and `/api/v1` path segments are automatically normalized. |
| `apiToken` | `string` | ✅ Yes | — | API Token generated from your Invoice Ninja Admin Panel (**Settings &gt; Account Management &gt; API Tokens**). |
| `operation` | `enum` | ✅ Yes | — | The operation to perform (see list below). |

### Available Operations

| Operation | Description |
|-----------|-------------|
| `create` | Create a task with duration, hourly billing rate, client, project, and timer state |
| `get` | Retrieve full task details, tracked time, and status |
| `getAll` | List tasks with optional filters for client, status, vendor, category, or project |
| `delete` | Delete a task record by ID |

### Operation Parameters

### Operation: `create`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ❌ No | — | Associated client ID. |
| `projectId` | `string` | ❌ No | — | Associated project ID. |
| `description` | `string` | ❌ No | — | Description or work summary of the task. |
| `duration` | `number` | ❌ No | — | Tracked task duration in seconds. |
| `isRunning` | `boolean` | ❌ No | `false` | Set to `true` if the timer is actively running, or `false` if stopped. |
| `rate` | `number` | ❌ No | — | Hourly billing rate for the task. |
| `customValue1` | `string` | ❌ No | — | Custom field value 1. |
| `customValue2` | `string` | ❌ No | — | Custom field value 2. |

### Operation: `get` & `delete`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | `string` | ✅ Yes | — | The unique Invoice Ninja task ID. |

### Operation: `getAll`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ❌ No | — | Filter tasks by client ID. |
| `statusId` | `string` | ❌ No | — | Filter tasks by status ID. |
| `vendorId` | `string` | ❌ No | — | Filter tasks by vendor ID. |
| `categoryId` | `string` | ❌ No | — | Filter tasks by category ID. |
| `projectId` | `string` | ❌ No | — | Filter tasks by project ID. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Payload passed from the upstream node containing configuration parameters or dynamic query overrides. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the Invoice Ninja API returns a successful response (`200 OK`, `201 Created`, or `204 No Content`). |
| `error` | `Error` | Emitted when validation fails, network issues occur, or the API returns an error response (4xx/5xx). |

### Output Schema

When an operation succeeds, the response data from Invoice Ninja is emitted through the `success` port:

```json
{
  "data": {
    "id": "VolejRejNm",
    "created_at": 1727974800,
    "updated_at": 1727974800
  }
}
```

For `getAll` operations, an array of matching records is returned under the `data` key:

```json
{
  "data": [
    {
      "id": "VolejRejNm",
      "created_at": 1727974800
    }
  ]
}
```

---

### Examples

#### Example: `create`

**Configuration:**
```json
{
  "operation": "create",
  "clientId": "VolejRejNm",
  "description": "API Integration and Automated Testing",
  "duration": 7200,
  "rate": 95
}
```

**Output (`success`):**
```json
{
  "data": {
    "id": "Tsk9981aB",
    "client_id": "VolejRejNm",
    "description": "API Integration and Automated Testing",
    "duration": 7200,
    "rate": 95,
    "created_at": 1727974800
  }
}
```

#### Example: `get`

**Configuration:**
```json
{
  "operation": "get",
  "id": "VolejRejNm"
}
```

#### Example: `getAll` with Filters

**Configuration:**
```json
{
  "operation": "getAll",
  "statusId": "1"
}
```

#### Example: `delete`

**Configuration:**
```json
{
  "operation": "delete",
  "id": "VolejRejNm"
}
```

---

### HTTP Request Node Fallback Guidance

If your workflow requires specialized Invoice Ninja endpoints not directly exposed by this action node (such as raw PDF binary downloads, account backups, or custom bulk actions), you can use Fusion's core **HTTP Request** node as a fallback:

1. Add an **HTTP Request** node to your canvas.
2. Set **Method** to `GET`, `POST`, or `PUT`.
3. Set **URL** to: `https://<your-ninja-domain>/api/v1/<endpoint>`.
4. Include the required headers:
   - `X-Api-Token`: `{{secrets.INVOICE_NINJA_API_TOKEN}}`
   - `X-Requested-With`: `XMLHttpRequest`
   - `Content-Type`: `application/json`
<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Sample Workflow: Sync Completed GitHub Issues to Billable Tasks

When an issue closes in GitHub, log the spent time as a billable task in Invoice Ninja.

```json
{
  "nodes": [
    {
      "id": "github-webhook",
      "type": "webhook",
      "position": { "x": 80, "y": 100 }
    },
    {
      "id": "log-task",
      "type": "invoice-ninja-tasks",
      "position": { "x": 300, "y": 100 },
      "config": {
        "operation": "create",
        "clientId": "{{input.clientId}}",
        "description": "Fix: {{input.issueTitle}}",
        "duration": 3600,
        "rate": 100
      }
    },
    {
      "id": "log",
      "type": "log",
      "position": { "x": 520, "y": 100 }
    }
  ],
  "connections": [
    { "source": "github-webhook", "target": "log-task", "sourceHandle": "success", "targetHandle": "input" },
    { "source": "log-task", "target": "log", "sourceHandle": "success", "targetHandle": "input" }
  ]
}
```

**How it flows:**
1. GitHub webhook sends issue closed event.
2. **Invoice Ninja Tasks** logs the task duration and hourly rate.
3. Downstream node records the synchronized task ID.

### Common Patterns

- **Issue Tracker Sync:** GitHub / Linear Webhook ➔ Tasks (`create`)
- **Timer Control:** External Timer Start/Stop ➔ Tasks (`create` with `isRunning`)
- **Project Audit:** Cron ➔ Tasks (`getAll` by `projectId`) ➔ Slack Progress Digest
<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `401 Unauthorized`
- **Cause:** Missing, invalid, or expired `apiToken`.
- **Solution:** Generate or verify your API token in Invoice Ninja under **Settings &gt; Account Management &gt; API Tokens**. Ensure the token has adequate permissions.

#### `404 Not Found`
- **Cause:** The requested entity ID does not exist, was permanently deleted, or belongs to a different company account.
- **Solution:** Confirm the record ID in your Invoice Ninja dashboard. For self-hosted setups, check that the `baseUrl` does not contain extra subpath prefixes.

#### `422 Unprocessable Entity`
- **Cause:** Validation failed on the remote server (e.g., missing required fields, invalid date formats, or malformed line items).
- **Solution:** Inspect the error output for specific field validation messages. Ensure all dates use the `YYYY-MM-DD` standard format.

#### `Self-Hosted Connection Refused / Timeout`
- **Cause:** Firewall, reverse proxy (Nginx, Traefik), or SSL certificate issue preventing the request from reaching the Invoice Ninja server.
- **Solution:** Verify network connectivity and make sure the reverse proxy forwards the `X-Api-Token` and `X-Requested-With` headers without stripping them.

### Error Codes

| Code | Message | Solution |
|------|---------|----------|
| `401` | Unauthorized | Check that `apiToken` is valid and not expired |
| `404` | Not Found | Verify the target resource `id` |
| `422` | Unprocessable Entity | Ensure all required fields and valid formats are provided |
| `ECONNREFUSED` | Connection Refused | Check host availability and SSL certificate validity |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [invoice-ninja-invoices](../invoice-ninja-invoices/en.md)
- [invoice-ninja-quotes](../invoice-ninja-quotes/en.md)
- [invoice-ninja-clients](../invoice-ninja-clients/en.md)
- [http-request](../http-request/en.md)
<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-03 | Initial release of modular Invoice Ninja action node |
<!-- /SECTION: changelog -->
