---
node_id: "invoice-ninja-payments"
title: "Invoice Ninja Payments"
description: "Record customer payments, query payment history, and reconcile balances in Invoice Ninja."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-03"
author: "Fusion Team"
tags:
  - invoice-ninja
  - payments
  - reconciliation
  - finance
  - accounting
  - peer-only
  - action
related_nodes:
  - invoice-ninja-invoices
  - invoice-ninja-clients
  - invoice-ninja-expenses
  - http-request
---

<!-- SECTION: overview -->
# Invoice Ninja Payments

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Record customer payments, query payment history, and reconcile balances in Invoice Ninja.

The **Invoice Ninja Payments** node connects Fusion workflows directly to Invoice Ninja (both hosted SaaS and self-hosted instances). It provides production-ready operations with automated token authentication, URL normalization, robust input validation, and structured error handling.

### Key Features

- **Automated Reconciliation:** Instantly credit payments against specific invoice IDs
- **Payment Method Mapping:** Support standard payment types (Bank Transfer, Credit Card, PayPal)
- **Audit Trail:** Store external transaction gateway references and private bookkeeping notes
- **Payment Queries:** Retrieve and filter transaction histories by client or status

### Use Cases

- Record incoming payments received from payment gateways (Stripe, PayPal, Wire Transfer)
- Reconcile outstanding invoice balances automatically in real-time
- Audit payment transaction history for accounting and tax reporting
- Void or delete erroneous payment entries securely
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
| `create` | Record a payment against an invoice or client account with transaction reference |
| `get` | Retrieve full details of an existing payment record |
| `getAll` | List payment transactions with filters by client, status, vendor, or project |
| `delete` | Delete or void a recorded payment by ID |

### Operation Parameters

### Operation: `create`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `amount` | `number` | ✅ Yes | — | Monetary payment amount. |
| `clientId` | `string` | ❌ No | — | Associated client ID. |
| `invoiceId` | `string` | ❌ No | — | Target invoice ID to apply payment towards. |
| `paymentDate` | `string` | ❌ No | — | Date of payment in `YYYY-MM-DD` format. |
| `transactionReference` | `string` | ❌ No | — | Bank or gateway transaction reference ID. |
| `typeId` | `string` | ❌ No | — | Payment method type ID (1=Bank, 4=Credit Card, 6=PayPal, etc.). |
| `privateNotes` | `string` | ❌ No | — | Internal bookkeeping notes. |
| `customValue1` | `string` | ❌ No | — | Custom field value 1. |
| `customValue2` | `string` | ❌ No | — | Custom field value 2. |

### Operation: `get` & `delete`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | `string` | ✅ Yes | — | The unique Invoice Ninja payment ID. |

### Operation: `getAll`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ❌ No | — | Filter payments by client ID. |
| `statusId` | `string` | ❌ No | — | Filter payments by status ID. |
| `vendorId` | `string` | ❌ No | — | Filter payments by vendor ID. |
| `categoryId` | `string` | ❌ No | — | Filter payments by category ID. |
| `projectId` | `string` | ❌ No | — | Filter payments by project ID. |

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
  "invoiceId": "OpkldkE48d",
  "amount": 1500,
  "transactionReference": "ch_3N82jx...",
  "typeId": "4"
}
```

**Output (`success`):**
```json
{
  "data": {
    "id": "Pay77a98v",
    "amount": 1500,
    "invoice_id": "OpkldkE48d",
    "transaction_reference": "ch_3N82jx...",
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

### Sample Workflow: Reconcile Stripe Webhook Payments in Invoice Ninja

Catch a charge.succeeded webhook and record the payment against the matching invoice.

```json
{
  "nodes": [
    {
      "id": "stripe-webhook",
      "type": "webhook",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "record-payment",
      "type": "invoice-ninja-payments",
      "position": { "x": 320, "y": 100 },
      "config": {
        "operation": "create",
        "invoiceId": "{{input.invoiceId}}",
        "amount": "{{input.amount}}",
        "transactionReference": "{{input.chargeId}}",
        "typeId": "4"
      }
    },
    {
      "id": "notify",
      "type": "log",
      "position": { "x": 540, "y": 100 }
    }
  ],
  "connections": [
    { "source": "stripe-webhook", "target": "record-payment", "sourceHandle": "success", "targetHandle": "input" },
    { "source": "record-payment", "target": "notify", "sourceHandle": "success", "targetHandle": "input" }
  ]
}
```

**How it flows:**
1. Stripe triggers webhook when payment clears.
2. **Invoice Ninja Payments** (`create`) logs the payment and updates the invoice balance to 0.
3. Notification node alerts the finance team.

### Common Patterns

- **Gateway Reconcile:** Stripe / PayPal Webhook ➔ Invoice Ninja Payments (`create`)
- **Audit Ledger:** Cron ➔ Invoice Ninja Payments (`getAll`) ➔ ERP Export
- **Payment Lookup:** API Trigger ➔ Invoice Ninja Payments (`get`)
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
- [invoice-ninja-expenses](../invoice-ninja-expenses/en.md)
- [http-request](../http-request/en.md)
<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-03 | Initial release of modular Invoice Ninja action node |
<!-- /SECTION: changelog -->
