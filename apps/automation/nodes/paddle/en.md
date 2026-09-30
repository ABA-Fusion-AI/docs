---
node_id: "paddle"
title: "Paddle"
description: "Manage products, plans, customers, subscriptions, transactions, and discount coupons via the Paddle Billing API."
category: "business-commerce"
subcategory: "payments"
version: "1.1.0"
language: "en"
last_updated: "2026-09-30"
author: "Fusion Team"
tags:
  - paddle
  - payments
  - billing
  - subscriptions
  - transactions
  - discounts
  - saas
related_nodes:
  - stripe
  - paypal
  - chargebee
  - manual-trigger
  - function
  - log
---

<!-- SECTION: overview -->
# Paddle

> **Category:** Business & Commerce&nbsp;&nbsp;|&nbsp;&nbsp;**Subgroup:** Payments&nbsp;&nbsp;|&nbsp;&nbsp;**Type:** Action Node

The **Paddle** node connects Fusion workflows directly to the [Paddle Billing API](https://developer.paddle.com/api-reference/overview). Paddle acts as a Merchant of Record, handling global payments, billing subscriptions, recurring charges, and sales tax compliance. 

With this node, you can automate customer onboarding, inspect financial transactions, update or cancel subscriptions, manage your catalog of products and prices, and generate promotional discount coupons on the fly.

### Key Features

- **Subscription Lifecycle Management:** Retrieve subscription details, cancel active subscriptions, or reschedule future billing dates with custom timestamps.
- **Product & Price Catalog Integration:** Query active products and multi-currency pricing plans to synchronize internal catalogs or display live checkout prices.
- **Customer Directory Operations:** Fetch paginated customer records or look up detailed user billing profiles by customer ID.
- **Transaction & Payment Auditing:** Inspect past and pending payment transactions, total amounts, and payment statuses for accounting workflows.
- **Dynamic Coupon & Discount Engine:** Programmatically generate new percentage or flat discount coupons, update active promotions, or query existing discounts.
- **Sandbox & Live Environments:** Seamlessly toggle between Paddle Sandbox (`https://sandbox-api.paddle.com`) for testing and Production (`https://api.paddle.com`) for live transactions.

### Use Cases

- **Self-Service Customer Support:** Automate subscription renewal rescheduling or cancellation when requested through customer portal forms or tickets.
- **Personalized Promotional Campaigns:** Automatically generate bespoke 10% or 20% discount codes for churn-risk customers and email them through communication nodes.
- **Internal Database Synchronization:** Periodically query new customer signups or completed transactions and sync them to internal PostgreSQL, MySQL, or CRM platforms.
- **Subscription Verification:** Verify whether an incoming user ID has an active paid subscription before granting access to premium workflow features.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure your API credentials, execution environment, and the desired Paddle Billing operation in the node parameters panel.

### General Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | Paddle API authentication token (`pdl_live_...` or `pdl_sdbx_...`). Supports expressions. |
| `sandbox` | `boolean` | ❌ No | `false` | When enabled, requests are routed to the Paddle Sandbox API (`https://sandbox-api.paddle.com`). When disabled, requests target the Live API (`https://api.paddle.com`). |
| `operation` | `enum` | ✅ Yes | `listProducts` | The Paddle Billing operation to execute (see operation reference below). |

---

### Operations Reference

The node supports 16 operations across 6 billing entities:

| Category | Operation | HTTP Method & Path | Description |
|----------|-----------|--------------------|-------------|
| **Products** | `listProducts` | `GET /products` | Retrieve a paginated list of products. |
| | `getProduct` | `GET /products/{id}` | Retrieve details for a specific product. |
| **Plans (Prices)** | `listPrices` | `GET /prices` | Retrieve a paginated list of pricing plans. |
| | `getPrice` | `GET /prices/{id}` | Retrieve details for a specific price plan. |
| **Customers** | `listCustomers` | `GET /customers` | Retrieve a paginated list of customer accounts. |
| | `getCustomer` | `GET /customers/{id}` | Retrieve details for a specific customer. |
| **Subscriptions** | `listSubscriptions` | `GET /subscriptions` | Retrieve a paginated list of customer subscriptions. |
| | `getSubscription` | `GET /subscriptions/{id}` | Retrieve details for a specific subscription. |
| | `cancelSubscription` | `POST /subscriptions/{id}/cancel` | Cancel an active subscription. |
| | `rescheduleSubscription` | `PATCH /subscriptions/{id}` | Update the `next_billed_at` renewal date for a subscription. |
| **Transactions** | `listTransactions` | `GET /transactions` | Retrieve a paginated list of payment transactions. |
| | `getTransaction` | `GET /transactions/{id}` | Retrieve details for a specific transaction. |
| **Discounts** | `listDiscounts` | `GET /discounts` | Retrieve a paginated list of discount coupons. |
| | `getDiscount` | `GET /discounts/{id}` | Retrieve details for a specific discount. |
| | `createDiscount` | `POST /discounts` | Create a new percentage or flat discount coupon. |
| | `updateDiscount` | `PATCH /discounts/{id}` | Modify an existing discount coupon. |

---

### Operation-Specific Parameters

Depending on the selected `operation`, additional conditional fields become available:

| Parameter | Type | Required | Default | Description | Relevant Operations |
|-----------|------|----------|---------|-------------|---------------------|
| `id` | `string` | Conditional | — | The unique resource identifier. Prefix varies by entity (e.g. `pro_...`, `pri_...`, `ctm_...`, `sub_...`, `txn_...`, `dsc_...`). | `getProduct`, `getPrice`, `getCustomer`, `getSubscription`, `cancelSubscription`, `rescheduleSubscription`, `getTransaction`, `getDiscount`, `updateDiscount` |
| `perPage` | `number` | ❌ No | `10` | Maximum number of records to return per page in list operations. | `listProducts`, `listPrices`, `listCustomers`, `listSubscriptions`, `listTransactions`, `listDiscounts` |
| `description` | `string` | Conditional | — | Name or description for the discount coupon (e.g. `Summer Flash Sale 20%`). | Required for `createDiscount`; optional for `updateDiscount`. |
| `amount` | `string` | Conditional | — | Value of the discount. For `percentage`, specify the percent (e.g. `20` for 20%). For `flat`, specify the amount in minor currency units (e.g. `1000` for $10.00). | Required for `createDiscount`; optional for `updateDiscount`. |
| `discountType` | `enum` | ❌ No | `percentage` | Calculation mode: `percentage` (proportional reduction) or `flat` (fixed currency reduction). | `createDiscount`, `updateDiscount` |
| `nextBilledAt` | `string` | Conditional | — | New next billing date in RFC 3339 / ISO 8601 format (e.g. `2026-11-15T00:00:00Z`). | Required for `rescheduleSubscription`. |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data from the preceding node. Can be used in expressions to dynamically pass parameters such as `id`, `amount`, or `nextBilledAt`. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the Paddle API request completes successfully (HTTP 200–299). Contains response payload. |
| `error` | `object` | Emitted when the request fails due to invalid parameters, unauthorized API key (401), missing resource (404), or validation error (422). |

---

### Output Schema Examples (`success`)

#### 1. Entity List Operations (e.g. `listProducts`, `listSubscriptions`)

List queries return an array of items under `data` alongside a `meta` pagination object:

```json
{
  "data": [
    {
      "id": "pro_01h8x1v9m8d2k6p4e7w0z",
      "name": "Pro SaaS Plan",
      "description": "Unlimited team access and advanced analytics",
      "tax_category": "standard",
      "status": "active",
      "created_at": "2026-01-15T10:00:00.000Z"
    }
  ],
  "meta": {
    "request_id": "req_01h8x20a8c4f",
    "pagination": {
      "per_page": 10,
      "next": "https://api.paddle.com/products?after=pro_01h8x1v9m8d2k6p4e7w0z",
      "has_more": false,
      "estimated_total": 1
    }
  }
}
```

#### 2. Single Entity Operations (e.g. `getCustomer`, `getSubscription`)

Single-item queries return the resource object directly under `data`:

```json
{
  "data": {
    "id": "sub_01h9y4c2e1f5g8h0j2k4l6m",
    "status": "active",
    "customer_id": "ctm_01h8z7b4a2c1d3e5f6g7h8",
    "address_id": "add_01h8z7c9d1e2f3g4h5j6k7",
    "business_id": null,
    "currency_code": "USD",
    "current_billing_period": {
      "starts_at": "2026-09-01T00:00:00.000Z",
      "ends_at": "2026-10-01T00:00:00.000Z"
    },
    "next_billed_at": "2026-10-01T00:00:00.000Z",
    "items": [
      {
        "price": {
          "id": "pri_01h8x2k4m6n8p0r2t4v6w8",
          "product_id": "pro_01h8x1v9m8d2k6p4e7w0z",
          "unit_price": {
            "amount": "4900",
            "currency_code": "USD"
          }
        },
        "quantity": 1
      }
    ]
  },
  "meta": {
    "request_id": "req_01h9y4d8f2a1b3c4"
  }
}
```

#### 3. Discount Creation (`createDiscount`)

Creating a discount returns the created coupon definition:

```json
{
  "data": {
    "id": "dsc_01h9z3k8m2p4r6t8v0w2x4",
    "status": "active",
    "description": "Black Friday 25% Off",
    "enabled_for_checkout": true,
    "amount": "25",
    "currency_code": null,
    "type": "percentage",
    "recur_until": null,
    "times_used": 0,
    "exempt_from_discounts": false
  },
  "meta": {
    "request_id": "req_01h9z3m1a2b3c4d5"
  }
}
```

---

### Output Schema (`error`)

If Paddle rejects the request or an unhandled exception occurs, an error is emitted:

```json
{
  "error": "Paddle error (404): {\"error\":{\"type\":\"request_error\",\"code\":\"not_found\",\"detail\":\"Subscription sub_999999 was not found\",\"docs_url\":\"https://developer.paddle.com/errors/not_found\"}}"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `error` | `string` | Formatted error message containing the HTTP status code and full JSON diagnostic message returned by Paddle. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

The following example workflow demonstrates triggering an execution manually, querying product catalog records via the **Paddle** node, and displaying the resulting catalog in a Log display node:

```fusion-workflow
src: example.workflow.json
title: Query Paddle products and log results
```

### Complete Workflow Definition

```json
{
  "name": "Paddle",
  "nodes": [
    {
      "id": "manual-trigger",
      "type": "trigger",
      "position": {
        "x": 80,
        "y": 120
      },
      "width": 72,
      "height": 72,
      "data": {
        "name": "manual-trigger",
        "label": "Manual Trigger",
        "inputs": {},
        "outputs": {
          "success": {
            "label": "Success"
          },
          "error": {
            "label": "Error"
          }
        }
      }
    },
    {
      "id": "paddle",
      "type": "action",
      "position": {
        "x": 280,
        "y": 120
      },
      "width": 72,
      "height": 72,
      "data": {
        "name": "paddle",
        "label": "Paddle",
        "inputs": {
          "input": {
            "label": "Input"
          }
        },
        "outputs": {
          "success": {
            "label": "Success"
          },
          "error": {
            "label": "Error"
          }
        }
      }
    },
    {
      "id": "log",
      "type": "display",
      "position": {
        "x": 480,
        "y": 100
      },
      "width": 256,
      "height": 100,
      "data": {
        "name": "log",
        "label": "Log",
        "inputs": {
          "input": {
            "label": "Input"
          }
        },
        "outputs": {
          "success": {
            "label": "Success"
          }
        }
      }
    }
  ],
  "connections": [
    {
      "id": "xy-edge__manual-triggersuccess-paddleinput",
      "type": "directed",
      "source": "manual-trigger",
      "target": "paddle",
      "sourceHandle": "success",
      "targetHandle": "input"
    },
    {
      "id": "xy-edge__paddlesuccess-loginput",
      "type": "directed",
      "source": "paddle",
      "target": "log",
      "sourceHandle": "success",
      "targetHandle": "input"
    }
  ]
}
```

---

### Architecture Patterns

#### Pattern 1: Subscription Renewal Rescheduling

Allow customers to request a temporary billing pause or date adjustment from an internal web portal or ticketing system:

1. **Webhook Trigger:** Receives an approved pause request containing the subscription ID and desired new billing date.
2. **Function Node:** Formats the target date to RFC 3339 format (e.g. `2026-11-01T00:00:00Z`).
3. **Paddle Node:** Executes `rescheduleSubscription` with `id: {{input.subscriptionId}}` and `nextBilledAt: {{input.formattedDate}}`.
4. **Email / Slack Node:** Sends confirmation to the customer with updated renewal dates.

#### Pattern 2: Promotional Churn-Prevention Discount

When an account signals high churn probability, programmatically generate a 30% retention coupon:

1. **CRM / Event Trigger:** Triggers when a cancellation intention is submitted.
2. **Paddle Node:** Executes `createDiscount` with:
   - `description`: `"Retention Offer - 30% Off"`
   - `amount`: `"30"`
   - `discountType`: `"percentage"`
3. **Email Action:** Sends the newly created `{{input.data.id}}` coupon code directly to the customer.

#### Pattern 3: Financial Reconciliation Pipeline

Reconcile Paddle transactions with internal accounting ledgers:

1. **Schedule Trigger:** Triggers daily at midnight.
2. **Paddle Node:** Calls `listTransactions` with `perPage: 50`.
3. **Database Action (PostgreSQL):** Upserts transaction records into the company financial database.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Paddle error (401): authentication_missing / invalid_token`

- **Cause:** The `apiKey` parameter is missing, expired, or invalid.
- **Solution:** 
  1. Confirm your Paddle API key in the [Paddle Developer Console](https://vendors.paddle.com/authentication-v2).
  2. Verify that you are not using a Sandbox key (`pdl_sdbx_...`) with `sandbox: false`, or a Live key (`pdl_live_...`) with `sandbox: true`.

#### `Paddle error (404): not_found`

- **Cause:** The resource specified in the `id` field does not exist or was deleted.
- **Solution:** Verify the resource ID spelling and prefix:
  - Products: `pro_...`
  - Prices: `pri_...`
  - Customers: `ctm_...`
  - Subscriptions: `sub_...`
  - Transactions: `txn_...`
  - Discounts: `dsc_...`

#### `id is required`

- **Cause:** A single-entity operation (such as `getProduct`, `getSubscription`, or `cancelSubscription`) was executed without filling in the `id` field.
- **Solution:** Provide a valid identifier either statically in the node configuration or dynamically via expressions like `{{input.subscriptionId}}`.

#### `nextBilledAt is required (RFC 3339 date string)`

- **Cause:** `rescheduleSubscription` was executed without specifying the `nextBilledAt` parameter, or the date format was invalid.
- **Solution:** Supply a valid RFC 3339 datetime string, including the UTC offset or `Z` indicator (e.g. `2026-11-15T00:00:00Z`).

#### `Paddle error (422): unprocessable_entity`

- **Cause:** A parameter value violates Paddle's validation rules (e.g. invalid discount amount, attempting to set a next billing date in the past, or conflicting discount parameters).
- **Solution:** Inspect the detailed JSON error message emitted on the node's `error` handle to identify the invalid field and correct upstream values.

---

### Error Reference

| Status Code | Paddle Error Type | Possible Cause | Recommended Resolution |
|-------------|-------------------|----------------|------------------------|
| `400` | `bad_request` | Malformed JSON request body or query parameter | Verify parameters and syntax. |
| `401` | `authentication_missing` | Invalid or missing API Bearer Token | Regenerate API key in Paddle Dashboard. |
| `403` | `forbidden` | API key lacks permission for the requested entity | Check API key permission scope. |
| `404` | `not_found` | Resource ID does not exist | Verify ID format and environment (Live vs Sandbox). |
| `422` | `unprocessable_entity` | Business logic validation failure (e.g. date in past) | Ensure dates and amounts follow Paddle rules. |
| `429` | `too_many_requests` | Paddle API rate limit exceeded | Introduce throttling or reduce polling frequency. |
| `500` | `internal_error` | Remote Paddle platform service disruption | Check [Paddle Status Page](https://status.paddle.com/). |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Stripe](./stripe.md) — Alternative payment processing integration
- [PayPal](./paypal.md) — Process orders and payments via PayPal
- [Chargebee](./chargebee.md) — Subscription management integration
- [Manual Trigger](./manual-trigger.md) — Trigger workflows manually for testing
- [Function](./function.md) — Transform, format, and construct API payloads
- [Log](./log.md) — Inspect workflow outputs in the execution log

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.1.0 | 2026-09-30 | Expanded documentation with all 16 operations across products, prices, customers, subscriptions, transactions, and discounts. Added clean workflow preview and troubleshooting guides. |
| 1.0.0 | 2026-08-04 | Initial Paddle billing integration release. |

<!-- /SECTION: changelog -->
