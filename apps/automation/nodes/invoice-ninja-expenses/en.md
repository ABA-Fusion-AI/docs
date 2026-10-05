---
node_id: "invoice-ninja-expenses"
title: "Invoice Ninja Expenses"
description: "Track operational expenses, billable costs, vendor charges, and categories in Invoice Ninja."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-03"
author: "Fusion Team"
tags:
  - invoice-ninja
  - expenses
  - bookkeeping
  - finance
  - accounting
  - peer-only
  - action
related_nodes:
  - invoice-ninja-invoices
  - invoice-ninja-clients
  - invoice-ninja-payments
  - http-request
---

<!-- SECTION: overview -->
# Invoice Ninja Expenses

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Track operational expenses, billable costs, vendor charges, and categories in Invoice Ninja.

The **Invoice Ninja Expenses** node connects Fusion workflows directly to Invoice Ninja (both hosted SaaS and self-hosted instances). It provides production-ready operations with automated token authentication, URL normalization, robust input validation, and structured error handling.

### Key Features

- **Vendor & Category Attribution:** Categorize costs by vendor ID and accounting category ID
- **Client Billability:** Associate expenses with clients for later re-invoicing
- **Multi-Currency Recording:** Track global expenses in native currencies
- **Flexible Filtering:** Query expenses across vendors, clients, dates, and categories

### Use Cases

- Log supplier bills, software subscriptions, and operational expenses automatically
- Track billable expenses incurred on behalf of a specific client for later invoicing
- Filter and aggregate quarterly expenses by category or vendor for tax preparation
- Delete or adjust obsolete expense records via automated maintenance jobs
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
| `create` | Record a new expense with amount, vendor, category, and billable client assignment |
| `get` | Retrieve full details of an expense record |
| `getAll` | List expenses filtered by vendor, client, status, category, or project |
| `delete` | Delete an expense record by its ID |

### Operation Parameters

### Operation: `create`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `amount` | `number` | ✅ Yes | — | Total monetary amount of the expense. |
| `date` | `string` | ❌ No | — | Expense date in `YYYY-MM-DD` format. |
| `vendorId` | `string` | ❌ No | — | Unique ID of the vendor or supplier. |
| `categoryId` | `string` | ❌ No | — | Expense category ID for classification. |
| `clientId` | `string` | ❌ No | — | Client ID (if expense is re-billable to a client). |
| `currencyId` | `string` | ❌ No | — | Currency ID of the expense. |
| `customValue1` | `string` | ❌ No | — | Custom field value 1. |
| `customValue2` | `string` | ❌ No | — | Custom field value 2. |

### Operation: `get` & `delete`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | `string` | ✅ Yes | — | The unique Invoice Ninja expense ID. |

### Operation: `getAll`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ❌ No | — | Filter expenses by assigned client ID. |
| `statusId` | `string` | ❌ No | — | Filter expenses by status ID. |
| `vendorId` | `string` | ❌ No | — | Filter expenses by vendor ID. |
| `categoryId` | `string` | ❌ No | — | Filter expenses by category ID. |
| `projectId` | `string` | ❌ No | — | Filter expenses by project ID. |

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
  "amount": 250,
  "vendorId": "Ven98Kll3",
  "categoryId": "Cat112",
  "date": "2026-10-02"
}
```

**Output (`success`):**
```json
{
  "data": {
    "id": "Exp4429vL",
    "amount": 250,
    "vendor_id": "Ven98Kll3",
    "category_id": "Cat112",
    "date": "2026-10-02",
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

### Sample Workflow: Log Monthly Cloud Infrastructure Expenses

Scheduled automation that captures AWS invoice data and logs it as a categorized expense in Invoice Ninja.

```json
{
  "nodes": [
    {
      "id": "monthly-trigger",
      "type": "cron",
      "position": { "x": 80, "y": 100 }
    },
    {
      "id": "create-expense",
      "type": "invoice-ninja-expenses",
      "position": { "x": 300, "y": 100 },
      "config": {
        "operation": "create",
        "amount": 340.50,
        "vendorId": "AWS-VENDOR-ID",
        "categoryId": "INFRA-CAT-ID"
      }
    },
    {
      "id": "log",
      "type": "log",
      "position": { "x": 520, "y": 100 }
    }
  ],
  "connections": [
    { "source": "monthly-trigger", "target": "create-expense", "sourceHandle": "success", "targetHandle": "input" },
    { "source": "create-expense", "target": "log", "sourceHandle": "success", "targetHandle": "input" }
  ]
}
```

**How it flows:**
1. Cron trigger fires on the 1st of every month.
2. **Invoice Ninja Expenses** creates the operational expenditure record.
3. Downstream node records execution metrics.

### Common Patterns

- **Subscription Logging:** Cron ➔ Invoice Ninja Expenses (`create`)
- **Billable Expenses:** Incident resolution ➔ Invoice Ninja Expenses (`create` with `clientId`)
- **Financial Reporting:** Cron ➔ Invoice Ninja Expenses (`getAll`) ➔ Spreadsheet
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
- [invoice-ninja-clients](../invoice-ninja-clients/en.md)
- [invoice-ninja-payments](../invoice-ninja-payments/en.md)
- [http-request](../http-request/en.md)
<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-03 | Initial release of modular Invoice Ninja action node |
<!-- /SECTION: changelog -->
