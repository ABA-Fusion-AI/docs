---
node_id: "erpnext"
title: "ERPNext"
description: "Manage documents, run reports, and call methods in ERPNext ERP system."
category: "peer-only"
subcategory: "Integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-08"
author: "Fusion Team"
tags:
  - erpnext
  - frappe
  - erp
  - accounting
  - integration
  - peer-only
related_nodes:
  - http-request
  - function
  - cron
---

<!-- SECTION: overview -->
# ERPNext

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Read and write data in [ERPNext](https://erpnext.com), the open-source ERP built on the Frappe framework: list, read, create, update, and delete any document type (Customer, Item, Sales Order, …), run reports, and call server methods. The node uses the Frappe REST API with an API key and secret, and works with Frappe Cloud sites and self-hosted ERPNext instances.

### Use Cases

- **Sync customers and items:** Create or update ERPNext customers, suppliers, or items from a form, a CRM, or another system.
- **Read business documents:** Fetch sales orders, invoices, or stock entries to send them to a report or a notification.
- **Run reports:** Get the result of an ERPNext report (e.g. sales, stock, or customer reports) in a workflow.
- **Call custom logic:** Trigger a whitelisted Frappe method from your own app.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | No | `listDocuments` | `listDocuments`, `getDocument`, `createDocument`, `updateDocument`, `deleteDocument`, `runReport`, or `callMethod`. |
| `host` | `string` | Yes | — | URL of the ERPNext site, without a trailing path, e.g. `https://mycompany.erpnext.com`. Supports expressions. |
| `apiKey` | `string` | Yes | — | API key of the ERPNext user. Supports expressions. |
| `apiSecret` | `string` | Yes | — | API secret of the ERPNext user. Supports expressions. |
| `doctype` | `string` | For document operations | — | DocType name, exactly as in ERPNext, e.g. `Customer`, `Item`, `Sales Order`. Supports expressions. |
| `name` | `string` | For `getDocument`, `updateDocument`, `deleteDocument` | — | Document name (ID), e.g. `CUST-2026-00001` or `Fusion Test Customer`. Supports expressions. |
| `data` | `string` (JSON) | For `createDocument`, `updateDocument` | — | JSON object of document fields, sent as-is. Supports expressions. |
| `reportName` | `string` | For `runReport` | — | Name of the report, exactly as in ERPNext. Supports expressions. |
| `filtersJson` | `string` (JSON) | No | `{}` | JSON object of report filters, for `runReport`. Supports expressions. |
| `method` | `string` | For `callMethod` | — | Dotted path of a whitelisted Frappe method, e.g. `frappe.auth.get_logged_user`. Supports expressions. |
| `args` | `string` (JSON) | No | `{}` | JSON object of method arguments, for `callMethod`. Supports expressions. |
| `limit` | `string` | No | `20` | Maximum number of documents returned by `listDocuments`. Supports expressions. |

### Available Operations

| Operation | Description | ERPNext request | Required |
|-----------|-------------|-----------------|----------|
| `listDocuments` | List documents of a DocType (fields `name` and `modified`). | `GET /api/resource/<doctype>?limit_page_length=<limit>&fields=["name","modified"]` | `doctype` |
| `getDocument` | Read one document with all its fields. | `GET /api/resource/<doctype>/<name>` | `doctype`, `name` |
| `createDocument` | Create a document from `data`. | `POST /api/resource/<doctype>` | `doctype`, `data` |
| `updateDocument` | Update the fields given in `data`. | `PUT /api/resource/<doctype>/<name>` | `doctype`, `name`, `data` |
| `deleteDocument` | Delete a document. | `DELETE /api/resource/<doctype>/<name>` | `doctype`, `name` |
| `runReport` | Run a report and return its rows and columns. | `GET /api/method/frappe.desk.query_report.run?report_name=<reportName>&filters=<filtersJson>` | `reportName` |
| `callMethod` | Call a whitelisted server method. | `POST /api/method/<method>` | `method` |

All parameters are shown for every operation. `listDocuments` always returns only the `name` and `modified` fields; use `getDocument` for the full document.

### Authentication

Every request sends `Authorization: token <apiKey>:<apiSecret>`. The API key and secret belong to an ERPNext user, and the node has the same permissions as that user.

### Creating an API Key

1. Sign in to ERPNext as the user the workflow should act as (or as an administrator).
2. Open the user record: search for **User List** and open the user, or go to `<host>/app/user/<user-email>`.
3. On the **Settings** tab, in **API Access**, click **Generate Keys**.
4. Copy the **API Secret** from the popup (it is shown only once). The **API Key** stays visible in the API Access section.
5. Paste both into `apiKey` and `apiSecret`.

### Writing `data`

`data` is a JSON object of field names and values, as in the DocType. Example for a new customer:

```json
{
  "customer_name": "Acme Corp",
  "customer_type": "Company"
}
```

For `updateDocument`, send only the fields to change, e.g. `{ "customer_details": "Key account" }`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. The incoming data is not used directly; use expressions in the parameters to pass upstream values. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The ERPNext response: `{ "data": [ ... ] }` for `listDocuments`, `{ "data": { ... } }` for `getDocument`, `createDocument` and `updateDocument`, `{}` for `deleteDocument`, and `{ "message": ... }` for `runReport` and `callMethod`. |
| `error` | `Error` | `ERPNext Error <status>: <response>` for API errors, a JSON parse error for invalid `args`, or a validation message (`host, apiKey, and apiSecret are required`, `doctype is required for listDocuments`, …). |

### Output Examples

#### `listDocuments`

```json
{
  "data": [
    { "name": "Fusion Test Customer", "modified": "2026-10-08 15:34:39.393638" }
  ]
}
```

#### `createDocument` / `getDocument` / `updateDocument`

```json
{
  "data": {
    "name": "Fusion Test Customer",
    "owner": "admin@example.com",
    "creation": "2026-10-08 15:34:20.600332",
    "modified": "2026-10-08 15:34:39.393638",
    "docstatus": 0,
    "naming_series": "CUST-.YYYY.-",
    "customer_name": "Fusion Test Customer",
    "customer_type": "Company",
    "customer_details": "Updated by Fusion node test"
  }
}
```

#### `runReport`

```json
{
  "message": {
    "result": [
      {
        "customer": "Fusion Test Customer",
        "customer_name": "Fusion Test Customer",
        "territory": null,
        "customer_group": null
      }
    ],
    "columns": [
      { "fieldname": "customer", "label": "Customer", "fieldtype": "Link", "options": "Customer", "width": "120" },
      { "fieldname": "customer_name", "label": "Customer Name", "fieldtype": "", "width": "120" }
    ]
  }
}
```

#### `callMethod` (`frappe.auth.get_logged_user`)

```json
{
  "message": "admin@example.com"
}
```

#### `deleteDocument`

```json
{}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use ERPNext in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow on demand.
2. **ERPNext Node:** Runs the configured operation (e.g. `listDocuments` on `Customer`).
3. **Log Node:** Displays the ERPNext response.

### Example Configurations

#### Create a customer from upstream data

```json
{
  "operation": "createDocument",
  "host": "https://mycompany.erpnext.com",
  "apiKey": "{{ secrets.ERPNEXT_API_KEY }}",
  "apiSecret": "{{ secrets.ERPNEXT_API_SECRET }}",
  "doctype": "Customer",
  "data": "{\"customer_name\":\"{{ input.company }}\",\"customer_type\":\"Company\"}"
}
```

#### Run a report with filters

```json
{
  "operation": "runReport",
  "host": "https://mycompany.erpnext.com",
  "apiKey": "{{ secrets.ERPNEXT_API_KEY }}",
  "apiSecret": "{{ secrets.ERPNEXT_API_SECRET }}",
  "reportName": "Customers Without Any Sales Transactions",
  "filtersJson": "{}"
}
```

#### Call a server method

```json
{
  "operation": "callMethod",
  "host": "https://mycompany.erpnext.com",
  "apiKey": "{{ secrets.ERPNEXT_API_KEY }}",
  "apiSecret": "{{ secrets.ERPNEXT_API_SECRET }}",
  "method": "frappe.auth.get_logged_user"
}
```

### Common Patterns

- **Webhook → ERPNext (`createDocument`):** Create a customer or lead when a form is submitted.
- **Cron → ERPNext (`runReport`) → Notification:** Send a daily report result by e-mail or chat.
- **ERPNext (`listDocuments`) → Function → ERPNext (`getDocument`):** Loop over documents and read each one in full.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store `apiKey` and `apiSecret` in Fusion's **Secrets** system. Never paste them directly into workflow parameters or commit them to version control.

- The node acts with the permissions of the user who owns the API key; use a dedicated user with only the roles the workflow needs.
- Regenerate the keys in ERPNext (User → Settings → API Access) if they leak; this invalidates the old secret.
- Always use an HTTPS `host`.
- `callMethod` can run any method the user is allowed to call; only call methods you trust.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `ERPNext Error 401` / `403`
- **Cause:** Wrong API key or secret, keys regenerated, or the user lacks permission on the DocType.
- **Solution:** Generate new keys on the user record and check the user's roles.

#### `ERPNext Error 404: … DoesNotExistError`
- **Cause:** The document `name` or the `doctype` does not exist (DocType names are case-sensitive, e.g. `Sales Order`).
- **Solution:** Get names from `listDocuments` and copy the DocType name exactly as shown in ERPNext.

#### `ERPNext Error 417` / `ValidationError` on create or update
- **Cause:** `data` misses a mandatory field or contains an invalid value.
- **Solution:** Check the mandatory fields of the DocType in ERPNext and send them in `data`.

#### `Unexpected token` / JSON error
- **Cause:** `args` is not valid JSON (parsed by the node), or `data` / `filtersJson` is not valid JSON (rejected by ERPNext with an error response).
- **Solution:** Validate the JSON; inside expressions, escape quotes correctly.

#### `fetch failed` or HTML error
- **Cause:** `host` is wrong or not reachable from Fusion (e.g. `localhost`).
- **Solution:** Use the public URL of the site. For a local test instance, expose it with a tunnel.

#### Only `name` and `modified` are returned
- **Cause:** `listDocuments` requests only these two fields.
- **Solution:** Use `getDocument` for each document, or the HTTP Request node with a custom `fields` parameter.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-10-08 | Full documentation: parameters, operations, authentication, API key setup, output examples, troubleshooting |
| 1.0.0 | 2026-08-04 | Initial documentation |

<!-- /SECTION: changelog -->