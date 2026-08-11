---
node_id: "binance-price"
title: "Binance Price"
description: "Get real-time cryptocurrency trading pair prices from the Binance API."
category: "peer-only"
subcategory: "crypto"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - binance
  - crypto
  - cryptocurrency
  - price
  - ticker
  - bitcoin
  - ethereum
  - usdt
related_nodes:
  - http-request
  - function
  - log
  - filter-array
---

<!-- SECTION: header -->
# Binance Price

> **Category:** Peer-Only Integrations | **Type:** Action Node

Fetch real-time cryptocurrency prices for any active trading pair directly from the public Binance API (e.g., `BTCUSDT`, `ETHUSDT`, `SOLUSDT`).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Binance Price** node queries Binance's public REST API (`/api/v3/ticker/price`) to retrieve the current market price for a specified cryptocurrency trading pair.

It accepts trading symbols (such as `BTCUSDT` or `ETHUSDT`), automatically normalizes case, and returns the current price formatted as a numerical value alongside ticker metadata.

### Key Features

- **Real-Time Market Data:** Connects directly to Binance API v3 for up-to-date ticker pricing.
- **Universal Pair Support:** Works with any valid Binance trading pair (`BTCUSDT`, `ETHUSDT`, `BNBUSDT`, `SOLUSDT`, `ETHBTC`, `BTCEUR`, etc.).
- **No API Key Required:** Queries public Binance endpoints without needing authentication credentials or API secrets.
- **Flexible Parameter Mapping:** Accepts symbol configuration directly in node parameters or dynamically from preceding workflow node output.
- **Numeric Price Output:** Parses string prices into floating-point numbers for easy downstream calculations or threshold triggers.

### Use Cases

- **Crypto Price Alerts:** Trigger automated email, Slack, or Telegram alerts when Bitcoin or Ethereum crosses price thresholds.
- **Portfolio Tracking & Valuation:** Periodically query prices to calculate total asset value across crypto portfolios.
- **Trading Bots & Automations:** Supply price inputs to conditional logic nodes or custom decision scripts.
- **Dashboard Displays:** Stream live ticker prices to analytics dashboards or user portals.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `symbol` | `string` | ✅ Yes | — | The Binance trading pair symbol (e.g. `BTCUSDT`, `ETHUSDT`, `SOLUSDT`). Case-insensitive. |

---

### Parameter Details

#### `symbol`
The trading pair symbol to query on Binance. 
- Must be a valid pair available on Binance (e.g., `BTCUSDT`, `ETHUSDT`, `BNBUSDT`, `SOLUSDT`, `BTCEUR`).
- Automatically converted to uppercase by the node.
- If left empty in parameter configuration, the node attempts to use incoming string data from the preceding node.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming data from preceding node. If parameter `symbol` is empty, the string value of `input` is used as the symbol. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when price fetch succeeds. Contains `success`, `symbol`, `price`, and `note`. |
| `error` | `Error` | Emitted when symbol is invalid, network fails, or Binance returns an error. |

---

### Output Data Structure

The `success` output returns a structured price response object:

```json
{
  "success": true,
  "symbol": "BTCUSDT",
  "price": 64520.5,
  "note": "Price is in the quote currency (e.g., USDT for BTCUSDT)"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates successful retrieval (`true`) |
| `symbol` | `string` | The uppercase trading pair symbol (e.g., `"BTCUSDT"`) |
| `price` | `number` | The current market price as a floating-point number |
| `note` | `string` | Explanatory note indicating the quote currency |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Query Bitcoin (BTC/USDT) Price

Retrieve the current price of Bitcoin in USDT.

**Parameter Configuration:**

```text
Symbol: BTCUSDT
```

**Output Response:**

```json
{
  "success": true,
  "symbol": "BTCUSDT",
  "price": 64520.5,
  "note": "Price is in the quote currency (e.g., USDT for BTCUSDT)"
}
```

---

### Example 2: Dynamic Symbol from Function Node

Pass the symbol dynamically from an upstream node (e.g., Function or Webhook).

**Workflow Pattern:**

```text
Manual Trigger
  → Function (returns "ETHUSDT")
  → Binance Price (symbol: expression from Function output)
  → Log
```

---

### Example 3: Crypto Price Threshold Alert

Check Ethereum price every 15 minutes and send a notification if it drops below a set limit.

**Workflow Pattern:**

```text
Cron Trigger (every 15 min)
  → Binance Price (symbol: "ETHUSDT")
  → Function (if price < 2500 → send alert)
  → Slack / Email Send
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Get cryptocurrency price from Binance API
```

### Common Patterns

- **Live Price Monitoring:** Cron Trigger → Binance Price → Log / Database Store.
- **Multi-Asset Check:** Function (list of symbols) → For Each → Binance Price → Aggregate.
- **Price Alert Bot:** Binance Price → Conditional IF Node (price > target) → Telegram Send.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `Symbol is required (e.g., BTCUSDT)`

**Cause:** The `symbol` parameter was not provided and no incoming string data was supplied.

**Solution:** Set `symbol` parameter explicitly (e.g., `BTCUSDT`) or ensure the preceding node outputs a valid symbol string.

#### Error: `Binance API error: 400`

**Cause:** The symbol specified does not exist or is misspelled on Binance (e.g. `INVALIDPAIR`).

**Solution:** Check the pair symbol on [Binance.com](https://www.binance.com) to confirm it is actively traded (e.g. use `BTCUSDT` instead of `BTC-USDT`).

#### Network or Timeout Failures

**Cause:** Outbound network connectivity issue or temporary Binance API rate limiting.

**Solution:** Binance public rate limits allow up to 1200 requests per minute per IP. Ensure your workflow does not exceed these limits.

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Symbol is required (e.g., BTCUSDT)` | Missing symbol parameter or input | Provide a valid trading pair symbol |
| `Binance API error: 400` | Invalid or unsupported trading pair | Verify symbol on Binance (e.g. `BTCUSDT`) |
| `Binance price lookup failed: ...` | API error or network connectivity loss | Check internet connection and retry |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](../http-request/en.md) — Make custom API calls to Binance or other crypto exchanges
- [Function](../function/en.md) — Process and format price outputs or check thresholds
- [Log](../log/en.md) — View price query results in workflow console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-10 | Initial release |

<!-- /SECTION: changelog -->
