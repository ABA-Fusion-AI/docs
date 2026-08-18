---
node_id: "payment-gateway-comparison"
title: "Payment Gateway Comparison"
description: "Compare payment gateway fees across Stripe, PayPal, and Square"
category: "Business & Commerce"
subcategory: "Payments"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - payment-gateway
  - stripe
  - paypal
  - square
  - fee-comparison
  - payments
  - merchant-fees
  - transaction-fees
  - e-commerce
  - finance
related_nodes:
  - stripe-charges
  - paypal-create-order
  - square
  - profit-margin-calculator
  - salary-converter
  - function
  - log
---

<!-- SECTION: header -->
# Payment Gateway Comparison

> **Category:** Business & Commerce | **Subcategory:** Payments | **Type:** Action Node

Compare merchant transaction fees, effective processing rates, and net payouts across major payment processors—including **Stripe**, **PayPal**, and **Square**—for any transaction or order amount.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Payment Gateway Comparison** node provides automated merchant fee analysis across top payment service providers (PSPs). Every payment gateway applies a unique pricing structure consisting of variable percentage rates combined with fixed per-transaction fees (for example, `2.9% + $0.30` vs `3.49% + $0.49`).

Because fixed fees disproportionately affect micro-transactions and variable percentages dominate large transactions, selecting the most cost-effective gateway depends heavily on individual cart totals. This node calculates the exact processing fee, net merchant payout, and effective fee percentage for each gateway side-by-side, identifying the lowest-cost provider and potential cost savings.

### Key Features

- **Side-by-Side Gateway Comparison:** Simultaneously computes transaction costs for **Stripe**, **PayPal**, and **Square**.
- **Accurate Fee Breakdown:** Breaks down both the variable percentage charge and the fixed per-transaction fee for each provider.
- **Net Payout Calculation:** Computes the exact amount deposited into your merchant account after fees are deducted (`amount - fee`).
- **Effective Fee Rate:** Calculates the real-world percentage cost (`fee / amount * 100`) to highlight fee efficiency across varying order sizes.
- **Optimal Gateway Identification:** Automatically highlights the provider offering the lowest fee and highest merchant payout.
- **Dynamic Expression Support:** Bind dynamic order totals directly from incoming checkout webhooks, shopping carts, or invoicing nodes using `{{outputs.NodeName.success.amount}}`.

### Comparison Logic & Flow

```text
Incoming Transaction Amount (e.g. $100.00)
                    ↓
   Apply Gateway Fee Calculation Models
 ┌──────────────────┼──────────────────┐
 ↓                  ↓                  ↓
Stripe:            PayPal:            Square:
2.9% + $0.30       3.49% + $0.49      2.9% + $0.30
$3.20 fee          $3.98 fee          $3.20 fee
Net: $96.80        Net: $96.02        Net: $96.80
 └──────────────────┼──────────────────┘
                    ↓
  Evaluate Summary & Cost Savings
  - Cheapest Gateway: Stripe / Square ($3.20)
  - Potential Savings: $0.78 vs PayPal
                    ↓
  Structured Output Payload (Gateways & Summary)
```

### Use Cases

- **Dynamic Smart Checkout Routing:** Route customer payments dynamically to the gateway that incurs the lowest processing fee based on cart value.
- **E-Commerce Fee Analytics & Auditing:** Audit historical orders to evaluate how much money your business would save by switching default gateways.
- **Merchant Pricing & Surcharge Modeling:** Calculate appropriate checkout surcharges or adjust retail pricing to ensure target profit margins after processing fees.
- **Billing & SaaS Invoicing:** Automatically determine payment method recommendations (e.g. credit card vs bank transfer vs PayPal) based on invoice size.
- **Marketplace Payout Optimization:** Calculate accurate net vendor payouts when distributing marketplace commission shares across multi-vendor platforms.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `amount` | `number` | ✅ Yes | `0` | The transaction or order amount to evaluate (e.g., `10`, `100`, `1500`). Must be greater than or equal to `0`. |

---

### Parameter Details

#### `amount`
The gross transaction value in base currency units (e.g., USD, EUR) to be evaluated across all payment processors.
- **Type:** `number`
- **Required:** Yes
- **Default:** `0`
- **Validation:** Must be a non-negative number (`>= 0`).
- **Expression Enabled:** Yes (e.g., `{{outputs.Webhook.success.body.total}}`)
- **Example Values:** `9.99`, `49.50`, `100`, `1250.00`

---

### Supported Gateway Fee Schedules

The node evaluates standard commercial online transaction pricing:

| Gateway | Pricing Structure | Variable Rate | Fixed Fee | Best Suited For |
|---------|-------------------|:-------------:|:---------:|-----------------|
| **Stripe** | Standard Online Cards | `2.90%` | `$0.30` | Developer platforms, subscription SaaS, general e-commerce |
| **PayPal** | Standard Commercial / Checkout | `3.49%` | `$0.49` | High-trust brand checkout, consumer marketplaces |
| **Square** | Standard Online Payments | `2.90%` | `$0.30` | Omnichannel retail, integrated POS and web storefronts |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution payload. Can supply dynamic `amount` values via expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when calculation completes successfully. Contains fee breakdowns for each gateway and the comparison summary. |
| `error` | `Error` | Emitted if `amount` is invalid, negative, or not a numeric value. |

---

### Output Data Structure

When an amount of `$100.00` is evaluated, the `success` output returns the following structured JSON:

```json
{
  "amount": 100.00,
  "gateways": {
    "stripe": {
      "name": "Stripe",
      "rate": "2.9% + $0.30",
      "fee": 3.20,
      "netPayout": 96.80,
      "effectiveRate": "3.20%"
    },
    "paypal": {
      "name": "PayPal",
      "rate": "3.49% + $0.49",
      "fee": 3.98,
      "netPayout": 96.02,
      "effectiveRate": "3.98%"
    },
    "square": {
      "name": "Square",
      "rate": "2.9% + $0.30",
      "fee": 3.20,
      "netPayout": 96.80,
      "effectiveRate": "3.20%"
    }
  },
  "summary": {
    "cheapestGateway": "Stripe",
    "lowestFee": 3.20,
    "highestPayout": 96.80,
    "maxFee": 3.98,
    "potentialSavings": 0.78
  }
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `amount` | `number` | The evaluated gross transaction amount. |
| `gateways` | `object` | Map containing individual calculation results for each payment processor. |
| `gateways.<gateway>.name` | `string` | Display name of the payment gateway (`Stripe`, `PayPal`, `Square`). |
| `gateways.<gateway>.rate` | `string` | Standard fee formula applied for the calculation. |
| `gateways.<gateway>.fee` | `number` | Total processing fee deducted by the gateway for this transaction. |
| `gateways.<gateway>.netPayout` | `number` | Net revenue deposited to the merchant (`amount - fee`). |
| `gateways.<gateway>.effectiveRate` | `string` | Total fee expressed as an effective percentage of the gross amount. |
| `summary` | `object` | Aggregated comparison metrics and cost optimization insights. |
| `summary.cheapestGateway` | `string` | Name of the payment gateway with the lowest processing fee. |
| `summary.lowestFee` | `number` | The lowest fee amount among all compared processors. |
| `summary.highestPayout` | `number` | The maximum net amount retained by the merchant. |
| `summary.maxFee` | `number` | The highest fee amount among the compared processors. |
| `summary.potentialSavings` | `number` | Difference between the highest and lowest fee (`maxFee - lowestFee`). |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Micro-Transaction ($10.00 Order)

Demonstrates how fixed fees significantly impact smaller transaction sizes.

**Parameter Configuration:**

```text
Amount: 10.00
```

**Calculation Breakdown:**
- **Stripe:** `(10.00 * 0.029) + 0.30` = **$0.59** (Effective Rate: 5.90%, Net: $9.41)
- **PayPal:** `(10.00 * 0.0349) + 0.49` = **$0.84** (Effective Rate: 8.39%, Net: $9.16)
- **Square:** `(10.00 * 0.029) + 0.30` = **$0.59** (Effective Rate: 5.90%, Net: $9.41)
- **Outcome:** Stripe and Square save **$0.25 (29.8%)** compared to PayPal.

---

### Example 2: Standard E-Commerce Order ($100.00)

A typical retail shopping cart total.

**Parameter Configuration:**

```text
Amount: 100.00
```

**Calculation Breakdown:**
- **Stripe:** **$3.20** fee | Net: **$96.80** | Effective: **3.20%**
- **PayPal:** **$3.98** fee | Net: **$96.02** | Effective: **3.98%**
- **Square:** **$3.20** fee | Net: **$96.80** | Effective: **3.20%**
- **Savings:** **$0.78** per transaction by choosing Stripe/Square over PayPal.

---

### Example 3: High-Ticket B2B Invoice ($2,500.00)

Examines how percentage rates scale with high-value transactions.

**Parameter Configuration:**

```text
Amount: 2500.00
```

**Calculation Breakdown:**
- **Stripe:** `(2500 * 0.029) + 0.30` = **$72.80** (Effective: 2.91%)
- **PayPal:** `(2500 * 0.0349) + 0.49` = **$87.74** (Effective: 3.51%)
- **Square:** `(2500 * 0.029) + 0.30` = **$72.80** (Effective: 2.91%)
- **Savings:** **$14.94** saved on a single transaction.

---

### Example 4: Dynamic Smart Gateway Router

Automatically route checkout orders to the optimal gateway or present fee warnings to admin.

**Workflow Pipeline:**

```text
Manual Trigger / Webhook Trigger (Receives: { "orderTotal": 150.00 })
  → Payment Gateway Comparison (amount: {{outputs.WebhookTrigger.success.orderTotal}})
  → Function (Select cheapestGateway)
  → Stripe / PayPal Action (Create Payment Session)
  → Log (Display calculated savings)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Payment Gateway Comparison Workflow
```

### How it flows

1. **Manual Trigger:** Initiates the workflow execution manually.
2. **Payment Gateway Comparison:** Receives the transaction amount (default: `0` or passed dynamically) and calculates fee breakdowns for Stripe, PayPal, and Square.
3. **Log Node:** Displays the structured results, lowest fee, and net payout summary in the execution console.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **Account for Payment Method Variations:** Standard card rates differ from ACH/bank transfers (typically `0.8%` capped at `$5.00` on Stripe) or international cards (typically `+1.0%` to `+1.5%`). Use this node as a baseline for credit card transactions.
2. **Micro-Transaction Optimization:** If selling digital goods under `$5.00`, consider specialized micro-payment pricing models or batching transactions to minimize the impact of fixed `$0.30`–`$0.49` fees.
3. **Volume Discounts:** Enterprises with monthly processing volumes exceeding `$80,000`–`$100,000` often qualify for custom interchange-plus pricing from Stripe and Square.
4. **Automate Surcharge Calculations:** Combine this node with **Profit Margin Calculator** to ensure merchant processing overhead is factored into wholesale and retail pricing.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Calculation Returns `$0` Fees
- **Symptom:** All fees and payouts return `0`.
- **Cause:** The `amount` parameter is set to `0` or not supplied.
- **Solution:** Provide a positive numeric value for `amount` (e.g. `amount: 100`).

#### Unexpected String Concatenation
- **Symptom:** Calculation results in `NaN` or incorrect values.
- **Cause:** Passing a string with currency symbols (e.g. `"$100.00"` or `"100 EUR"`) into `amount`.
- **Solution:** Strip non-numeric currency characters before passing values to the node (e.g., using `parseFloat(input.replace(/[^0-9.]/g, ''))`).

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Amount must be a number` | Passed a non-numeric string or object | Ensure `amount` is a valid JavaScript number |
| `Amount cannot be negative` | Input amount is less than `0` | Pass a non-negative number (`>= 0`) |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [Stripe Charges](../stripe-charges/en.md) — Create and manage Stripe payment charges
- [PayPal Create Order](../paypal-create-order/en.md) — Create PayPal checkout orders
- [Square](../square/en.md) — Process payments and manage inventory via Square
- [Profit Margin Calculator](../profit-margin-calculator/en.md) — Calculate profit margins and cost markups
- [Salary Converter](../salary-converter/en.md) — Convert compensation rates across multiple pay frequencies
- [Function](../function/en.md) — Apply custom discount, tax, or surcharge logic
- [Log](../log/en.md) — Print calculation results to the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial release with multi-gateway fee comparison across Stripe, PayPal, and Square |

<!-- /SECTION: changelog -->
