---
node_id: "transfer-pricing"
title: "Transfer Pricing Calculator"
description: "Calculate transfer prices using cost-plus, resale-price, or market-price methods"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - transfer-pricing
  - cost-plus
  - resale-price
  - market-price
  - finance
  - accounting
  - pricing-model
related_nodes:
  - profit-margin-calculator
  - break-even-calculator
  - financial-ratio-analyzer
  - budget-variance
---

<!-- SECTION: header -->
# Transfer Pricing Calculator

> **Category:** Mathematical & Statistical Analysis | **Subcategory:** Calculators & Models | **Type:** Action Node

Calculate intercompany transfer prices and profit margins using standard Cost-Plus, Resale Price, and Market Price methods.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Transfer Pricing Calculator** node determines the price charged between related business entities or international subsidiaries for goods, services, or intellectual property. It implements standard transfer pricing methodologies—such as **Cost-Plus**, **Resale Price**, and **Market Price**—ensuring transparent pricing rules and cross-border compliance.

Given a base cost and an applicable markup or target margin, the node automatically computes the final transfer price, nominal profit margin, percentage margin, and an audit explanation.

### Key Pricing Methodologies

1. **Cost-Plus Method (`cost-plus`):** Adds a predetermined markup percentage on top of production or acquisition costs.
   $$\text{Transfer Price} = \text{Cost Basis} \times \left(1 + \frac{\text{Markup}}{100}\right)$$
2. **Resale Price Method (`resale-price`):** Determines the transfer price by backing out an appropriate gross margin percentage from the final resale price.
   $$\text{Transfer Price} = \frac{\text{Cost Basis}}{1 - \frac{\text{Markup}}{100}}$$
3. **Market Price Method (`market-price`):** Sets the transfer price directly to the prevailing uncontrolled market rate.
   $$\text{Transfer Price} = \text{Cost Basis}$$

### Key Features

- **Three Standard Methods:** Seamlessly switch between Cost-Plus, Resale Price, and Market Price calculations.
- **Automated Profit Metrics:** Calculates nominal profit amount and percentage profit margin.
- **Audit Explanation:** Generates human-readable descriptions of the pricing logic for invoices and ERP audits.
- **Zero Latency:** Computes pricing formulas in-memory with instant execution.

### Processing Flow

```text
Workflow Trigger / Upstream ERP Data
  ↓
Validate Parameters (costBasis ≥ 0, markup between 0–1000%, method)
  ↓
Apply Pricing Formula (Cost-Plus, Resale Price, or Market Price)
  ↓
Compute Profit Margin & Percentage
  ↓
Emit Output Envelope { transferPrice, profitMargin, profitMarginPercent, explanation }
```

### Use Cases

- **Intercompany Invoicing:** Automatically generate transfer prices for cross-subsidiary billing and ERP synchronizations.
- **Supply Chain Costing:** Calculate internal billing rates when products move between manufacturing plants and regional distributors.
- **Tax Compliance & Documentation:** Ensure consistent markup rules across accounting departments to satisfy arm's length standard audits.
- **Dynamic Pricing Workflows:** Adjust transfer prices dynamically based on fluctuating raw material costs or market benchmarks.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `costBasis` | `number` | No | `100` | The baseline production cost, purchase cost, or reference market price (must be $\ge 0$). |
| `markup` | `number` | No | `20` | Markup or margin percentage (value between `0` and `1000`). |
| `method` | `enum` | No | `cost-plus` | The transfer pricing methodology: `cost-plus`, `resale-price`, or `market-price`. |

### Method Comparison

| Method | Formula | When to Use |
|--------|---------|-------------|
| `cost-plus` | `costBasis * (1 + markup / 100)` | Manufacturing entities selling semi-finished or finished goods to sales affiliates. |
| `resale-price` | `costBasis / (1 - markup / 100)` | Distributors buying goods from a parent company to resell to external third parties. |
| `market-price` | `costBasis` | Commodities or products with active, transparent open-market pricing benchmarks. |

### Default Configuration

```json
{
  "costBasis": 100,
  "markup": 20,
  "method": "cost-plus"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `unknown` | Incoming workflow trigger or data passed from upstream nodes (e.g. ERP purchase order or invoice). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Returned on successful calculation, containing the transfer price, profit margin, and calculation breakdown. |
| `error` | `object` | Returned if invalid parameters or calculation errors occur. |

### Output Schema

```typescript
{
  success: boolean;               // Always true when calculation succeeds
  costBasis: number;              // Initial cost basis value
  markup: number;                 // Configured markup percentage
  method: string;                 // Selected pricing method ("cost-plus", "resale-price", "market-price")
  transferPrice: number;          // Calculated intercompany transfer price
  profitMargin: number;           // Nominal profit amount (transferPrice - costBasis)
  profitMarginPercent: number;    // Percentage profit margin on cost basis
  explanation: string;            // Text summary of the applied formula
}
```

### Successful Response Example

```json
{
  "success": true,
  "costBasis": 100,
  "markup": 20,
  "method": "cost-plus",
  "transferPrice": 120,
  "profitMargin": 20,
  "profitMarginPercent": 20,
  "explanation": "Cost basis + 20% markup"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Cost-Plus Pricing (20% Markup)

Calculate transfer price for a manufactured item with a $100 base cost:

```json
{
  "method": "cost-plus",
  "costBasis": 100,
  "markup": 20
}
```

**Output:**
```json
{
  "success": true,
  "costBasis": 100,
  "markup": 20,
  "method": "cost-plus",
  "transferPrice": 120,
  "profitMargin": 20,
  "profitMarginPercent": 20,
  "explanation": "Cost basis + 20% markup"
}
```

### Example 2: Resale Price Method

Calculate the internal transfer price for an item with an $80 cost basis targeting a 20% margin:

```json
{
  "method": "resale-price",
  "costBasis": 80,
  "markup": 20
}
```

**Output:**
```json
{
  "success": true,
  "costBasis": 80,
  "markup": 20,
  "method": "resale-price",
  "transferPrice": 100,
  "profitMargin": 20,
  "profitMarginPercent": 25,
  "explanation": "Resale price - 20% margin"
}
```

### Example 3: Market Price Method

Use external open-market commodity benchmark price ($150):

```json
{
  "method": "market-price",
  "costBasis": 150,
  "markup": 0
}
```

**Output:**
```json
{
  "success": true,
  "costBasis": 150,
  "markup": 0,
  "method": "market-price",
  "transferPrice": 150,
  "profitMargin": 0,
  "profitMarginPercent": 0,
  "explanation": "Market-based pricing"
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Calculate intercompany transfer prices using different methods
```

### Common Workflow Patterns

- **Automated ERP Invoicing:** Webhook Trigger (New Intercompany Order) → Transfer Pricing Calculator → ERP / QuickBooks (Create Intercompany Invoice).
- **Cost Margin Validation:** Database Trigger → Transfer Pricing Calculator → If/Else (Check if `{{ $node["Transfer Pricing Calculator"].profitMarginPercent }} >= 15`) → Notification / Approval.
- **Multi-Jurisdiction Tax Report:** Scheduled Trigger → Transfer Pricing Calculator → Google Sheets / Excel (Append transfer pricing log).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Resale Price division by zero / Infinity

**Cause:** Setting `markup` to `100%` or higher under the `resale-price` method causes division by zero in $(1 - \text{markup}/100)$.

**Solution:** In the `resale-price` method, markup represents the target gross margin rate, which must be strictly less than `100` (e.g. `10%`–`50%`).

### Cost basis is zero

**Cause:** A `costBasis` of `0` was supplied.

**Solution:** Ensure the `costBasis` field is populated with a positive numerical cost value.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Profit Margin Calculator](../profit-margin-calculator/en.md) - Calculate gross, operating, and net profit margins
- [Break-Even Calculator](../break-even-calculator/en.md) - Determine break-even sales volume and revenue
- [Financial Ratio Analyzer](../financial-ratio-analyzer/en.md) - Evaluate liquidity, solvency, and operational ratios

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation for Transfer Pricing Calculator node |

<!-- /SECTION: changelog -->
