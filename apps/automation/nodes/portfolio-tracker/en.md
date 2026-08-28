---
node_id: "portfolio-tracker"
title: "Portfolio Tracker"
description: "Analyze investment portfolio performance, returns, and allocation"
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-28"
author: "Fusion Team"
tags:
  - portfolio
  - finance
  - investment
  - stocks
  - crypto
  - roi
  - allocation
  - returns
related_nodes:
  - alpha-vantage
  - financial-modeling-prep
  - awesome-currency
  - currency-converter
  - function
  - log
---

<!-- SECTION: header -->
# Portfolio Tracker

> **Category:** Business & Commerce | **Subcategory:** Finance & Accounting | **Type:** Action Node

Analyze investment portfolios across stocks, ETFs, bonds, and cryptocurrencies to calculate total valuations, capital gains, dividend yields, asset allocation weights, and net return percentages.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Portfolio Tracker** node provides comprehensive financial analytics for multi-asset investment holdings. It evaluates cost basis, current market valuation, unrealized profit/loss, dividend cashflow, and portfolio allocation weights for each holding as well as aggregate portfolio totals.

The node supports fractional shares and crypto decimals, accepts dynamic price expressions from upstream market data nodes, and enforces strict non-negative number validation to ensure accurate financial calculations.

### Key Features

- **Comprehensive Performance Metrics:** Computes total invested capital, total market value, capital gains, cumulative dividends, and overall ROI percentage.
- **Asset Allocation Weighting:** Automatically calculates each asset's percentage share (`weight`) relative to the total portfolio value.
- **Dynamic & Flexible Inputs:** Accepts numbers or numeric string expressions, making it seamless to bind live prices from nodes like Alpha Vantage, Binance, or CoinGecko.
- **Fractional & Crypto Support:** Fully handles decimal quantities (e.g. `0.5 BTC` or `12.345` shares).
- **Total Return with Dividends:** Integrates dividend and staking income into the net return calculation (`totalReturn = totalValue - totalInvested + totalDividends`).

### Use Cases

- **Automated Portfolio Reports:** Generate periodic daily or weekly portfolio performance summaries and dispatch them via email or Slack.
- **Asset Allocation & Rebalancing Alerts:** Trigger alerts when an individual asset's weight deviates from target allocation boundaries (e.g., tech stocks exceed 40%).
- **Multi-Asset Performance Monitoring:** Combine stocks, crypto, commodities, and fiat holdings into a unified valuation dashboard.
- **Profit-Taking & Stop-Loss Automations:** Evaluate portfolio gains dynamically to trigger rebalancing or order execution workflows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `holdings` | `array` | ✅ Yes | — | Array of investment holding objects. Must contain at least 1 holding. |

---

### Holding Item Properties

Each object in the `holdings` array supports the following properties:

| Property | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `symbol` | `string` | ✅ Yes | — | Ticker or asset symbol identifier (e.g., `AAPL`, `MSFT`, `NVDA`, `BTC`). |
| `shares` | `number \| string` | ✅ Yes | — | Number of shares or units owned (`>= 0`). Supports decimals. |
| `buyPrice` | `number \| string` | ✅ Yes | — | Average purchase price (cost basis) per share/unit (`>= 0`). |
| `currentPrice` | `number \| string` | ✅ Yes | — | Current market price per share/unit (`>= 0`). |
| `dividends` | `number \| string` | ❌ No | `0` | Cumulative dividend or staking income earned for this position (`>= 0`). |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow data. Can be referenced dynamically in holding parameters using expressions. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when calculation succeeds. Contains aggregate portfolio metrics and detailed per-holding performance breakdown. |
| `error` | `object` | Emitted if validation fails (e.g. negative numbers or empty holdings array). |

---

### Output Data Structure

```json
{
  "success": true,
  "totalInvested": 3300,
  "totalValue": 4100,
  "totalDividends": 30,
  "totalReturn": 830,
  "returnPercent": 25.15,
  "holdings": [
    {
      "symbol": "AAPL",
      "shares": 10,
      "invested": 1500,
      "currentValue": 1800,
      "gain": 300,
      "gainPercent": 20.0,
      "dividends": 20,
      "weight": 43.9
    },
    {
      "symbol": "MSFT",
      "shares": 5,
      "invested": 1000,
      "currentValue": 1300,
      "gain": 300,
      "gainPercent": 30.0,
      "dividends": 0,
      "weight": 31.71
    },
    {
      "symbol": "NVDA",
      "shares": 8,
      "invested": 800,
      "currentValue": 1000,
      "gain": 200,
      "gainPercent": 25.0,
      "dividends": 10,
      "weight": 24.39
    }
  ]
}
```

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | `true` if calculations succeeded |
| `totalInvested` | `number` | Sum of all cost bases (`shares * buyPrice`) across all holdings |
| `totalValue` | `number` | Total current market value of all positions (`shares * currentPrice`) |
| `totalDividends` | `number` | Cumulative dividend income received across all positions |
| `totalReturn` | `number` | Total net profit (`totalValue - totalInvested + totalDividends`) |
| `returnPercent` | `number` | Total return on investment percentage (`(totalReturn / totalInvested) * 100`) |
| `holdings[].symbol` | `string` | Asset ticker symbol |
| `holdings[].shares` | `number` | Quantity of shares / units |
| `holdings[].invested` | `number` | Cost basis for this specific position |
| `holdings[].currentValue` | `number` | Market valuation for this position |
| `holdings[].gain` | `number` | Unrealized capital gain/loss (`currentValue - invested`) |
| `holdings[].gainPercent` | `number` | Capital gain percentage for this position |
| `holdings[].dividends` | `number` | Dividends credited for this position |
| `holdings[].weight` | `number` | Percentage allocation of this asset within the total portfolio (`(currentValue / totalValue) * 100`) |

---

### Formula Reference

| Metric | Formula |
|--------|---------|
| **Position Cost Basis** | `invested = shares * buyPrice` |
| **Position Market Value** | `currentValue = shares * currentPrice` |
| **Position Capital Gain** | `gain = currentValue - invested` |
| **Position Gain %** | `gainPercent = (gain / invested) * 100` |
| **Position Weight %** | `weight = (currentValue / totalValue) * 100` |
| **Total Net Return** | `totalReturn = totalValue - totalInvested + totalDividends` |
| **Total Portfolio ROI %** | `returnPercent = (totalReturn / totalInvested) * 100` |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Static Multi-Asset Portfolio Tracking

Analyze a three-stock investment portfolio with dividend income.

**Parameter Configuration:**

```json
{
  "holdings": [
    {
      "symbol": "AAPL",
      "shares": 10,
      "buyPrice": 150,
      "currentPrice": 180,
      "dividends": 20
    },
    {
      "symbol": "MSFT",
      "shares": 5,
      "buyPrice": 200,
      "currentPrice": 260,
      "dividends": 0
    }
  ]
}
```

---

### Example 2: Dynamic Live Price Binding via Market Data Node

Fetch real-time stock prices using Alpha Vantage or CoinGecko, and pass them into the Portfolio Tracker node.

**Workflow Pattern:**

```text
Cron Trigger (Daily 09:00)
  → Alpha Vantage / Crypto Node (Fetch current prices)
  → Function (Format holdings with live prices)
  → Portfolio Tracker (holdings: {{Function.output}})
  → Log / Slack Notification
```

---

### Example 3: Portfolio Rebalancing Alert

Detect if any individual holding exceeds a maximum allocation threshold (e.g. 50%).

**Workflow Pattern:**

```text
Schedule Trigger
  → Portfolio Tracker
  → Function (Find holdings where weight > 50)
  → If (has overweight assets)
      → Email Send ("Portfolio rebalancing required")
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Analyze investment portfolio performance and allocation
```

### Common Patterns

- **Daily Valuation Digest:** Schedule Trigger → Portfolio Tracker → Format Markdown Summary → Send Email / Discord.
- **Crypto & Stock Hybrid Tracker:** CoinGecko + Alpha Vantage → Combine in Function → Portfolio Tracker → Google Sheets.
- **Tax & Capital Gain Reporter:** Portfolio Tracker → Filter Array (gain > 0) → Export CSV.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `Holdings array is required and must not be empty`

**Cause:** The `holdings` parameter was empty or not configured.

**Solution:** Click **+ Add Item** in the node parameter editor and add at least one holding with valid values.

#### Error: `Invalid <field>: must be a non-negative finite number`

**Cause:** One of `shares`, `buyPrice`, `currentPrice`, or `dividends` was configured with a negative number, `NaN`, or an invalid non-numeric string.

**Solution:** Ensure all numeric values are positive or zero (`>= 0`) numbers.

#### Error: `Invalid <field>: value cannot be blank`

**Cause:** A string field was passed with empty whitespace (e.g. `""` or `" "`).

**Solution:** Provide an actual number or a non-empty numeric string expression.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Holdings array is required and must not be empty` | Missing holdings array | Add at least 1 holding in configuration |
| `Invalid shares: must be a non-negative finite number` | Negative or invalid shares value | Set `shares >= 0` |
| `Invalid buyPrice: must be a non-negative finite number` | Negative or invalid buyPrice | Set `buyPrice >= 0` |
| `Invalid currentPrice: must be a non-negative finite number` | Negative or invalid currentPrice | Set `currentPrice >= 0` |
| `Invalid <field>: value cannot be blank` | Empty string provided | Provide a valid numeric value |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Alpha Vantage](../alpha-vantage/en.md) — Retrieve real-time stock and forex quotes
- [Financial Modeling Prep](../financial-modeling-prep/en.md) — Access institutional financial metrics and market data
- [Currency Converter](../currency-converter/en.md) — Convert portfolio values across international currencies
- [Function](../function/en.md) — Transform market data and calculate custom financial KPIs
- [Log](../log/en.md) — Output portfolio analysis results in console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-28 | Initial release |

<!-- /SECTION: changelog -->
