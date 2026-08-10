---
node_id: "awesome-currency"
title: "AwesomeAPI: Currency Quotes"
description: "Get real-time or historical exchange rates via AwesomeAPI for automation workflows."
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - currency
  - fx
  - exchange-rate
  - finance
related_nodes:
  - http-request
  - schedule
  - function
---

<!-- SECTION: header -->
# AwesomeAPI: Currency Quotes

> **Category:** Business & Commerce | **Subcategory:** Finance & Accounting | **Type:** Action Node

Retrieve current or historical currency exchange rates using the AwesomeAPI service. This node simplifies fetching pair quotes (for example `USD-EUR`) and returns normalized JSON results for use in downstream steps.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **AwesomeAPI: Currency Quotes** node calls the AwesomeAPI endpoints to fetch foreign-exchange rates. It supports `latest` (real-time) and `historical` modes and accepts one or more currency pairs.

Key features:

- Fetch latest FX quotes for one or many pairs (e.g., `USD-EUR,USD-BRL`)
- Retrieve historical rates for a specific date
- Normalize API response into a concise JSON schema
- Expose errors through the `error` output for robust flows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mode` | `enum` | ✅ Yes | `latest` | `latest` or `historical` to choose the endpoint behavior |
| `pairs` | `string` | ✅ Yes | — | Comma-separated currency pairs using dash notation, e.g., `USD-EUR,USD-BRL` |
| `date` | `string` | Conditional | — | Required when `mode` is `historical`. Format: `YYYY-MM-DD` |
| `apiKey` | `string` | No | — | Optional API key if your AwesomeAPI plan requires authentication |

### Example Request Body

```json
{
  "mode": "latest",
  "pairs": "USD-EUR,USD-BRL",
  "apiKey": "<your_api_key>"
}
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Node reads configuration from its parameters or `input` payload (e.g., `input.pairs`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Normalized result containing one entry per pair with rate, base, quote, and timestamp |
| `error` | `Error` | Emitted when the API call fails or required parameters are missing |

### Example Success Payload

```json
{
  "quotes": [
    {"pair": "USD-EUR", "base": "USD", "quote": "EUR", "rate": 0.9142, "timestamp": "2026-08-10T11:00:00Z"},
    {"pair": "USD-BRL", "base": "USD", "quote": "BRL", "rate": 5.1234, "timestamp": "2026-08-10T11:00:00Z"}
  ]
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Latest Rates

**Configuration:** `mode=latest`, `pairs=USD-EUR`

**Output:** `success` emits the `quotes` array with the latest rate and timestamp.

### Historical Rate

**Configuration:** `mode=historical`, `pairs=USD-EUR`, `date=2023-12-31`

**Output:** Rate for the requested date is returned; when unavailable, the node emits an `error` with details.

---

## Notes

- Respect AwesomeAPI rate limits and authentication requirements configured by your subscription.
- For production workflows, store `apiKey` securely (secrets manager) rather than embedding it in flow parameters.
- Use a scheduled trigger node to poll rates periodically and persist or act on changes.
