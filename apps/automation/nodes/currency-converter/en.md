---
node_id: "currency-converter"
title: "Currency Converter"
description: "Convert currencies and get exchange rates with historical data support"
category: "Business & Commerce"
subcategory: "Finance & Accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - currency
  - exchange-rate
  - finance
  - conversion
related_nodes:
  - date-calculator
  - function
  - http-request
---

<!-- SECTION: header -->
# Currency Converter

> **Category:** Utilities | **Type:** Action Node

Convert currencies, retrieve exchange rates, and optionally work with historical exchange data.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Currency Converter** node helps convert amounts between currencies and retrieve current or historical exchange rates. It is useful for invoices, financial workflows, multi-currency reporting, and pricing automation.

### Key Features

- **Currency Conversion:** Convert an amount from one currency to another
- **Exchange Rate Lookup:** Retrieve the latest market rate for a currency pair
- **Historical Data Support:** Access exchange rates for specific dates
- **Simple Configuration:** Use a compact set of parameters for common financial use cases

### Use Cases

- Convert transaction values to the base currency
- Prepare invoices or quotes in multiple currencies
- Pull historical exchange rates for reporting
- Automate financial calculations in business workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fromCurrency` | `string` | ✅ Yes | — | Source currency code such as `USD` |
| `toCurrency` | `string` | ✅ Yes | — | Target currency code such as `EUR` |
| `amount` | `number` | ✅ Yes | — | Amount to convert |
| `date` | `string` | ❌ No | — | Optional date for historical exchange rate lookup |

### Example

```text
fromCurrency: "USD"
toCurrency: "EUR"
amount: 100
date: "2026-01-15"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Optional incoming data that can drive the conversion values |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Conversion result or exchange rate data |
| `error` | `object` | Error details if the conversion fails |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Convert USD to EUR

```text
fromCurrency: "USD"
toCurrency: "EUR"
amount: 100
```

**Result:**

```json
{
  "amount": 100,
  "fromCurrency": "USD",
  "toCurrency": "EUR",
  "rate": 0.92,
  "convertedAmount": 92
}
```

### Example: Historical Rate Lookup

```text
fromCurrency: "USD"
toCurrency: "GBP"
amount: 50
date: "2025-12-01"
```

<!-- /SECTION: examples -->
