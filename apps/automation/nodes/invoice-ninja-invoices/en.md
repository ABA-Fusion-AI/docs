---
node_id: "invoice-ninja-invoices"
title: "Invoice Ninja Invoices"
description: "Create, retrieve, query, delete, and email customer invoices in Invoice Ninja."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-03"
author: "Fusion Team"
tags:
  - invoice-ninja
  - invoices
  - billing
  - finance
  - accounting
  - peer-only
  - action
related_nodes:
  - invoice-ninja-clients
  - invoice-ninja-payments
  - invoice-ninja-quotes
  - http-request
---

<!-- SECTION: overview -->
# Invoice Ninja Invoices

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Create, retrieve, query, delete, and email customer invoices in Invoice Ninja.

The **Invoice Ninja Invoices** node connects Fusion workflows directly to Invoice Ninja (both hosted SaaS and self-hosted instances). It provides production-ready operations with automated token authentication, URL normalization, robust input validation, and structured error handling.

### Key Features

- **Flexible Itemization:** Pass line items as JSON strings or structured object arrays
- **Discount Flexibility:** Apply fixed monetary amounts or percentage-based discounts
- **Direct Email Dispatch:** Send generated invoices to clients with a single operation
- **Advanced Query Filters:** List invoices filtered by client ID, status, project, or category

### Use Cases

- Generate itemized invoices automatically upon order checkout or project milestone completion
- Trigger transactional email notifications delivering branded PDF invoices directly to clients
- Fetch invoice balances to verify pending payments before fulfilling digital service requests
- Query unpaid invoices for automated accounting reminders and overdue collections
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
| `create` | Generate an invoice with itemized line items, due date, discounts, terms, and footer |
| `get` | Retrieve complete invoice details including balance, line items, and payment history |
| `getAll` | List invoices with optional filters for client, status, vendor, category, or project |
| `delete` | Delete or cancel an invoice by its unique ID |
| `email` | Send the invoice directly to the client via Invoice Ninja email engine |

### Operation Parameters

### Operation: `create`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ✅ Yes | — | Unique ID of the client to invoice. |
| `lineItems` | `array` or `string` | ✅ Yes | — | Array of line items or JSON string. Each item supports `product_key`, `notes`, `cost`, and `quantity`. |
| `date` | `string` | ❌ No | — | Invoice issue date in `YYYY-MM-DD` format. |
| `dueDate` | `string` | ❌ No | — | Payment due date in `YYYY-MM-DD` format. |
| `discount` | `number` | ❌ No | — | Discount applied to the invoice. |
| `isAmountDiscount` | `boolean` | ❌ No | `false` | Set to `true` for fixed monetary discount, or `false` for percentage. |
| `terms` | `string` | ❌ No | — | Payment terms and conditions. |
| `footer` | `string` | ❌ No | — | Custom footer message printed on invoice. |
| `customValue1` | `string` | ❌ No | — | Custom field value 1. |
| `customValue2` | `string` | ❌ No | — | Custom field value 2. |

### Operation: `get`, `delete`, & `email`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | `string` | ✅ Yes | — | The unique Invoice Ninja invoice ID. |

### Operation: `getAll`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ❌ No | — | Filter invoices by recipient client ID. |
| `statusId` | `string` | ❌ No | — | Filter invoices by status ID. |
| `vendorId` | `string` | ❌ No | — | Filter by vendor ID. |
| `categoryId` | `string` | ❌ No | — | Filter by category ID. |
| `projectId` | `string` | ❌ No | — | Filter by project ID. |

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
  "lineItems": [
    { "notes": "Website Development", "cost": 1500, "quantity": 1 }
  ],
  "date": "2026-10-01",
  "dueDate": "2026-10-31"
}
```

**Output (`success`):**
```json
{
  "data": {
    "id": "OpkldkE48d",
    "client_id": "VolejRejNm",
    "amount": 1500,
    "balance": 1500,
    "status_id": "1",
    "line_items": [
      { "notes": "Website Development", "cost": 1500, "quantity": 1 }
    ]
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

### Sample Workflow: Create and Automatically Email Invoice on Order Complete

Generate a customer invoice when an e-commerce order completes, then dispatch it via email.

```json
{
  "nodes": [
    {
      "id": "order-paid",
      "type": "webhook",
      "position": { "x": 80, "y": 120 }
    },
    {
      "id": "create-invoice",
      "type": "invoice-ninja-invoices",
      "position": { "x": 300, "y": 120 },
      "config": {
        "operation": "create",
        "clientId": "{{input.clientId}}",
        "lineItems": "{{input.items}}"
      }
    },
    {
      "id": "send-email",
      "type": "invoice-ninja-invoices",
      "position": { "x": 520, "y": 120 },
      "config": {
        "operation": "email",
        "id": "{{input.data.id}}"
      }
    }
  ],
  "connections": [
    { "source": "order-paid", "target": "create-invoice", "sourceHandle": "success", "targetHandle": "input" },
    { "source": "create-invoice", "target": "send-email", "sourceHandle": "success", "targetHandle": "input" }
  ]
}
```

**How it flows:**
1. Webhook receives completed order payload with client ID and line items.
2. **Invoice Ninja Invoices** (`create`) generates the invoice in draft/sent state.
3. The subsequent **Invoice Ninja Invoices** (`email`) operation sends the PDF directly to the client.

### Common Patterns

- **Order Fulfillment:** Webhook ➔ Invoice Ninja Invoices (`create`) ➔ Invoice Ninja Invoices (`email`)
- **Payment Verification:** Cron ➔ Invoice Ninja Invoices (`get`) ➔ Branch on `balance === 0`
- **Overdue Chasing:** Cron ➔ Invoice Ninja Invoices (`getAll`) ➔ Filter past due ➔ Slack Alert
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

- [invoice-ninja-clients](../invoice-ninja-clients/en.md)
- [invoice-ninja-payments](../invoice-ninja-payments/en.md)
- [invoice-ninja-quotes](../invoice-ninja-quotes/en.md)
- [http-request](../http-request/en.md)
<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-03 | Initial release of modular Invoice Ninja action node |
<!-- /SECTION: changelog -->
