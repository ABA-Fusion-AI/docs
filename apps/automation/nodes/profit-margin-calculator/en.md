---
node_id: "profit-margin-calculator"
title: "Profit Margin Calculator"
description: "Calculate gross, operating, and net profit margins with markup analysis"
category: "mathematical-statistical-analysis"
subcategory: "calculators-models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - profit
  - margin
  - markup
  - revenue
  - finance
related_nodes:
  - break-even-calculator
  - budget-variance
  - financial-ratio-analyzer
---

<!-- SECTION: header -->
# Profit Margin Calculator

> **Category:** Mathematical & Statistical Analysis | **Type:** Action Node

Calculate gross, operating, and net profit margins with markup analysis.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Profit Margin Calculator** node calculates gross profit, operating profit, net profit, their corresponding margins, and markup based on revenue, cost of goods sold, operating expenses, and taxes.

### Key Features

- Calculates gross profit.
- Calculates gross margin.
- Calculates operating profit.
- Calculates operating margin.
- Calculates net profit.
- Calculates net margin.
- Calculates markup based on cost of goods sold.
- Returns structured financial calculation results.

### Processing Flow

```text
Revenue
  ↓
Cost of goods sold
  ↓
Operating expenses
  ↓
Taxes
  ↓
Calculate gross profit
  ↓
Calculate operating profit
  ↓
Calculate net profit
  ↓
Calculate margins and markup
  ↓
Return result
```

### Use Cases

- Measuring product or business profitability.
- Calculating gross margin.
- Calculating operating margin.
- Calculating net profit margin.
- Calculating markup over cost of goods sold.
- Preparing profitability metrics for downstream workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `revenue` | `number` | No | `100000` | Total revenue. The schema accepts values greater than or equal to `0`; use a value greater than `0` for meaningful margin calculations. |
| `cogs` | `number` | No | `60000` | Cost of goods sold. The schema accepts values greater than or equal to `0`; use a value greater than `0` for meaningful markup calculations. |
| `operatingExpenses` | `number` | No | `20000` | Operating expenses. Must be greater than or equal to `0`. |
| `taxes` | `number` | No | `5000` | Taxes deducted from operating profit. Must be greater than or equal to `0`. |

### Revenue

Provide the total revenue.

Example:

```text
100000
```

### Cost of Goods Sold

Provide the cost of goods sold.

Example:

```text
60000
```

### Operating Expenses

Provide operating expenses.

Example:

```text
20000
```

### Taxes

Provide the tax amount deducted from operating profit.

Example:

```text
5000
```

### Calculation Details

Gross profit:

```text
grossProfit = revenue - cogs
```

Gross margin:

```text
grossMargin = grossProfit / revenue × 100
```

Operating profit:

```text
operatingProfit = grossProfit - operatingExpenses
```

Operating margin:

```text
operatingMargin = operatingProfit / revenue × 100
```

Net profit:

```text
netProfit = operatingProfit - taxes
```

Net margin:

```text
netMargin = netProfit / revenue × 100
```

Markup:

```text
markup = (revenue - cogs) / cogs × 100
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses configured values from:

- `revenue`
- `cogs`
- `operatingExpenses`
- `taxes`

Incoming workflow data is not used for the calculation.

### Output

The node returns an object containing:

- calculation status;
- revenue;
- cost of goods sold;
- gross profit;
- gross margin;
- operating expenses;
- operating profit;
- operating margin;
- taxes;
- net profit;
- net margin;
- markup.

Example:

```json
{
  "success": true,
  "revenue": 250000,
  "cogs": 150000,
  "grossProfit": 100000,
  "grossMargin": 40,
  "operatingExpenses": 50000,
  "operatingProfit": 50000,
  "operatingMargin": 20,
  "taxes": 10000,
  "netProfit": 40000,
  "netMargin": 16,
  "markup": 66.66666666666666
}
```

### Gross Margin

For:

```text
revenue: 250000
grossProfit: 100000
```

the gross margin is:

```text
40
```

### Net Margin

For:

```text
revenue: 250000
netProfit: 40000
```

the net margin is:

```text
16
```

### Markup

For:

```text
revenue: 250000
cogs: 150000
```

the markup is approximately:

```text
66.66666666666666
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Default Profit Margin Calculation

**Configuration**

```text
revenue: 100000
cogs: 60000
operatingExpenses: 20000
taxes: 5000
```

**Output**

```json
{
  "success": true,
  "revenue": 100000,
  "cogs": 60000,
  "grossProfit": 40000,
  "grossMargin": 40,
  "operatingExpenses": 20000,
  "operatingProfit": 20000,
  "operatingMargin": 20,
  "taxes": 5000,
  "netProfit": 15000,
  "netMargin": 15,
  "markup": 66.66666666666666
}
```

### Example 2: Higher Revenue

**Configuration**

```text
revenue: 250000
cogs: 150000
operatingExpenses: 50000
taxes: 10000
```

**Output**

```json
{
  "success": true,
  "revenue": 250000,
  "cogs": 150000,
  "grossProfit": 100000,
  "grossMargin": 40,
  "operatingExpenses": 50000,
  "operatingProfit": 50000,
  "operatingMargin": 20,
  "taxes": 10000,
  "netProfit": 40000,
  "netMargin": 16,
  "markup": 66.66666666666666
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Profit Margin Calculator Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Revenue Is Invalid

**Cause:** `revenue` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Revenue Is Zero

**Cause:** `revenue` is `0`, which prevents meaningful gross, operating, and net margin percentages from being calculated.

**Solution:** Use a `revenue` value greater than `0` when margin percentage calculations are required.

### Cost of Goods Sold Is Invalid

**Cause:** `cogs` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Cost of Goods Sold Is Zero

**Cause:** `cogs` is `0`, which prevents a meaningful markup percentage from being calculated.

**Solution:** Use a `cogs` value greater than `0` when markup analysis is required.

### Operating Expenses Are Invalid

**Cause:** `operatingExpenses` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Taxes Are Invalid

**Cause:** `taxes` is below `0`.

**Solution:** Use a value greater than or equal to `0`.

### Unexpected Profit Margin Result

**Cause:** One or more configured financial values may not match the intended calculation.

**Solution:** Verify `revenue`, `cogs`, `operatingExpenses`, and `taxes`.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Break-Even Calculator** — Calculate break-even point and contribution margin.
- **Budget Variance Analysis** — Compare budgeted and actual spending.
- **Financial Ratio Analyzer** — Analyze financial ratios.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial documentation for the Profit Margin Calculator node. |

<!-- /SECTION: changelog -->