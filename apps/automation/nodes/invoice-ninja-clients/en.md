---
node_id: "invoice-ninja-clients"
title: "Invoice Ninja Clients"
description: "Manage client lifecycles, contact details, and account configurations in Invoice Ninja."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-03"
author: "Fusion Team"
tags:
  - invoice-ninja
  - clients
  - crm
  - finance
  - accounting
  - peer-only
  - action
related_nodes:
  - invoice-ninja-invoices
  - invoice-ninja-payments
  - invoice-ninja-quotes
  - http-request
---

<!-- SECTION: overview -->
# Invoice Ninja Clients

> **Category:** Peer-only Integrations&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage client lifecycles, contact details, and account configurations in Invoice Ninja.

The **Invoice Ninja Clients** node connects Fusion workflows directly to Invoice Ninja (both hosted SaaS and self-hosted instances). It provides production-ready operations with automated token authentication, URL normalization, robust input validation, and structured error handling.

### Key Features

- **Full Client Lifecycle:** Create, inspect, filter, and archive customer accounts
- **Tax & Currency Controls:** Assign default currencies and tax/VAT numbers per client
- **Custom Field Support:** Extend client schemas with custom attributes (`customValue1`, `customValue2`)
- **Multi-Criteria Querying:** Filter customer directories by status, vendor, category, or project

### Use Cases

- Automatically provision new clients when a lead converts in your CRM
- Look up client account records and tax identifiers before generating invoices
- Filter and export active customer lists for synchronization with external data warehouses
- Safely archive or delete deprecated client profiles from automated workflows
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
| `create` | Create a new client with business details, VAT number, and currency settings |
| `get` | Fetch a single client record by its unique identifier |
| `getAll` | Retrieve a list of clients with optional filtering by status, project, vendor, or category |
| `delete` | Permanently remove or archive a client by ID |

### Operation Parameters

### Operation: `create`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `name` | `string` | ✅ Yes | — | Name of the client or company. |
| `idNumber` | `string` | ❌ No | — | Customer ID or internal accounting identifier. |
| `vatNumber` | `string` | ❌ No | — | Tax or VAT registration number. |
| `currencyId` | `string` | ❌ No | — | Currency ID (e.g. `'1'` for USD, `'3'` for EUR). |
| `customValue1` | `string` | ❌ No | — | Custom field value 1. |
| `customValue2` | `string` | ❌ No | — | Custom field value 2. |

### Operation: `get` & `delete`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `id` | `string` | ✅ Yes | — | The unique Invoice Ninja client ID. |

### Operation: `getAll`

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `clientId` | `string` | ❌ No | — | Filter by specific client ID. |
| `statusId` | `string` | ❌ No | — | Filter by client status ID. |
| `vendorId` | `string` | ❌ No | — | Filter by associated vendor ID. |
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
  "name": "Acme Corporation",
  "vatNumber": "FR12345678901",
  "currencyId": "1"
}
```

**Output (`success`):**
```json
{
  "data": {
    "id": "VolejRejNm",
    "name": "Acme Corporation",
    "vat_number": "FR12345678901",
    "currency_id": "1",
    "created_at": 1727974800,
    "updated_at": 1727974800
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

### Sample Workflow: Sync CRM Contacts to Invoice Ninja Clients

Listen for new contacts via webhook and automatically create them as clients in Invoice Ninja.

```json
{
  "nodes": [
    {
      "id": "crm-webhook",
      "type": "webhook",
      "position": { "x": 100, "y": 100 }
    },
    {
      "id": "create-client",
      "type": "invoice-ninja-clients",
      "position": { "x": 320, "y": 100 },
      "config": {
        "operation": "create",
        "name": "{{input.companyName}}",
        "vatNumber": "{{input.vat}}",
        "currencyId": "1"
      }
    },
    {
      "id": "log-result",
      "type": "log",
      "position": { "x": 540, "y": 100 }
    }
  ],
  "connections": [
    { "source": "crm-webhook", "target": "create-client", "sourceHandle": "success", "targetHandle": "input" },
    { "source": "create-client", "target": "log-result", "sourceHandle": "success", "targetHandle": "input" }
  ]
}
```

**How it flows:**
1. Webhook receives a converted CRM lead with `{ "companyName": "Acme Corp", "vat": "FR998877" }`.
2. The **Invoice Ninja Clients** node creates the client and assigns a unique ID.
3. Downstream nodes log the newly created client record or trigger onboarding sequences.

### Common Patterns

- **CRM Sync:** Hubspot / Pipedrive Webhook ➔ Invoice Ninja Clients (`create`)
- **Client Validation:** Pre-invoice check ➔ Invoice Ninja Clients (`get`)
- **Batch Reconcile:** Scheduled Cron ➔ Invoice Ninja Clients (`getAll`) ➔ Database
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
