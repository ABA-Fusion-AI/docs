---
node_id: "financial-ratio-analyzer"
title: "Financial Ratio Analyzer"
description: "Analyze liquidity, profitability, leverage, and efficiency financial ratios."
category: "Finance"
version: "1.0.0"
language: "en"
last_updated: "2026-08-11"
author: "Fusion Team"
tags:

- finance
- financial-analysis
- financial-ratios
- liquidity
- profitability
- leverage
- efficiency
- accounting
- business
- analysis
- action

related_nodes:
- function
- if
- calculator

---

# Financial Ratio Analyzer

> **Category:** finance-nodes | **Type:** Action Node

Analyze financial statements using common **liquidity, profitability, leverage, and efficiency ratios**.

The **Financial Ratio Analyzer** node takes financial values such as current assets, liabilities, equity, revenue, net income, and cost of goods sold and calculates a set of commonly used financial ratios.

The node returns the calculated ratios as structured workflow data grouped into four categories:

- Liquidity
- Profitability
- Leverage
- Efficiency

The node performs calculations directly from the configured financial values and does not require an external API.

### Supported Features

- Calculate current ratio
- Calculate working capital
- Calculate return on assets (ROA)
- Calculate return on equity (ROE)
- Calculate net profit margin
- Calculate operating margin
- Calculate debt-to-equity ratio
- Calculate debt-to-assets ratio
- Calculate equity multiplier
- Calculate asset turnover
- Calculate equity turnover
- Automatically calculate operating income from revenue and COGS
- Accept explicit operating income
- Validate non-negative financial input values where configured
- Return structured financial analysis results
- Handle calculation errors

### Use Cases

- Analyze company financial performance
- Build financial dashboards
- Evaluate liquidity
- Analyze profitability
- Analyze financial leverage
- Measure asset efficiency
- Compare financial performance between periods
- Build accounting workflows
- Automate financial ratio calculations
- Prepare financial analysis reports
- Feed financial ratios into downstream workflow nodes
- Filter financial results using an `If` node
- Transform financial results using a `Function` node

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| ---------- | ---- | -------- | ------- | ----------- |
| `currentAssets` | `number` | ❌ No | `50000` | Total current assets. Must be greater than or equal to `0`. |
| `currentLiabilities` | `number` | ❌ No | `30000` | Total current liabilities. Must be greater than or equal to `0`. |
| `totalAssets` | `number` | ❌ No | `200000` | Total assets. Must be greater than or equal to `0`. |
| `totalLiabilities` | `number` | ❌ No | `120000` | Total liabilities. Must be greater than or equal to `0`. |
| `equity` | `number` | ❌ No | `80000` | Total shareholders' equity. Must be greater than or equal to `0`. |
| `revenue` | `number` | ❌ No | `100000` | Total revenue. Must be greater than or equal to `0`. |
| `netIncome` | `number` | ❌ No | `15000` | Net income used for profitability calculations. |
| `cogs` | `number` | ❌ No | `0` | Cost of goods sold. Must be greater than or equal to `0`. |
| `operatingIncome` | `number` | ❌ No | `0` | Operating income. When falsy, operating income is calculated as revenue minus COGS. |

### Configuration Schema

```typescript
const schema: SchemaTypeAny = v.object({
  currentAssets: v.number().min(0).default(50000),
  currentLiabilities: v.number().min(0).default(30000),
  totalAssets: v.number().min(0).default(200000),
  totalLiabilities: v.number().min(0).default(120000),
  equity: v.number().min(0).default(80000),
  revenue: v.number().min(0).default(100000),
  netIncome: v.number().default(15000),
  cogs: v.number().min(0).optional().default(0),
  operatingIncome: v.number().optional().default(0),
});
```

---

## Inputs & Outputs

### Inputs

The node does not use workflow input data.

The `handleTick()` method ignores the incoming workflow data and performs all calculations using the configured financial parameters.

All values must therefore be supplied through the node configuration.

### Outputs

The node returns a structured financial analysis result.

| Output | Type | Description |
| ------ | ---- | ----------- |
| `success` | `boolean` | Indicates whether the calculations completed successfully. |
| `liquidity` | `object` | Contains liquidity ratios and working capital. |
| `profitability` | `object` | Contains profitability ratios. |
| `leverage` | `object` | Contains leverage ratios. |
| `efficiency` | `object` | Contains efficiency ratios. |
| `error` | `string` | Error message returned when calculation fails. Only present when `success` is `false`. |

### Liquidity Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `currentRatio` | `number` | Current assets divided by current liabilities. |
| `workingCapital` | `number` | Current assets minus current liabilities. |

### Profitability Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `roa` | `number` | Return on Assets, expressed as a percentage. |
| `roe` | `number` | Return on Equity, expressed as a percentage. |
| `netMargin` | `number` | Net income as a percentage of revenue. |
| `operatingMargin` | `number` | Operating income as a percentage of revenue. |

### Leverage Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `debtToEquity` | `number` | Total liabilities divided by equity. |
| `debtToAssets` | `number` | Total liabilities divided by total assets. |
| `equityMultiplier` | `number` | Total assets divided by equity. |

### Efficiency Fields

| Field | Type | Description |
| ----- | ---- | ----------- |
| `assetTurnover` | `number` | Revenue divided by total assets. |
| `equityTurnover` | `number` | Revenue divided by equity. |

---

## Output Example

Using the default configuration:

```json
{
  "success": true,
  "liquidity": {
    "currentRatio": 1.6666666666666667,
    "workingCapital": 20000
  },
  "profitability": {
    "roa": 7.5,
    "roe": 18.75,
    "netMargin": 15,
    "operatingMargin": 15
  },
  "leverage": {
    "debtToEquity": 1.5,
    "debtToAssets": 0.6,
    "equityMultiplier": 2.5
  },
  "efficiency": {
    "assetTurnover": 0.5,
    "equityTurnover": 1.25
  }
}
```

---

## Configuration Examples

### Default Configuration

Uses the default financial values:

```json
{
  "currentAssets": 50000,
  "currentLiabilities": 30000,
  "totalAssets": 200000,
  "totalLiabilities": 120000,
  "equity": 80000,
  "revenue": 100000,
  "netIncome": 15000,
  "cogs": 0,
  "operatingIncome": 0
}
```

### Example Financial Data

```json
{
  "currentAssets": 75000,
  "currentLiabilities": 50000,
  "totalAssets": 300000,
  "totalLiabilities": 180000,
  "equity": 120000,
  "revenue": 250000,
  "netIncome": 30000,
  "cogs": 140000,
  "operatingIncome": 50000
}
```

### Calculate Operating Income from COGS

If `operatingIncome` is not provided or has a falsy value, the node calculates:

```text
Operating Income = Revenue - COGS
```

Example:

```json
{
  "revenue": 200000,
  "cogs": 120000,
  "operatingIncome": 0
}
```

The node calculates:

```text
Operating Income = 200000 - 120000
Operating Income = 80000
```

### Explicit Operating Income

An explicit operating income can also be supplied:

```json
{
  "revenue": 200000,
  "cogs": 120000,
  "operatingIncome": 60000
}
```

When `operatingIncome` is truthy, the configured value is used directly.

---

## Workflow Integration

### Sample Workflow: Financial Analysis

```json
{
  "nodes": [
    {
      "id": "financial-ratio-analyzer",
      "type": "financial-ratio-analyzer",
      "config": {
        "currentAssets": 50000,
        "currentLiabilities": 30000,
        "totalAssets": 200000,
        "totalLiabilities": 120000,
        "equity": 80000,
        "revenue": 100000,
        "netIncome": 15000,
        "cogs": 0,
        "operatingIncome": 0
      }
    }
  ]
}
```

### Sample Workflow: Financial Data → Analyzer

A workflow can collect or calculate financial values in previous nodes and then pass them through configuration to the analyzer.

```json
{
  "nodes": [
    {
      "id": "financial-data",
      "type": "input"
    },
    {
      "id": "financial-ratio-analyzer",
      "type": "financial-ratio-analyzer",
      "config": {
        "currentAssets": 75000,
        "currentLiabilities": 50000,
        "totalAssets": 300000,
        "totalLiabilities": 180000,
        "equity": 120000,
        "revenue": 250000,
        "netIncome": 30000,
        "cogs": 140000,
        "operatingIncome": 50000
      }
    }
  ]
}
```

### Sample Workflow: Analyzer → Function

```json
{
  "nodes": [
    {
      "id": "financial-ratio-analyzer",
      "type": "financial-ratio-analyzer",
      "config": {
        "currentAssets": 50000,
        "currentLiabilities": 30000,
        "totalAssets": 200000,
        "totalLiabilities": 120000,
        "equity": 80000,
        "revenue": 100000,
        "netIncome": 15000,
        "cogs": 10000,
        "operatingIncome": 0
      }
    },
    {
      "id": "process-ratios",
      "type": "function"
    }
  ]
}
```

The `Function` node can transform the ratio results into a custom report or dashboard structure.

### Sample Workflow: Analyzer → If

```json
{
  "nodes": [
    {
      "id": "financial-ratio-analyzer",
      "type": "financial-ratio-analyzer",
      "config": {
        "currentAssets": 50000,
        "currentLiabilities": 30000,
        "totalAssets": 200000,
        "totalLiabilities": 120000,
        "equity": 80000,
        "revenue": 100000,
        "netIncome": 15000,
        "cogs": 10000,
        "operatingIncome": 20000
      }
    },
    {
      "id": "check-liquidity",
      "type": "if"
    }
  ]
}
```

The `If` node can be used to route the workflow based on calculated financial ratios.

### Common Patterns

- Financial Data → Financial Ratio Analyzer → Function
- Financial Ratio Analyzer → If → Notification
- Financial Ratio Analyzer → Function → Report
- Financial Ratio Analyzer → Database → Financial History
- Financial Ratio Analyzer → If → Financial Alert
- Financial Ratio Analyzer → Function → Dashboard
- Financial Ratio Analyzer → Calculator → Additional Analysis

---

## Ratio Calculations

The node calculates the following ratios.

### Current Ratio

The current ratio measures the relationship between current assets and current liabilities.

Formula:

```text
Current Ratio = Current Assets / Current Liabilities
```

Implementation:

```typescript
currentAssets / currentLiabilities
```

Example:

```text
Current Assets = 50,000
Current Liabilities = 30,000

Current Ratio = 50,000 / 30,000
              = 1.6667
```

---

### Working Capital

Formula:

```text
Working Capital = Current Assets - Current Liabilities
```

Implementation:

```typescript
currentAssets - currentLiabilities
```

Example:

```text
50,000 - 30,000 = 20,000
```

---

### Return on Assets

ROA measures net income relative to total assets.

Formula:

```text
ROA = (Net Income / Total Assets) × 100
```

Implementation:

```typescript
(netIncome / totalAssets) * 100
```

Example:

```text
Net Income = 15,000
Total Assets = 200,000

ROA = (15,000 / 200,000) × 100
    = 7.5%
```

---

### Return on Equity

ROE measures net income relative to shareholders' equity.

Formula:

```text
ROE = (Net Income / Equity) × 100
```

Implementation:

```typescript
(netIncome / equity) * 100
```

Example:

```text
Net Income = 15,000
Equity = 80,000

ROE = (15,000 / 80,000) × 100
    = 18.75%
```

---

### Net Profit Margin

Formula:

```text
Net Margin = (Net Income / Revenue) × 100
```

Implementation:

```typescript
(netIncome / revenue) * 100
```

Example:

```text
Net Income = 15,000
Revenue = 100,000

Net Margin = (15,000 / 100,000) × 100
           = 15%
```

---

### Operating Margin

Operating income is determined using:

```typescript
const opIncome = operatingIncome || revenue - (cogs || 0);
```

Therefore, when `operatingIncome` is falsy, the node calculates:

```text
Operating Income = Revenue - COGS
```

The operating margin formula is:

```text
Operating Margin = (Operating Income / Revenue) × 100
```

Implementation:

```typescript
(opIncome / revenue) * 100
```

---

### Debt-to-Equity Ratio

Formula:

```text
Debt-to-Equity = Total Liabilities / Equity
```

Implementation:

```typescript
totalLiabilities / equity
```

Example:

```text
Total Liabilities = 120,000
Equity = 80,000

Debt-to-Equity = 120,000 / 80,000
               = 1.5
```

---

### Debt-to-Assets Ratio

Formula:

```text
Debt-to-Assets = Total Liabilities / Total Assets
```

Implementation:

```typescript
totalLiabilities / totalAssets
```

Example:

```text
Total Liabilities = 120,000
Total Assets = 200,000

Debt-to-Assets = 120,000 / 200,000
               = 0.6
```

---

### Equity Multiplier

Formula:

```text
Equity Multiplier = Total Assets / Equity
```

Implementation:

```typescript
totalAssets / equity
```

Example:

```text
Total Assets = 200,000
Equity = 80,000

Equity Multiplier = 200,000 / 80,000
                  = 2.5
```

---

### Asset Turnover

Formula:

```text
Asset Turnover = Revenue / Total Assets
```

Implementation:

```typescript
revenue / totalAssets
```

Example:

```text
Revenue = 100,000
Total Assets = 200,000

Asset Turnover = 100,000 / 200,000
               = 0.5
```

---

### Equity Turnover

Formula:

```text
Equity Turnover = Revenue / Equity
```

Implementation:

```typescript
revenue / equity
```

Example:

```text
Revenue = 100,000
Equity = 80,000

Equity Turnover = 100,000 / 80,000
                = 1.25
```

---

## Liquidity Analysis

The `liquidity` output contains:

```json
{
  "currentRatio": 1.6666666666666667,
  "workingCapital": 20000
}
```

Liquidity ratios can be used to analyze a company's ability to meet short-term obligations.

The node does not assign a financial health rating or classify a ratio as good or bad. It only performs the calculations.

---

## Profitability Analysis

The `profitability` output contains:

```json
{
  "roa": 7.5,
  "roe": 18.75,
  "netMargin": 15,
  "operatingMargin": 15
}
```

The profitability calculations are expressed as percentages for ROA, ROE, net margin, and operating margin.

---

## Leverage Analysis

The `leverage` output contains:

```json
{
  "debtToEquity": 1.5,
  "debtToAssets": 0.6,
  "equityMultiplier": 2.5
}
```

These values describe the relationship between liabilities, assets, and equity.

The node does not interpret the leverage level as high, low, safe, or risky.

---

## Efficiency Analysis

The `efficiency` output contains:

```json
{
  "assetTurnover": 0.5,
  "equityTurnover": 1.25
}
```

These ratios relate revenue to assets and equity.

---

## Input Validation

The following parameters use a minimum value of `0`:

- `currentAssets`
- `currentLiabilities`
- `totalAssets`
- `totalLiabilities`
- `equity`
- `revenue`
- `cogs`

For example:

```typescript
currentAssets: v.number().min(0).default(50000)
```

Negative values for these parameters are rejected by the validation schema.

The following parameters do not have a minimum constraint:

- `netIncome`
- `operatingIncome`

Therefore, negative values can be supplied for these fields.

---

## Operating Income

The node supports two ways of determining operating income.

### Explicit Operating Income

If `operatingIncome` is truthy, the configured value is used:

```typescript
const opIncome = operatingIncome || revenue - (cogs || 0);
```

Example:

```json
{
  "revenue": 100000,
  "cogs": 40000,
  "operatingIncome": 25000
}
```

The node uses:

```text
Operating Income = 25,000
```

### Calculated Operating Income

If `operatingIncome` is falsy, the node calculates:

```text
Operating Income = Revenue - COGS
```

Example:

```json
{
  "revenue": 100000,
  "cogs": 40000,
  "operatingIncome": 0
}
```

The node calculates:

```text
Operating Income = 100,000 - 40,000
                 = 60,000
```

### Important Behavior

Because the implementation uses JavaScript's `||` operator, an `operatingIncome` value of `0` is treated as falsy and causes the node to calculate operating income from revenue and COGS.

---

## Error Handling

The calculation logic is wrapped in a `try/catch` block.

If an error occurs, the node returns:

```json
{
  "success": false,
  "error": "<error message>"
}
```

A successful calculation returns:

```json
{
  "success": true,
  "liquidity": {},
  "profitability": {},
  "leverage": {},
  "efficiency": {}
}
```

---

## Troubleshooting

### Invalid Financial Value

**Cause**

A value configured with `.min(0)` is negative.

**Solution**

Use a value greater than or equal to `0` for:

- `currentAssets`
- `currentLiabilities`
- `totalAssets`
- `totalLiabilities`
- `equity`
- `revenue`
- `cogs`

---

### Division by Zero

**Cause**

Some ratios divide by financial values that can be configured as `0`.

Examples include:

```text
Current Assets / Current Liabilities
Net Income / Total Assets
Net Income / Equity
Net Income / Revenue
Operating Income / Revenue
Total Liabilities / Equity
Total Liabilities / Total Assets
Total Assets / Equity
Revenue / Total Assets
Revenue / Equity
```

The schema permits zero for several denominator fields.

JavaScript numeric division by zero can produce `Infinity` or `NaN` rather than throwing a standard exception.

**Solution**

Provide non-zero denominator values when meaningful financial ratios are required.

---

### Unexpected Operating Margin

**Cause**

`operatingIncome` is set to `0`.

**Behavior**

Because the implementation uses:

```typescript
operatingIncome || revenue - (cogs || 0)
```

a value of `0` is treated as falsy.

The node therefore calculates operating income from:

```text
Revenue - COGS
```

**Solution**

Provide a non-zero `operatingIncome` if an explicit operating income value is intended.

---

### Calculation Error

**Cause**

An unexpected runtime error occurred during the calculations.

**Solution**

Check the returned `error` field:

```json
{
  "success": false,
  "error": "..."
}
```

---

## Notes

The node performs mathematical calculations only.

It does not:

- Retrieve financial data from an external API
- Connect to a financial database
- Interpret financial ratios
- Provide investment advice
- Assign financial health scores
- Compare ratios against industry benchmarks
- Generate financial forecasts

The node returns calculated values and leaves interpretation to downstream workflow nodes.

---

## Related

- [Function](./function.md) – Transform and process financial analysis results
- [If](./if.md) – Filter and route workflows based on calculated ratios
- [Calculator](./calculator.md) – Perform additional calculations

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-11 | Initial release |
