---
node_id: "chargebee"
title: "Chargebee"
description: "Chargebee subscription billing - Manage subscriptions, customers, invoices, and plans."
category: "peer-only"
subcategory: "integrations"
version: "1.0.0"
language: "en"
last_updated: "2026-10-07"
author: "Fusion Team"
tags:
  - chargebee
  - billing
  - subscriptions
  - invoices
  - customers
  - payments
  - peer-only
  - integration
related_nodes:
  - stripe
  - webhook-trigger
  - http-request
---

<!-- SECTION: overview -->
# Chargebee

> **Category:** peer-only | **Subcategory:** integrations | **Type:** Action Node

Automate subscription billing and recurring revenue operations with **Chargebee**.

The **Chargebee** node enables workflows to manage customer profiles, retrieve and generate invoices (including direct PDF download URLs), manage subscriptions (create, get, list, update, cancel, delete), and browse product catalogs across both Product Catalog 1.0 and Product Catalog 2.0.

### Key Capabilities

- **Customer Management**: Create, retrieve, update, and list customer accounts.
- **Invoice Processing**: List invoices, inspect invoice details, and generate secure PDF download URLs.
- **Subscription Lifecycle**: Create subscriptions (supporting Product Catalog 2.0 item prices & legacy plans), retrieve subscription details, cancel at term-end or immediately, and delete subscriptions.
- **Catalog & Pricing**: Query product items and item prices.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `string` | ✅ Yes | `listCustomers` | Operation to perform: `createCustomer`, `getCustomer`, `listCustomers`, `updateCustomer`, `getInvoice`, `listInvoices`, `getInvoicePdfUrl`, `createSubscription`, `getSubscription`, `listSubscriptions`, `updateSubscription`, `cancelSubscription`, `deleteSubscription`, `getItemPrices`. |
| `site` | `string` | ✅ Yes | — | Chargebee site subdomain (e.g. `your-company` or `your-company-test`). Protocol (`https://`) and `.chargebee.com` suffix are automatically sanitized. |
| `apiKey` | `string` | ✅ Yes | — | Chargebee API Key (Full-access or Read-only depending on operation). |
| `customerId` | `string` | ❌ Conditional | — | Customer ID (required for `getCustomer`, `updateCustomer`). |
| `customerFirstName` | `string` | ❌ No | — | Customer first name. |
| `customerLastName` | `string` | ❌ No | — | Customer last name. |
| `customerEmail` | `string` | ❌ No | — | Customer email address. |
| `customerPhone` | `string` | ❌ No | — | Customer phone number. |
| `customerCompany` | `string` | ❌ No | — | Customer company name. |
| `invoiceId` | `string` | ❌ Conditional | — | Invoice ID (required for `getInvoice`, `getInvoicePdfUrl`). |
| `subscriptionId` | `string` | ❌ Conditional | — | Subscription ID (required for `getSubscription`, `updateSubscription`, `cancelSubscription`, `deleteSubscription`). |
| `itemPriceId` | `string` | ❌ Conditional | — | Item Price ID for Product Catalog 2.0 subscription creation/update. |
| `planId` | `string` | ❌ Conditional | — | Legacy Plan ID (PC 1.0) fallback. |
| `cancelEndOfTerm` | `boolean` | ❌ No | `false` | Whether to cancel subscription at the end of the billing term rather than immediately. |
| `limit` | `number` | ❌ No | — | Number of records to return in list operations (max 100). |
| `offset` | `string` | ❌ No | — | Pagination token for retrieving the next page of results. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Workflow payload containing data or variables to bind dynamically. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Response payload returned from Chargebee API. |
| `error` | `Error` | Error thrown if authentication fails, required fields are missing, or Chargebee returns an API error. |

### Output Response Examples

#### 1. Retrieve Customer (`getCustomer`)
```json
{
  "customer": {
    "id": "cbdemo_peter",
    "first_name": "Peter",
    "last_name": "Wright",
    "email": "peter.wright@example.com",
    "company": "Greenplus Enterprises",
    "auto_collection": "on",
    "created_at": 1791386366,
    "card_status": "valid"
  }
}
```

#### 2. Get Invoice PDF URL (`getInvoicePdfUrl`)
```json
{
  "download": {
    "download_url": "https://chargebee-downloads.s3.amazonaws.com/...",
    "valid_till": 1791390000
  }
}
```

#### 3. Create Subscription (`createSubscription`)
```json
{
  "subscription": {
    "id": "sub_16BiJ4VXNSeIi46Ah",
    "customer_id": "cust_123",
    "status": "active",
    "current_term_start": 1791386400,
    "current_term_end": 1794064800,
    "subscription_items": [
      {
        "item_price_id": "enterprise-USD-Monthly",
        "quantity": 1,
        "unit_price": 50000
      }
    ]
  },
  "customer": {
    "id": "cust_123",
    "email": "user@example.com"
  }
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Example Workflows

### Example 1: Generate and Send Invoice PDF
```json
{
  "nodes": [
    {
      "id": "chargebee-invoice",
      "type": "chargebee",
      "config": {
        "operation": "getInvoicePdfUrl",
        "site": "your-site-test",
        "apiKey": "{{secrets.CHARGEBEE_API_KEY}}",
        "invoiceId": "{{input.invoiceId}}"
      }
    },
    {
      "id": "send-email",
      "type": "sendgrid",
      "config": {
        "to": "{{input.customerEmail}}",
        "subject": "Your Invoice is ready",
        "content": "Download your invoice here: {{chargebee-invoice.download.download_url}}"
      }
    }
  ]
}
```

### Example 2: Cancel Subscription on User Request
```json
{
  "nodes": [
    {
      "id": "cancel-sub",
      "type": "chargebee",
      "config": {
        "operation": "cancelSubscription",
        "site": "your-site-test",
        "apiKey": "{{secrets.CHARGEBEE_API_KEY}}",
        "subscriptionId": "{{input.subscriptionId}}",
        "cancelEndOfTerm": true
      }
    }
  ]
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Product Catalog Version Mismatch (400 Bad Request)

**Error Message:**
`"The API endpoint is incompatible with the product catalog version. You are calling product catalog 1.0 API endpoint but you are using product catalog 2.0"`

**Cause:**
Your Chargebee site uses **Product Catalog 2.0** (Items & Item Prices) rather than Legacy Product Catalog 1.0 (Plans).

**Solution:**
- For `createSubscription` and `updateSubscription`, configure `itemPriceId` with the Price ID created under your Product Catalog (e.g. `enterprise-USD-Monthly`).
- Use `getItemPrices` to inspect available price points.

---

### 401 Unauthorized / Invalid API Key

**Cause:**
The provided `apiKey` is invalid or lacks necessary permissions.

**Solution:**
Generate a **Full Access Key** from **Settings > Configure Chargebee > API Keys and Webhooks > API Keys** in your Chargebee dashboard.

---

### Missing Required Identifier

**Cause:**
Operations such as `getCustomer`, `getInvoice`, or `cancelSubscription` require their corresponding IDs (`customerId`, `invoiceId`, `subscriptionId`).

**Solution:**
Ensure the required ID parameter is populated in the configuration or mapped from previous workflow nodes.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: security -->
## Security & Best Practices

- Store API keys securely inside Fusion Secrets (`{{secrets.CHARGEBEE_API_KEY}}`).
- Use separate Chargebee Test Sites (`your-site-test`) during workflow development and staging.
- Set appropriate `limit` parameters for list operations to prevent excessive memory and network usage.

<!-- /SECTION: security -->
