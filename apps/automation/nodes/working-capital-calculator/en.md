---
node_id: "working-capital-calculator"
title: "Working Capital Calculator"
description: "Calculate working capital, liquidity ratios (current and quick ratios), and operational cash conversion cycle metrics."
category: "Business & Commerce"
subcategory: "Financial Analysis & Calculators"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - finance
  - accounting
  - working-capital
  - liquidity
  - current-ratio
  - quick-ratio
  - cash-conversion-cycle
  - dso
  - dio
  - dpo
  - balance-sheet
  - business-metrics
related_nodes:
  - function
  - webhook
  - slack
  - discord-bot-send
  - filter-array
  - log
---

<!-- SECTION: header -->
# Working Capital Calculator

> **Category:** Business & Commerce | **Subcategory:** Financial Analysis & Calculators | **Type:** Action Node

Calculate essential corporate liquidity metrics, working capital (fonds de roulement), acid-test ratios, and cash conversion cycle (CCC) days to evaluate financial health and operational efficiency.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Working Capital Calculator** node enables financial analysts, accounting teams, ERP integrators, and business workflow automators to instantly compute key working capital metrics and evaluate short-term liquidity risk.

By providing balance sheet and income statement figures (such as current assets, current liabilities, inventory, accounts receivable, accounts payable, annual sales, and Cost of Goods Sold), this node calculates:
- **Net Working Capital:** The liquid surplus available to fund day-to-day operations (`Current Assets - Current Liabilities`).
- **Current Ratio:** A benchmark measure of short-term debt coverage (`Current Assets / Current Liabilities`).
- **Quick Ratio (Acid-Test):** A stringent solvency metric excluding less-liquid inventory (`(Current Assets - Inventory) / Current Liabilities`).
- **Operational Cycle Metrics:** Days Sales Outstanding (DSO), Days Inventory Outstanding (DIO), Days Payables Outstanding (DPO), and the complete **Cash Conversion Cycle (CCC)**.
- **Automated Health Interpretations:** Contextual assessments classifying ratios into *"Healthy"*, *"Acceptable"*, or *"Concerning"*.

### Key Features

- **Comprehensive Financial Formulas:** Integrates standard balance sheet ratios and cash cycle equations in a single node.
- **Graceful Partial Calculation:** Automatically computes core working capital and liquidity ratios even if operational turnover figures (sales, COGS) are omitted.
- **Automated Qualitative Interpretations:** Instantly tags liquidity metrics with health indicators based on established financial industry thresholds.
- **Zero External Latency:** Pure deterministic calculation with instant local execution and no external API dependencies or rate limits.
- **Dynamic Expression Support:** Map inputs directly from upstream ERP webhooks, Google Sheets, databases, or accounting systems.

### Common Use Cases

- **Credit Risk & Loan Underwriting:** Automatically analyze prospective borrower balance sheets and flag high-risk liquidity profiles.
- **Automated Financial Reporting:** Pull monthly trial balance figures from accounting systems (e.g. QuickBooks, Xero, NetSuite) and post liquidity dashboards to Slack or executive channels.
- **Supply Chain & Cash Flow Optimization:** Track inventory turnover (DIO) and supplier credit periods (DPO) to identify trapped cash in the operating cycle.
- **E-Commerce Health Monitoring:** Calculate real-time Cash Conversion Cycles for fast-moving retail businesses.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure the Working Capital Calculator by specifying your organization's current assets, liabilities, and optional operational turnover metrics.

![Working Capital Calculator Configuration](icon.svg)

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|:-------:|-------------|
| `currentAssets` | `number` | ✅ Yes | `50000` | Total liquid and short-term assets (cash, marketable securities, receivables, inventory). Must be `>= 0`. |
| `currentLiabilities` | `number` | ✅ Yes | `30000` | Total short-term obligations and debts due within one year (payables, short-term debt). Must be `>= 0`. |
| `inventory` | `number` | ❌ No | `0` | Total value of raw materials, work-in-progress, and finished goods inventory. Must be `>= 0`. |
| `receivables` | `number` | ❌ No | `0` | Accounts receivable owed by customers for credit sales. Must be `>= 0`. |
| `payables` | `number` | ❌ No | `0` | Accounts payable owed to suppliers and vendors. Must be `>= 0`. |
| `sales` | `number` | ❌ No | `0` | Annual or period total net sales revenue. Must be `>= 0`. |
| `cogs` | `number` | ❌ No | `0` | Annual or period total Cost of Goods Sold. Must be `>= 0`. |

---

### Detailed Parameter Descriptions

#### `currentAssets` (Required)
The sum of all assets expected to be converted into cash within one year.
- **Formula Impact:** Directly increases Working Capital, Current Ratio, and Quick Ratio.
- **Constraints:** Must be a non-negative number (`>= 0`).

#### `currentLiabilities` (Required)
The sum of all debts and financial obligations due within one year.
- **Formula Impact:** Directly decreases Working Capital and serves as the denominator for Current and Quick ratios.
- **Constraints:** Must be a non-negative number (`>= 0`).

#### `inventory` (Optional)
The carrying value of stock and goods held for resale.
- **Formula Impact:** Subtracted from Current Assets to determine Quick Assets. When provided alongside `cogs`, used to calculate Days Inventory Outstanding (DIO).

#### `receivables` (Optional)
Credit extended to customers awaiting settlement.
- **Formula Impact:** When provided alongside `sales`, used to calculate Days Sales Outstanding (DSO).

#### `payables` (Optional)
Short-term credit extended by suppliers for purchased goods.
- **Formula Impact:** When provided alongside `cogs`, used to calculate Days Payables Outstanding (DPO).

#### `sales` (Optional)
Total top-line net revenue over the financial period (annualized).

#### `cogs` (Optional)
Total direct cost of producing goods or delivering services over the financial period (annualized).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow trigger or financial data payload. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted upon calculation. Contains working capital value, financial ratios, cycle days, and qualitative interpretations. |
| `error` | `Error` | Emitted if calculation fails or unexpected input types are supplied. |

---

### Output Data Structure Example

```json
{
  "success": true,
  "workingCapital": 60000,
  "currentRatio": 2.5,
  "quickRatio": 2.0,
  "dso": 45.625,
  "dio": 91.25,
  "dpo": 45.625,
  "cashConversionCycle": 91.25,
  "interpretation": {
    "currentRatio": "Healthy",
    "quickRatio": "Healthy",
    "ccc": "91 days to convert investments back to cash"
  }
}
```

---

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether the financial calculation completed successfully (`true`). |
| `workingCapital` | `number` | Net Working Capital in currency units (`currentAssets - currentLiabilities`). |
| `currentRatio` | `number` | General liquidity ratio (`currentAssets / currentLiabilities`). |
| `quickRatio` | `number` | Acid-test liquidity ratio (`(currentAssets - inventory) / currentLiabilities`). |
| `dso` | `number | null` | Days Sales Outstanding in days (`(receivables / sales) * 365`). `null` if sales or receivables are omitted. |
| `dio` | `number | null` | Days Inventory Outstanding in days (`(inventory / cogs) * 365`). `null` if COGS or inventory are omitted. |
| `dpo` | `number | null` | Days Payables Outstanding in days (`(payables / cogs) * 365`). `null` if COGS or payables are omitted. |
| `cashConversionCycle` | `number | null` | Net Cash Conversion Cycle in days (`dso + dio - dpo`). `null` if any component is missing. |
| `interpretation.currentRatio` | `string` | Qualitative assessment: `"Healthy"` (>= 2.0), `"Acceptable"` (>= 1.0), or `"Concerning"` (< 1.0). |
| `interpretation.quickRatio` | `string` | Qualitative assessment: `"Healthy"` (>= 1.0) or `"Concerning"` (< 1.0). |
| `interpretation.ccc` | `string` | Formatted summary of CCC turnaround time or `"Insufficient data"`. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Standard Liquidity Assessment (Healthy Baseline)

Evaluate a business with $50,000 in current assets and $30,000 in current liabilities:

**Configuration:**
- **CurrentAssets:** `50000`
- **CurrentLiabilities:** `30000`
- **Inventory:** `0`

**Output:**
```json
{
  "success": true,
  "workingCapital": 20000,
  "currentRatio": 1.6666666666666667,
  "quickRatio": 1.6666666666666667,
  "dso": null,
  "dio": null,
  "dpo": null,
  "cashConversionCycle": null,
  "interpretation": {
    "currentRatio": "Acceptable",
    "quickRatio": "Healthy",
    "ccc": "Insufficient data"
  }
}
```

---

### Example 2: Full Operating Cycle & Cash Conversion Cycle

Analyze complete financial statements including inventory turnover and credit terms:

**Configuration:**
- **CurrentAssets:** `100000`
- **CurrentLiabilities:** `40000`
- **Inventory:** `20000`
- **Receivables:** `15000`
- **Payables:** `10000`
- **Sales:** `120000`
- **Cogs:** `80000`

**Output:**
```json
{
  "success": true,
  "workingCapital": 60000,
  "currentRatio": 2.5,
  "quickRatio": 2,
  "dso": 45.625,
  "dio": 91.25,
  "dpo": 45.625,
  "cashConversionCycle": 91.25,
  "interpretation": {
    "currentRatio": "Healthy",
    "quickRatio": "Healthy",
    "ccc": "91 days to convert investments back to cash"
  }
}
```

---

### Example 3: Negative Working Capital & Distressed Liquidity

Detect an under-capitalized company facing short-term solvency risk:

**Configuration:**
- **CurrentAssets:** `20000`
- **CurrentLiabilities:** `50000`
- **Inventory:** `15000`

**Output:**
```json
{
  "success": true,
  "workingCapital": -30000,
  "currentRatio": 0.4,
  "quickRatio": 0.1,
  "interpretation": {
    "currentRatio": "Concerning",
    "quickRatio": "Concerning",
    "ccc": "Insufficient data"
  }
}
```

---

### Example 4: Negative Cash Conversion Cycle (E-Commerce Model)

Model a business with rapid collections and extended supplier payment terms:

**Configuration:**
- **CurrentAssets:** `200000`
- **CurrentLiabilities:** `150000`
- **Inventory:** `10000`
- **Receivables:** `5000`
- **Payables:** `60000`
- **Sales:** `300000`
- **Cogs:** `200000`

**Output:**
```json
{
  "success": true,
  "workingCapital": 50000,
  "currentRatio": 1.3333333333333333,
  "quickRatio": 1.2666666666666666,
  "dso": 6.083333333333333,
  "dio": 18.25,
  "dpo": 109.5,
  "cashConversionCycle": -85.16666666666667,
  "interpretation": {
    "currentRatio": "Acceptable",
    "quickRatio": "Healthy",
    "ccc": "-85 days to convert investments back to cash"
  }
}
```

---

### Example 5: Heavy Inventory Divergence Analysis

Analyze a firm whose Current Ratio appears healthy, but Quick Ratio reveals illiquid stock concentration:

**Configuration:**
- **CurrentAssets:** `100000`
- **CurrentLiabilities:** `50000`
- **Inventory:** `80000`

**Output:**
```json
{
  "success": true,
  "workingCapital": 50000,
  "currentRatio": 2,
  "quickRatio": 0.4,
  "interpretation": {
    "currentRatio": "Healthy",
    "quickRatio": "Concerning",
    "ccc": "Insufficient data"
  }
}
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Calculate working capital, liquidity ratios, and cash conversion cycle
```

### Common Architecture Patterns

- **Automated Solvency Alert Pipeline:** Accounting Webhook (Month-end close) → Working Capital Calculator → Filter Array (Condition: `currentRatio < 1.0`) → Slack Send (Alert CFO: *"Solvency alert: Working capital fell below threshold"*).
- **ERP Credit Underwriting Engine:** CRM Lead Form → Function (Extract Balance Sheet JSON) → Working Capital Calculator → Branch Node (Auto-approve credit terms if `currentRatio >= 2.0` and `quickRatio >= 1.0`).
- **Cash Flow Health Dashboard:** Schedule Trigger (Weekly) → Database Query (Extract current balances) → Working Capital Calculator → Discord Webhook (Post Executive Liquidity Digest).

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Number must be greater than or equal to 0`
- **Cause:** One of the numerical fields (e.g. `currentAssets`, `inventory`, `sales`) was supplied with a negative number.
- **Solution:** Balance sheet accounts and revenue inputs must be positive numbers or zero (`>= 0`). Ensure adjustments or contra-accounts are subtracted before supplying inputs.

#### `interpretation.ccc: "Insufficient data"`
- **Cause:** The Cash Conversion Cycle equation requires all three operational turnover figures: DSO (requires `sales` and `receivables`), DIO (requires `cogs` and `inventory`), and DPO (requires `cogs` and `payables`).
- **Solution:** Supply non-zero values for `sales`, `cogs`, `inventory`, `receivables`, and `payables` if you require Cash Conversion Cycle computation.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `inventory: Number must be greater than or equal to 0` | Negative value entered | Provide a non-negative number (`>= 0`) |
| `currentAssets: Number must be greater than or equal to 0` | Negative value entered | Provide a non-negative number (`>= 0`) |
| `currentLiabilities: Number must be greater than or equal to 0` | Negative value entered | Provide a non-negative number (`>= 0`) |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Function](../function/en.md) — Execute custom financial transformation logic
- [Slack](../slack/en.md) — Broadcast automated financial health summaries to Slack channels
- [Webhook](../webhook/en.md) — Ingest trial balance data from external ERP and billing services
- [Filter Array](../filter-array/en.md) — Filter and route workflows based on ratio thresholds

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial release of Working Capital Calculator Action Node |

<!-- /SECTION: changelog -->
