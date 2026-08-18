---
node_id: "lease-vs-buy"
title: "Lease vs Buy Calculator"
description: "Compare leasing vs buying costs for vehicles or equipment"
category: "business-commerce"
subcategory: "finance-accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - lease
  - buy
  - vehicle
  - equipment
  - finance
related_nodes:
  - debt-consolidation
  - line-of-credit
  - loan-calculator
---

<!-- SECTION: header -->
# Lease vs Buy Calculator

> **Category:** Business & Commerce | **Type:** Action Node

Compare the cost of leasing an asset with the cost of purchasing it using financing.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Lease vs Buy Calculator** node compares the financial cost of buying an asset with a loan against leasing the same asset.

For the buy option, the node calculates the financed amount, monthly loan payment, and total purchase cost based on the purchase price, down payment, interest rate, and loan term.

For the lease option, it calculates total lease payments and the total cost when the residual value is included as a buyout option.

The node then recommends either `Buy` or `Lease` by comparing the total purchase cost with the total lease cost including the buyout.

### Key Features

- Calculates the monthly payment for the financed purchase.
- Calculates the total cost of the buy option.
- Calculates total lease payments.
- Includes the configured residual-value buyout in the lease comparison.
- Compares total buy cost with total lease cost including buyout.
- Returns a `Buy` or `Lease` recommendation.
- Supports configurable loan and lease terms.
- Returns structured buy and lease results.

### Processing Flow

```text
Purchase price
  ↓
Down payment
  ↓
Loan rate and term
  ↓
Calculate financed purchase payment
  ↓
Calculate total buy cost
  ↓
Monthly lease and lease term
  ↓
Calculate total lease cost
  ↓
Add residual value for buyout comparison
  ↓
Compare total costs
  ↓
Return recommendation
```

### Use Cases

- Comparing vehicle leasing with financing a purchase.
- Comparing equipment leasing with purchasing.
- Estimating financed purchase costs.
- Estimating total lease payments.
- Evaluating a lease buyout option.
- Supporting lease-versus-buy financial decisions.
- Preparing comparison results for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `purchasePrice` | `number` | No | `30000` | Purchase price of the asset. Must be greater than or equal to `0`. |
| `downPayment` | `number` | No | `5000` | Upfront payment applied to the purchase price. Must be greater than or equal to `0`. |
| `loanRate` | `number` | No | `5` | Annual loan interest rate as a percentage. Must be between `0` and `100`. |
| `loanTerm` | `number` | No | `60` | Loan term in months. Must be between `1` and `120`. |
| `monthlyLease` | `number` | No | `300` | Monthly lease payment. Must be greater than or equal to `0`. |
| `leaseTerm` | `number` | No | `36` | Lease term in months. Must be between `1` and `120`. |
| `residualValue` | `number` | No | `15000` | Residual value used as the lease buyout amount. Must be greater than or equal to `0`. |

### Purchase Price

Provide the total purchase price of the vehicle or equipment.

Example:

```text
30000
```

### Down Payment

Provide the upfront amount paid toward the purchase.

Example:

```text
5000
```

The financed amount is calculated as:

```text
loanAmount = purchasePrice - downPayment
```

For the default configuration:

```text
loanAmount = 30000 - 5000
loanAmount = 25000
```

### Loan Rate

Provide the annual interest rate as a percentage.

Example:

```text
5
```

The node converts the annual rate to a monthly rate:

```text
monthlyRate = loanRate / 100 / 12
```

### Loan Term

Provide the number of monthly payments for the financed purchase.

Example:

```text
60
```

### Monthly Lease

Provide the monthly lease payment.

Example:

```text
300
```

### Lease Term

Provide the lease duration in months.

Example:

```text
36
```

### Residual Value

Provide the amount required to purchase the asset at the end of the lease.

Example:

```text
15000
```

### Calculation Details

Financed amount:

```text
loanAmount = purchasePrice - downPayment
```

Monthly loan rate:

```text
monthlyRate = loanRate / 100 / 12
```

Monthly purchase payment:

```text
monthlyPayment =
  loanAmount ×
  (monthlyRate × (1 + monthlyRate)^loanTerm) /
  ((1 + monthlyRate)^loanTerm - 1)
```

Total buy cost:

```text
totalBuyCost = downPayment + monthlyPayment × loanTerm
```

Total lease cost:

```text
totalLeaseCost = monthlyLease × leaseTerm
```

Total lease cost with buyout:

```text
buyoutCost = totalLeaseCost + residualValue
```

Recommendation:

```text
if totalBuyCost < buyoutCost
  recommendation = "Buy"
else
  recommendation = "Lease"
```

The recommendation therefore compares the total financed purchase cost with the lease cost including the residual-value buyout.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `purchasePrice`
- `downPayment`
- `loanRate`
- `loanTerm`
- `monthlyLease`
- `leaseTerm`
- `residualValue`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- buy-option details;
- lease-option details;
- a `Buy` or `Lease` recommendation.

Example:

```json
{
  "success": true,
  "buy": {
    "downPayment": 5000,
    "monthlyPayment": 471.7808411002747,
    "totalPayments": 60,
    "totalCost": 33306.85046601648,
    "ownership": "You own the asset",
    "equity": 30000
  },
  "lease": {
    "monthlyPayment": 300,
    "totalPayments": 36,
    "totalCost": 10800,
    "buyoutOption": 15000,
    "totalWithBuyout": 25800,
    "ownership": "No ownership unless bought out"
  },
  "recommendation": "Lease"
}
```

### Buy

The `buy` object contains:

| Field | Description |
|-------|-------------|
| `downPayment` | Upfront payment applied to the purchase. |
| `monthlyPayment` | Calculated monthly loan payment. |
| `totalPayments` | Number of monthly loan payments. |
| `totalCost` | Down payment plus all calculated loan payments. |
| `ownership` | Ownership description returned by the node. |
| `equity` | Purchase price returned by the node in the `equity` field. |

### Lease

The `lease` object contains:

| Field | Description |
|-------|-------------|
| `monthlyPayment` | Configured monthly lease payment. |
| `totalPayments` | Number of lease payments. |
| `totalCost` | Monthly lease payment multiplied by the lease term. |
| `buyoutOption` | Configured residual value. |
| `totalWithBuyout` | Total lease payments plus residual value. |
| `ownership` | Ownership description returned by the node. |

### Recommendation

The node returns:

```text
Buy
```

when:

```text
buy.totalCost < lease.totalWithBuyout
```

Otherwise it returns:

```text
Lease
```

If both values are equal, the current implementation returns `Lease`.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Lease Recommendation

**Configuration**

```text
purchasePrice: 30000
downPayment: 5000
loanRate: 5
loanTerm: 60
monthlyLease: 300
leaseTerm: 36
residualValue: 15000
```

**Output**

```json
{
  "success": true,
  "buy": {
    "downPayment": 5000,
    "monthlyPayment": 471.7808411002747,
    "totalPayments": 60,
    "totalCost": 33306.85046601648,
    "ownership": "You own the asset",
    "equity": 30000
  },
  "lease": {
    "monthlyPayment": 300,
    "totalPayments": 36,
    "totalCost": 10800,
    "buyoutOption": 15000,
    "totalWithBuyout": 25800,
    "ownership": "No ownership unless bought out"
  },
  "recommendation": "Lease"
}
```

In this example:

```text
Buy total cost:        33306.85046601648
Lease with buyout:     25800
Recommendation:        Lease
```

### Example 2: Buy Recommendation

**Configuration**

```text
purchasePrice: 30000
downPayment: 5000
loanRate: 5
loanTerm: 60
monthlyLease: 700
leaseTerm: 60
residualValue: 15000
```

**Output**

```json
{
  "success": true,
  "buy": {
    "downPayment": 5000,
    "monthlyPayment": 471.7808411002747,
    "totalPayments": 60,
    "totalCost": 33306.85046601648,
    "ownership": "You own the asset",
    "equity": 30000
  },
  "lease": {
    "monthlyPayment": 700,
    "totalPayments": 60,
    "totalCost": 42000,
    "buyoutOption": 15000,
    "totalWithBuyout": 57000,
    "ownership": "No ownership unless bought out"
  },
  "recommendation": "Buy"
}
```

In this example:

```text
Buy total cost:        33306.85046601648
Lease with buyout:     57000
Recommendation:        Buy
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Lease vs Buy Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Purchase Price Is Invalid

**Cause:** `purchasePrice` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Down Payment Is Invalid

**Cause:** `downPayment` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Down Payment Exceeds Purchase Price

The schema allows `downPayment` to be greater than `purchasePrice`.

In that case:

```text
loanAmount = purchasePrice - downPayment
```

produces a negative financed amount.

**Solution:** For a standard financed purchase comparison, use a `downPayment` that does not exceed `purchasePrice`.

### Loan Rate Is Invalid

**Cause:** `loanRate` is outside the supported schema range.

**Solution:** Use a value between `0` and `100`.

### Loan Rate Is Zero

The schema accepts a `loanRate` of `0`, but the current monthly-payment formula does not contain a separate zero-interest calculation.

For standard calculations with the current implementation, use a positive loan rate.

### Loan Term Is Invalid

**Cause:** `loanTerm` is outside the supported range.

**Solution:** Use a value between `1` and `120`.

### Monthly Lease Is Invalid

**Cause:** `monthlyLease` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Lease Term Is Invalid

**Cause:** `leaseTerm` is outside the supported range.

**Solution:** Use a value between `1` and `120`.

### Residual Value Is Invalid

**Cause:** `residualValue` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Unexpected Recommendation

The recommendation is based only on:

```text
buy.totalCost
```

compared with:

```text
lease.totalWithBuyout
```

The current implementation does not include additional costs such as maintenance, taxes, insurance, fees, depreciation, or investment opportunity cost.

Verify all configured values before using the comparison.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Debt Consolidation Calculator** — Compare existing debt payments with a consolidated loan.
- **Line of Credit** — Work with line-of-credit calculations.
- **Loan Calculator** — Calculate loan payments and interest.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial documentation for the Lease vs Buy Calculator node. |

<!-- /SECTION: changelog -->