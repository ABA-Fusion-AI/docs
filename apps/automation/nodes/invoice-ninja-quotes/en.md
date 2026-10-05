---
node_id: "invoice-ninja-quotes"
title: "Invoice Ninja Quotes"
description: "Create, retrieve, query, delete, and email price estimates and quotes in Invoice Ninja."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-03"
author: "Fusion Team"
tags:
  - invoice-ninja
  - quotes
  - estimates
  - proposals
  - finance
  - accounting
  - peer-only
  - action
related_nodes:
  - invoice-ninja-invoices
  - invoice-ninja-clients
  - invoice-ninja-tasks
  - http-request
---

<!-- SECTION: overview -->
# Invoice Ninja Quotes

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Create, retrieve, query, delete, and email price estimates and quotes in Invoice Ninja.

The **Invoice Ninja Quotes** node connects Fusion workflows directly to Invoice Ninja (both hosted SaaS and self-hosted instances). It provides production-ready operations with automated token authentication, URL normalization, robust input validation, and structured error handling.

### Key Features

- **Detailed Estimates:** Build multi-item quotations with custom descriptions and pricing
- **Direct Client Delivery:** Email proposals directly from workflows using Invoice Ninja templates
- **Lifecycle Tracking:** Query quotes by status to track accepted, rejected, or pending proposals
- **Seamless Conversion:** Integrate quotes that can easily transition into full invoices

### Use Cases

- Generate customized price quotes automatically from web form inquiry submissions
- Email formal PDF proposals and quotes to prospective clients directly from workflows
- Track quote status (sent, approved, expired) to automate sales follow-up cadences
- Remove outdated or rejected quotations cleanly
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
| `create` | Create a price estimate with line items, valid-until date, terms, and discounts |
| `get` | Retrieve full details of an existing quote |
| `getAll` | List quotes filtered by client, status, vendor, category, or project |
| `delete` | Delete a quotation by its ID |
| `email` | Send the quote directly to the client via email |

### Operation Parameters

### Operation: `create`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ✅ Yes | — | Unique ID of the client receiving the quote. |
| `lineItems` | `array` or `string` | ✅ Yes | — | Array of line items or JSON string. Each item supports `product_key`, `notes`, `cost`, and `quantity`. |
| `date` | `string` | ❌ No | — | Quote issue date in `YYYY-MM-DD` format. |
| `dueDate` | `string` | ❌ No | — | Quote expiration / valid-until date in `YYYY-MM-DD` format. |
| `discount` | `number` | ❌ No | — | Discount applied to the quote. |
| `isAmountDiscount` | `boolean` | ❌ No | `false` | Set to `true` for fixed monetary discount, or `false` for percentage. |
| `terms` | `string` | ❌ No | — | Terms, conditions, and scope of work. |
| `footer` | `string` | ❌ No | — | Custom footer message. |
| `customValue1` | `string` | ❌ No | — | Custom field value 1. |
| `customValue2` | `string` | ❌ No | — | Custom field value 2. |

### Operation: `get`, `delete`, & `email`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | `string` | ✅ Yes | — | The unique Invoice Ninja quote ID. |

### Operation: `getAll`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ❌ No | — | Filter quotes by client ID. |
| `statusId` | `string` | ❌ No | — | Filter quotes by status ID. |
| `vendorId` | `string` | ❌ No | — | Filter quotes by vendor ID. |
| `categoryId` | `string` | ❌ No | — | Filter quotes by category ID. |
| `projectId` | `string` | ❌ No | — | Filter quotes by project ID. |

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
    { "notes": "Consulting Services", "cost": 120, "quantity": 10 }
  ],
  "dueDate": "2026-11-15"
}
```

**Output (`success`):**
```json
{
  "data": {
    "id": "Qte1088dF",
    "client_id": "VolejRejNm",
    "amount": 1200,
    "status_id": "1",
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

### Sample Workflow: Generate and Email Estimate from Web Inquiry

Receive a website proposal request form and immediately dispatch a quote to the client.

```json
{
  "nodes": [
    {
      "id": "form-submit",
      "type": "webhook",
      "position": { "x": 80, "y": 120 }
    },
    {
      "id": "create-quote",
      "type": "invoice-ninja-quotes",
      "position": { "x": 300, "y": 120 },
      "config": {
        "operation": "create",
        "clientId": "{{input.clientId}}",
        "lineItems": "{{input.lineItems}}"
      }
    },
    {
      "id": "email-quote",
      "type": "invoice-ninja-quotes",
      "position": { "x": 520, "y": 120 },
      "config": {
        "operation": "email",
        "id": "{{input.data.id}}"
      }
    }
  ],
  "connections": [
    { "source": "form-submit", "target": "create-quote", "sourceHandle": "success", "targetHandle": "input" },
    { "source": "create-quote", "target": "email-quote", "sourceHandle": "success", "targetHandle": "input" }
  ]
}
```

**How it flows:**
1. Website inquiry triggers workflow.
2. **Invoice Ninja Quotes** (`create`) generates the proposal.
3. The quote is emailed directly to the prospect via `email` operation.

### Common Patterns

- **Form ➔ Quote ➔ Email:** Webhook ➔ Quotes (`create`) ➔ Quotes (`email`)
- **Quote Status Audit:** Cron ➔ Quotes (`getAll`) ➔ Slack follow-up reminder
- **Quote Retrieval:** API Trigger ➔ Quotes (`get`)
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
- [invoice-ninja-tasks](../invoice-ninja-tasks/en.md)
- [http-request](../http-request/en.md)
<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-03 | Initial release of modular Invoice Ninja action node |
<!-- /SECTION: changelog -->
