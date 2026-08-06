---
node_id: "alpha-vantage"
title: "Alpha Vantage"
description: "Fetch stock market data, forex rates, and cryptocurrency information from the Alpha Vantage API."
category: "business-commerce"
subcategory: "finance-accounting"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - finance
  - stocks
  - forex
  - cryptocurrency
  - market-data
related_nodes:
  - financial-modeling-prep
  - awesome-currency
  - currency-converter
---

<!-- SECTION: header -->
# Alpha Vantage

> **Category:** Business & Commerce | **Type:** Action Node

Retrieve financial market data from the Alpha Vantage API, including stock prices, foreign exchange rates, cryptocurrency information, company symbols, and time series data.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Alpha Vantage** node connects to the Alpha Vantage API and retrieves financial market information. It supports multiple API functions, allowing workflows to fetch stock prices, foreign exchange rates, cryptocurrency data, company symbol searches, and historical market time series.

The node builds the API request automatically based on the configured parameters and returns the response in either JSON or CSV format.

### Key Features

- **Stock Market Data:** Retrieve real-time and historical stock information.
- **Forex Exchange Rates:** Get currency exchange rates between two currencies.
- **Cryptocurrency Data:** Retrieve cryptocurrency market information.
- **Symbol Search:** Search publicly traded companies by keyword.
- **Time Series Support:** Retrieve daily and intraday market data.
- **JSON or CSV Output:** Select the preferred response format.
- **Dynamic Symbol Input:** Accept the stock symbol from workflow input or node configuration.
- **Automatic Error Handling:** Handles API errors, invalid requests, and rate limiting.

### Supported API Functions

The node can call any Alpha Vantage API function.

Common examples include:

| Function | Description |
|----------|-------------|
| `GLOBAL_QUOTE` | Retrieve the latest market quote for a stock. |
| `TIME_SERIES_DAILY` | Daily historical prices. |
| `TIME_SERIES_INTRADAY` | Intraday market data. |
| `SYMBOL_SEARCH` | Search company symbols. |
| `CURRENCY_EXCHANGE_RATE` | Retrieve forex exchange rates. |

### Use Cases

- Retrieve stock prices
- Build financial dashboards
- Monitor investment portfolios
- Fetch exchange rates
- Search listed companies
- Automate market monitoring
- Integrate financial data into workflows
- Build reporting pipelines

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration
### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `function` | `string` | ✅ Yes | — | Alpha Vantage API function to execute (e.g. `GLOBAL_QUOTE`, `TIME_SERIES_DAILY`, `SYMBOL_SEARCH`). |
| `symbol` | `string` | ❌ No | — | Stock symbol such as `IBM`, `AAPL`, or `GOOGL`. Can also be provided by the previous workflow node. |
| `apiKey` | `string` | ✅ Yes | — | Alpha Vantage API key. |
| `interval` | `string` | ❌ No | — | Intraday interval (`1min`, `5min`, `15min`, `30min`, `60min`). Used by intraday functions. |
| `outputsize` | `string` | ❌ No | `compact` | Number of returned data points. Accepts `compact` or `full`. |
| `datatype` | `string` | ❌ No | `json` | Response format. Accepts `json` or `csv`. |
| `keywords` | `string` | ❌ No | — | Search keywords for the `SYMBOL_SEARCH` function. |
| `fromCurrency` | `string` | ❌ No | — | Source currency for exchange rate requests. |
| `toCurrency` | `string` | ❌ No | — | Destination currency for exchange rate requests. |

### Default Values

| Parameter | Default |
|-----------|---------|
| `outputsize` | `compact` |
| `datatype` | `json` |

### Query Resolution

The node resolves parameters using the following priority:

1. Values received from the previous workflow node.
2. Values configured in the node.

For example, if the incoming workflow data contains:

```json
{
  "symbol": "AAPL"
}
```

the node automatically uses `AAPL` even if another symbol is configured.

### Supported Output Formats

| Format | Description |
|--------|-------------|
| `json` | Structured JSON response. Recommended for most workflows. |
| `csv` | CSV response returned by the Alpha Vantage API. |

### Common API Functions

| Function | Required Parameters |
|----------|--------------------|
| `GLOBAL_QUOTE` | `symbol` |
| `TIME_SERIES_DAILY` | `symbol` |
| `TIME_SERIES_INTRADAY` | `symbol`, `interval` |
| `SYMBOL_SEARCH` | `keywords` |
| `CURRENCY_EXCHANGE_RATE` | `fromCurrency`, `toCurrency` |

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional workflow input used to override configured parameters. |

The node accepts workflow input such as:

```json
{
  "symbol": "IBM"
}
```

When a symbol is provided through the workflow input, it overrides the configured value.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Alpha Vantage API response. |
| `error` | `Error` | Returned when the request fails. |

### Successful Response

```json
{
  "success": true,
  "function": "GLOBAL_QUOTE",
  "symbol": "IBM",
  "data": {
    "Global Quote": {
      "01. symbol": "IBM",
      "02. open": "248.2500",
      "03. high": "251.6800",
      "04. low": "247.7800",
      "05. price": "251.2300",
      "06. volume": "3445123",
      "07. latest trading day": "2026-08-06",
      "08. previous close": "249.1800",
      "09. change": "2.0500",
      "10. change percent": "0.82%"
    }
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": "API Key is required"
}
```

or

```json
{
  "success": false,
  "error": "API rate limit exceeded. Please wait and try again."
}
```

<!-- /SECTION: inputs-outputs -->
<!-- SECTION: examples -->
## Examples

### Basic Example: Get a Stock Quote

Retrieve the latest stock market quote.

**Configuration:**

```text
Function: GLOBAL_QUOTE
Symbol: IBM
Outputsize: compact
Datatype: json
```

**Output:**

```json
{
  "success": true,
  "function": "GLOBAL_QUOTE",
  "symbol": "IBM",
  "data": {
    "Global Quote": {
      "01. symbol": "IBM",
      "05. price": "251.23",
      "10. change percent": "0.82%"
    }
  }
}
```

---

### Example: Daily Time Series

Retrieve historical daily prices.

**Configuration:**

```text
Function: TIME_SERIES_DAILY
Symbol: AAPL
Outputsize: compact
Datatype: json
```

The response contains daily historical market data for the selected stock.

---

### Example: Intraday Time Series

Retrieve intraday market data.

**Configuration:**

```text
Function: TIME_SERIES_INTRADAY
Symbol: MSFT
Interval: 5min
Datatype: json
```

The interval parameter is required for intraday requests.

---

### Example: Symbol Search

Search for companies matching a keyword.

**Configuration:**

```text
Function: SYMBOL_SEARCH
Keywords: microsoft
```

The response returns matching companies and their trading symbols.

---

### Example: Currency Exchange Rate

Retrieve a foreign exchange rate.

**Configuration:**

```text
Function: CURRENCY_EXCHANGE_RATE
From Currency: USD
To Currency: EUR
```

The response contains the current exchange rate.

---

### Example: CSV Output

Return the response in CSV format.

**Configuration:**

```text
Function: TIME_SERIES_DAILY
Symbol: IBM
Datatype: csv
```

The API returns CSV instead of JSON.

---

### Example: Full Dataset

Retrieve the complete historical dataset.

**Configuration:**

```text
Function: TIME_SERIES_DAILY
Symbol: IBM
Outputsize: full
```

The node requests the complete available time series.

---

### Example: Dynamic Symbol

Receive the symbol from the previous workflow node.

**Input**

```json
{
  "symbol": "TSLA"
}
```

**Configuration**

```text
Function: GLOBAL_QUOTE
Symbol: IBM
```

Because the workflow input contains a symbol, the node uses:

```text
TSLA
```

instead of the configured value.

---

### Example: Error — Missing API Key

**Configuration**

```text
Function: GLOBAL_QUOTE
Symbol: IBM
ApiKey: (empty)
```

**Output**

```json
{
  "success": false,
  "error": "API Key is required"
}
```

---

### Example: Error — Rate Limit

If the free Alpha Vantage API limit is exceeded, the node returns:

```json
{
  "success": false,
  "error": "API rate limit exceeded. Please wait and try again."
}
```

<!-- /SECTION: examples -->
---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch an IBM Global Quote
```

### Common Patterns

- **Stock Quote:** Manual Trigger → Alpha Vantage → Log
- **Portfolio Monitoring:** Scheduler → Alpha Vantage → Function
- **Currency Monitoring:** Scheduler → Alpha Vantage → Notification
- **Market Dashboard:** Alpha Vantage → Database → Dashboard
- **Symbol Lookup:** HTTP Request → Alpha Vantage → Function
- **Historical Analysis:** Alpha Vantage → Function → Google Sheets

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### "API Key is required"

**Cause:** No Alpha Vantage API key was configured.

**Solution:** Generate a free API key from Alpha Vantage and configure the `apiKey` parameter.

---

#### "Function is required"

**Cause:** The API function parameter is missing.

**Solution:** Configure a valid Alpha Vantage function such as:

- `GLOBAL_QUOTE`
- `TIME_SERIES_DAILY`
- `TIME_SERIES_INTRADAY`
- `SYMBOL_SEARCH`
- `CURRENCY_EXCHANGE_RATE`

---

#### "API rate limit exceeded"

**Cause:** The free Alpha Vantage API quota has been exceeded.

**Solution:** Wait before retrying or upgrade your Alpha Vantage plan.

---

#### "Error Message"

**Cause:** Alpha Vantage returned an API error.

**Solution:** Verify the configured parameters such as:

- Function
- Symbol
- Interval
- Currency codes
- Keywords

---

#### Empty response

**Cause:** No matching data was found.

**Solution:** Verify that the stock symbol, keywords, or currency codes are valid.

---

#### Invalid symbol

**Cause:** The provided stock symbol does not exist.

**Solution:** Use the `SYMBOL_SEARCH` function to find the correct trading symbol.

### Error Codes

| Error | Description |
|-------|-------------|
| `API Key is required` | No API key was configured. |
| `Function is required` | No Alpha Vantage function was configured. |
| `API rate limit exceeded` | The free API quota has been exceeded. |
| `Error Message` | Alpha Vantage returned an API error. |
| `Network Error` | Unable to connect to Alpha Vantage. |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- Financial Modeling Prep
- Currency Converter
- Binance Price
- CoinGecko Price

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release. |

<!-- /SECTION: changelog -->