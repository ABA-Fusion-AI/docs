---
node_id: "magento"
title: "Magento"
description: "List, read, create and update Magento 2 products, list and read orders, and create customers through the Magento REST API."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-11-24"
author: "Fusion Team"
tags:
  - magento
  - adobe-commerce
  - e-commerce
  - products
  - orders
  - customers
related_nodes:
  - shopify
  - woocommerce
---

<!-- SECTION: overview -->
# Magento

> **Category:** Peer-Only Integrations &nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

Manage a **Magento 2** (Magento Open Source, Adobe Commerce or Mage-OS) store from a workflow through its **REST API** (`<baseUrl>/rest/default/V1`). The node offers **7 operations**:

| Operation | Description |
|-----------|-------------|
| `listProducts` | List the products of the catalog (paged, with optional filters). |
| `getProduct` | Get one product by its SKU. |
| `createProduct` | Create a product. |
| `updateProduct` | Update a product by its SKU. |
| `listOrders` | List the orders (paged, with optional filters). |
| `getOrder` | Get one order by its ID. |
| `createCustomer` | Create a customer account. |

### Use Cases

- **Catalog sync:** Create or update products from a spreadsheet, an ERP or a PIM.
- **Price updates:** Change prices or stock data on a schedule.
- **Order reporting:** Read new orders and send them to a CRM, a database or a chat channel.
- **Customer onboarding:** Create customer accounts from a form or another platform.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Authentication

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `baseUrl` | `string` | Yes | Store URL, e.g. `https://shop.example.com`. `https://` is added if missing; a trailing `/rest/...` or `/` is removed. |
| `accessToken` | `string` | Yes | Bearer token, sent as `Authorization: Bearer <token>`. |

Two ways to get a token:

- **Integration token (recommended):** In the Magento admin, go to **System → Extensions → Integrations → Add New Integration**, give it access to the needed resources (Catalog, Sales, Customers), save, then **Activate** it. Use the **Access Token** it shows. On Magento 2.4.4+, enable **Stores → Configuration → Services → OAuth → Consumer Settings → Allow OAuth Access Tokens to be used as standalone Bearer tokens**.
- **Admin token:** `POST <baseUrl>/rest/V1/integration/admin/token` with `{"username": "...", "password": "..."}`. The token expires after 4 hours by default, and the request is refused while admin two-factor authentication is enabled.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `operation` | `enum` | Yes | One of the 7 operations. Default: `listProducts`. |
| `sku` | `string` | For `getProduct`, `updateProduct` | Product SKU. |
| `productData` | `object` | For `createProduct`, `updateProduct` | Product fields (see below). |
| `orderId` | `number` | For `getOrder` | Order `entity_id` (from `listOrders`), not the increment ID shown in the admin. |
| `customerData` | `object` | For `createCustomer` | Customer fields (see below). |
| `searchCriteria` | `object` | No (`listProducts`, `listOrders`) | Magento `searchCriteria`. Empty → `{ "currentPage": 1, "pageSize": 20 }`. |

> Fields are shown only for the operations that use them.

### Filling the object fields

`productData`, `customerData` and `searchCriteria` are **key/value editors**:

1. Type the key (e.g. `sku`) in the field and click **Add entry**.
2. Type the value in the new entry **as JSON**: text between quotes (`"simple"`), numbers and booleans without quotes (`19.99`, `4`, `true`), objects and arrays as JSON (`[1]`).

The node wraps the data for you: `productData` is sent as `{ "product": { ... } }` and `customerData` as `{ "customer": { ... } }`.

#### `productData` keys

| Key | Example value | Note |
|-----|---------------|------|
| `sku` | `"FUSION-TEST-001"` | Required to create. |
| `name` | `"Fusion Test Product"` | Required to create. |
| `price` | `19.99` | |
| `attribute_set_id` | `4` | `4` is the default attribute set. Required to create. |
| `type_id` | `"simple"` | `simple`, `virtual`, `configurable`, ... |
| `status` | `1` | `1` enabled, `2` disabled. |
| `visibility` | `4` | `1` not visible, `2` catalog, `3` search, `4` catalog and search. |
| `weight` | `1` | |

For `updateProduct`, send only the keys to change.

#### `customerData` keys

| Key | Example value | Note |
|-----|---------------|------|
| `email` | `"jane@example.com"` | Required. Must be unique on the website. |
| `firstname` | `"Jane"` | Required. |
| `lastname` | `"Doe"` | Required. |
| `website_id` | `1` | Optional. |
| `group_id` | `1` | Optional (`1` = General). |

#### `searchCriteria` keys

| Key | Example value |
|-----|---------------|
| `currentPage` | `1` |
| `pageSize` | `50` |
| `filterGroups` | `[{"filters":[{"field":"price","value":"20","conditionType":"gt"}]}]` |
| `sortOrders` | `[{"field":"created_at","direction":"DESC"}]` |

### Operations

| Operation | Request | Required | Returns |
|-----------|---------|----------|---------|
| `listProducts` | `GET /rest/default/V1/products?searchCriteria...` | — | `{ items, search_criteria, total_count }` |
| `getProduct` | `GET /rest/default/V1/products/{sku}` | `sku` | Product |
| `createProduct` | `POST /rest/default/V1/products` | `productData` | Created product |
| `updateProduct` | `PUT /rest/default/V1/products/{sku}` | `sku`, `productData` | Updated product |
| `listOrders` | `GET /rest/default/V1/orders?searchCriteria...` | — | `{ items, search_criteria, total_count }` |
| `getOrder` | `GET /rest/default/V1/orders/{orderId}` | `orderId` | Order |
| `createCustomer` | `POST /rest/default/V1/customers` | `customerData` | Created customer |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Triggers the node. Use upstream values in the parameters through expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | The Magento API response. |
| `error` | `Error` | Missing fields (`Product data is required`, `SKU is required`, ...) or API errors `Magento API error (<status>): <message>`. |

### Output Examples

#### `listProducts` / `listOrders` (empty store)

```json
{
  "items": [],
  "search_criteria": {
    "filter_groups": [],
    "page_size": 20,
    "current_page": 1
  },
  "total_count": 0
}
```

#### `createProduct` / `getProduct` / `updateProduct`

```json
{
  "id": 1,
  "sku": "FUSION-TEST-001",
  "name": "Fusion Test Product Updated",
  "attribute_set_id": 4,
  "price": 24.99,
  "status": 1,
  "visibility": 4,
  "type_id": "simple",
  "created_at": "2026-11-24 14:53:30",
  "updated_at": "2026-11-24 14:54:54",
  "extension_attributes": {
    "website_ids": [1],
    "stock_item": {
      "item_id": 1,
      "product_id": 1,
      "stock_id": 1,
      "qty": null,
      "is_in_stock": false
    }
  }
}
```

#### `createCustomer`

```json
{
  "id": 1,
  "group_id": 1,
  "created_at": "2026-11-24 14:55:41",
  "updated_at": "2026-11-24 14:55:41",
  "created_in": "Default Store View",
  "email": "fusion.test@example.com",
  "firstname": "Fusion",
  "lastname": "Tester",
  "store_id": 1,
  "website_id": 1,
  "addresses": [],
  "disable_auto_group_change": 0,
  "extension_attributes": {
    "is_subscribed": false
  }
}
```

#### `getOrder` (unknown ID)

```json
{
  "name": "Error",
  "message": "Magento API error (404): The entity that was requested doesn't exist. Verify the entity and try again."
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Use Magento in a workflow
```

### How it flows

1. **Manual Trigger:** Starts the workflow.
2. **Magento Node:** Runs the selected operation.
3. **Log Node:** Displays the response.

### Example Configurations

#### List the last 50 orders

```json
{
  "baseUrl": "https://shop.example.com",
  "accessToken": "{{ secrets.MAGENTO_TOKEN }}",
  "operation": "listOrders",
  "searchCriteria": {
    "currentPage": 1,
    "pageSize": 50,
    "sortOrders": [{ "field": "created_at", "direction": "DESC" }]
  }
}
```

#### Create a product

```json
{
  "baseUrl": "https://shop.example.com",
  "accessToken": "{{ secrets.MAGENTO_TOKEN }}",
  "operation": "createProduct",
  "productData": {
    "sku": "FUSION-TEST-001",
    "name": "Fusion Test Product",
    "price": 19.99,
    "attribute_set_id": 4,
    "type_id": "simple"
  }
}
```

#### Update a price

```json
{
  "baseUrl": "https://shop.example.com",
  "accessToken": "{{ secrets.MAGENTO_TOKEN }}",
  "operation": "updateProduct",
  "sku": "FUSION-TEST-001",
  "productData": {
    "price": 24.99
  }
}
```

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: security -->
## Security

> Store the access token in Fusion's **Secrets** vault. Do not commit it into version control.

- Give the integration only the resources the workflow needs (e.g. Catalog only for a price sync).
- `createProduct`, `updateProduct` and `createCustomer` change the live store: test on a staging store first.
- Use `https` for the base URL so the token is not sent in clear text.

<!-- /SECTION: security -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Magento API error (401): The consumer isn't authorized to access %resources.`
- **Cause:** Wrong or expired token, or the integration has no access to the resource.
- **Solution:** Create a new token, or add the resource to the integration and activate it again. For integration tokens on 2.4.4+, allow OAuth access tokens as standalone Bearer tokens.

#### Admin token request is refused
- **Cause:** Admin two-factor authentication is enabled.
- **Solution:** Use an integration token instead.

#### `Product data is required` / `Customer data is required`
- **Cause:** The object field is empty.
- **Solution:** Add the keys with **Add entry** and fill their values.

#### `Check these settings: Product data: Invalid record type`
- **Cause:** The field was switched to **Expression** and holds a JSON string, not an object.
- **Solution:** Turn Expression off and use the key/value editor.

#### Invalid JSON format under a value
- **Cause:** A text value was typed without quotes.
- **Solution:** Write text as JSON: `"simple"`, not `simple`.

#### `Magento API error (404): The entity that was requested doesn't exist ...`
- **Cause:** Unknown SKU or order ID. `getOrder` needs the order `entity_id`, not the increment ID (`000000001`).
- **Solution:** Get the right value with `listProducts` or `listOrders`.

#### `Magento API error (400): A customer with the same email address already exists ...`
- **Cause:** The email is already used on this website.
- **Solution:** Use another email, or look up the existing customer.

#### `fetch failed`
- **Cause:** The store URL cannot be reached from the Fusion engine (local URL, tunnel stopped, DNS error).
- **Solution:** Use a public URL for the store.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-11-24 | Full documentation: 7 operations, authentication, key/value fields, output examples, troubleshooting |

<!-- /SECTION: changelog -->
