---

node_id: "file-size-converter"

title: "File Size Converter"

description: "Convert between all digital storage units (B, KB, MB, GB, TB, PB) with binary and decimal options."

category: "Security & Networking"

subcategory: "APIs & Protocols"

version: "1.0.0"

language: "en"

last_updated: "2026-08-12"

author: "Fusion Team"

tags:

  - storage

  - file size

  - conversion

  - bytes

  - units

related_nodes:

  - base-converter

  - binary-to-text

---



<!-- SECTION:header -->

# File Size Converter



> **Category:** Storage & Files | **Subcategory:** Files & Documents | **Type:** Action Node



Convert file size values between bytes and higher storage units using either decimal (1000-based) or binary (1024-based) unit definitions.



<!-- /SECTION:header -->



---



## Overview



The **File Size Converter** node supports four operations: `convert`, `format`, `parse`, and `calculateTransferTime`.



- `convert` converts a numeric size from one unit to another.

- `format` converts a byte count into a formatted size string and returns the equivalent values across all supported units.

- `parse` reads a size string like `100 MB` and returns the interpreted bytes and formatted output.

- `calculateTransferTime` estimates transfer duration for a byte count over a given bandwidth in megabits per second.



Supported units:



- Binary / 1024-based: `B`, `KB`, `MB`, `GB`, `TB`, `PB`

- Decimal / 1000-based: `kB`, `mB`, `gB`, `tB`, `pB`



### Features



- Convert between bytes, kilobytes, megabytes, gigabytes, terabytes, and petabytes.

- Format raw byte counts into human-readable size strings.

- Parse size expressions into bytes and normalized outputs.

- Calculate transfer time from bytes and Mbps.



---



## Configuration



### Parameters



| Parameter | Type | Required | Default | Description |

|-----------|------|----------|---------|-------------|

| `operation` | `enum` | ✅ Yes | `convert` | Operation to perform: `convert`, `format`, `parse`, `calculateTransferTime`. |

| `value` | `number` | No | `1024` | Numeric amount used by `convert` along with `fromUnit` and `toUnit`. |

| `fromUnit` | `string` | No | `KB` | Source unit for `convert`. Supported units: `B`, `KB`, `MB`, `GB`, `TB`, `PB`, `kB`, `mB`, `gB`, `tB`, `pB`. |

| `toUnit` | `string` | No | `MB` | Target unit for `convert`. Same supported units as `fromUnit`. |

| `sizeString` | `string` | No | `""` | Size expression for `parse`, such as `100 MB` or `1.5 GB`. |

| `bytes` | `number` | No | `0` | Byte count used by `format` and `calculateTransferTime`. |

| `speedMbps` | `number` | No | `10` | Network speed in megabits per second for `calculateTransferTime`. |

| `decimals` | `number` | No | `2` | Number of decimal places for formatted output and unit conversion display. |



### Operation examples



Convert 2048 kilobytes to megabytes:



```json

{

  "operation": "convert",

  "value": 2048,

  "fromUnit": "KB",

  "toUnit": "MB",

  "decimals": 2

}

```



Format 1234567 bytes:



```json

{

  "operation": "format",

  "bytes": 1234567,

  "decimals": 2

}

```



Parse a size string:



```json

{

  "operation": "parse",

  "sizeString": "1.5 GB",

  "decimals": 2

}

```



Calculate transfer time for 5 GB over 50 Mbps:



```json

{

  "operation": "calculateTransferTime",

  "bytes": 5368709120,

  "speedMbps": 50

}

```



---



## Inputs & Outputs



### Inputs



| Input | Type | Description |

|-------|------|-------------|

| `input` | `any` | Node reads `input.value` when parameter values are not provided directly. |



### Outputs



| Output | Type | Description |

|--------|------|-------------|

| `success` | `object` | Operation result object with fields dependent on the selected operation. |

| `error` | `Error` | Returned when parsing or conversion fails. |



#### Example outputs by operation



`convert` success payload:



```json

{

  "success": true,

  "value": 2,

  "fromUnit": "KB",

  "toUnit": "MB",

  "bytes": 2097152,

  "formatted": "2 MB",

  "allUnits": {

    "B": 2097152,

    "KB": 2048,

    "MB": 2,

    "GB": 0.001953125,

    "TB": 0.0000019073486328125,

    "PB": 0.000000001862645149230957

  }

}

```



`format` success payload:



```json

{

  "success": true,

  "bytes": 1234567,

  "formatted": "1.18 MB",

  "allUnits": {

    "B": 1234567,

    "KB": 1205.63,

    "MB": 1.18,

    "GB": 0.00,

    "TB": 0.00,

    "PB": 0.00

  }

}

```



`parse` success payload:



```json

{

  "success": true,

  "input": "1.5 GB",

  "value": 1.5,

  "unit": "GB",

  "bytes": 1610612736,

  "formatted": "1.50 GB"

}

```



`calculateTransferTime` success payload:



```json

{

  "success": true,

  "totalSeconds": 858.9934592,

  "formatted": "0h 14m 18s",

  "hours": 0,

  "minutes": 14,

  "seconds": 18

}

```



---



## Notes



- The node uses uppercase units (`KB`, `MB`, `GB`, `TB`, `PB`) for 1024-based conversions and lowercase units (`kB`, `mB`, `gB`, `tB`, `pB`) for 1000-based conversions.

- `parse` accepts a numeric value followed by a unit suffix, such as `100 MB` or `1.5 GB`.

- `calculateTransferTime` converts bytes to bits before dividing by the network speed in megabits per second.

- `decimals` controls rounding for formatted outputs.



and make documnetation for this node :

{"nodes":[{"id":"nf6dxq748pk9nl9cnlhs0i72","position":{"x":299,"y":323},"type":"action","width":64,"height":64,"data":{"name":"exchange-rate-api","label":"ExchangeRate API1","inputs":{"input":{"label":"Input"}},"outputs":{"success":{"label":"Success"},"error":{"label":"Error"}},"description":"Get exchange rates from ExchangeRate-API.","showRunningStatus":true,"parameters":{},"_morphing":false},"selected":true}],"edges":[]}



this is his code base :

import { ActionNode } from "@aba-fusion-ai/workflow";

import v from "@aba-fusion-ai/validator";

import type { SchemaTypeAny } from "@aba-fusion-ai/validator";



const schema: SchemaTypeAny = v.object({

  key: v.string(),

  base: v.string().optional().default("USD"),

});



type Parameters = v.infer<typeof schema>;



export class ExchangeRateApiNode extends ActionNode<Parameters> {

  metadata = {

    label: "ExchangeRate API",

    description: "Get exchange rates from ExchangeRate-API.",

    showRunningStatus: true,

  };

  public readonly schema: SchemaTypeAny = schema;



  /**

   * Get exchange rates from ExchangeRate-API

   */

  private async getExchangeRates(

    apiKey: string,

    base: string,

  ): Promise<unknown> {

    if (!apiKey) {

      throw new Error("API key is required");

    }



    // Replace [key] in the path

    const url = `https://v6.exchangerate-api.com/v6/${apiKey}/latest/${base.toUpperCase()}`;



    const response = await fetch(url, {

      headers: {

        "Content-Type": "application/json",

      },

    });



    if (!response.ok) {

      throw new Error(`ExchangeRate-API error: ${response.status}`);

    }



    const data = (await response.json()) as any as {

      result?: string;

      documentation?: string;

      terms_of_use?: string;

      time_last_update_unix?: number;

      time_last_update_utc?: string;

      time_next_update_unix?: number;

      time_next_update_utc?: string;

      base_code?: string;

      conversion_rates?: Record<string, number>;

    };



    return {

      success: data.result === "success",

      base_currency: data.base_code || base.toUpperCase(),

      rates: data.conversion_rates || {},

      time_last_update: {

        unix: data.time_last_update_unix || null,

        utc: data.time_last_update_utc || null,

      },

      time_next_update: {

        unix: data.time_next_update_unix || null,

        utc: data.time_next_update_utc || null,

      },

      documentation: data.documentation || null,

      terms_of_use: data.terms_of_use || null,

      note: "Rates are relative to the base currency",

    };

  }



  protected async handleTick(

    data: unknown,

    _session: unknown,

  ): Promise<unknown> {

    const { key, base } = this.config;



    if (!key) {

      throw new Error("API key is required");

    }



    try {

      return await this.getExchangeRates(key, base || "USD");

    } catch (error) {

      throw new Error(

        `ExchangeRate-API lookup failed: ${error instanceof Error ? error.message : String(error)}`,

      );

    }

  }



  public async stop(): Promise<void> {

    return;

  }

}

node_id: "exchange-rate-api"
title: "ExchangeRate API"
description: "Fetch real-time foreign exchange rates for any base currency using ExchangeRate-API."
category: "Finance & Market"
subcategory: "Currency & FX"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:

exchange rate

currency

forex

finance

api
related_nodes:

currency-converter

crypto-price-tracker

ExchangeRate API
Category: Finance & Market | Subcategory: Currency & FX | Type: Action Node

Fetch latest foreign exchange rates relative to a specified base currency directly from the ExchangeRate-API endpoint.

Overview
The ExchangeRate API node retrieves up-to-date conversion rates for world currencies using your ExchangeRate-API key.

Queries the official v6 API endpoint (/latest/{base}).

Supports dynamic base currency specification (defaults to USD).

Standardizes API metadata, including update timestamps, rate mappings, and terms of use links.

Features
Fetch real-time foreign exchange conversion rates.

Flexible base currency selection (e.g., USD, EUR, GBP).

Standardized payload response structure including UNIX and UTC timestamps for cache tracking.

Configuration
Parameters
Parameter	Type	Required	Default	Description
key	string	✅ Yes	None	Your ExchangeRate-API key.
base	string	No	"USD"	Three-letter ISO 4217 currency code used as the conversion base (e.g., USD, EUR).
Operation examples
Fetch latest rates relative to USD:

JSON
{
  "key": "your_api_key_here",
  "base": "USD"
}
Fetch latest rates relative to EUR:

JSON
{
  "key": "your_api_key_here",
  "base": "EUR"
}
Inputs & Outputs
Inputs
Input	Type	Description
input	any	Trigger input data passed from the upstream node during workflow execution.
Outputs
Output	Type	Description
success	object	Response object containing rate tables, timestamps, and currency details upon successful API lookup.
error	Error	Thrown when the API key is missing, network request fails, or invalid response status is returned.
Example output
success payload:

JSON
{
  "success": true,
  "base_currency": "USD",
  "rates": {
    "USD": 1,
    "EUR": 0.92,
    "GBP": 0.78,
    "JPY": 155.42,
    "CAD": 1.36
  },
  "time_last_update": {
    "unix": 1723420801,
    "utc": "Mon, 12 Aug 2026 00:00:01 +0000"
  },
  "time_next_update": {
    "unix": 1723507201,
    "utc": "Tue, 13 Aug 2026 00:00:01 +0000"
  },
  "documentation": "https://www.exchangerate-api.com/docs",
  "terms_of_use": "https://www.exchangerate-api.com/terms",
  "note": "Rates are relative to the base currency"
}
Notes
An active API key from ExchangeRate-API is required to execute this node.

Input currency strings are automatically converted to uppercase when hitting the API (e.g., eur is evaluated as EUR).

If no base parameter is supplied, USD will be automatically assigned.

give me a valid md file block

Markdown
---
node_id: "exchange-rate-api"
title: "ExchangeRate API"
description: "Fetch real-time foreign exchange rates for any base currency using ExchangeRate-API."
category: "Finance & Market"
subcategory: "Currency & FX"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:
  - exchange rate
  - currency
  - forex
  - finance
  - api
related_nodes:
  - currency-converter
  - crypto-price-tracker
---

<!-- SECTION:header -->
# ExchangeRate API

> **Category:** Finance & Market | **Subcategory:** Currency & FX | **Type:** Action Node

Fetch latest foreign exchange rates relative to a specified base currency directly from the ExchangeRate-API endpoint.

<!-- /SECTION:header -->

---

## Overview

The **ExchangeRate API** node retrieves up-to-date conversion rates for world currencies using your ExchangeRate-API key.

- Queries the official v6 API endpoint (`/latest/{base}`).
- Supports dynamic base currency specification (defaults to `USD`).
- Standardizes API metadata, including update timestamps, rate mappings, and terms of use links.

### Features

- Fetch real-time foreign exchange conversion rates.
- Flexible base currency selection (e.g., `USD`, `EUR`, `GBP`).
- Standardized payload response structure including UNIX and UTC timestamps for cache tracking.

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `key` | `string` | ✅ Yes | *None* | Your ExchangeRate-API key. |
| `base` | `string` | No | `"USD"` | Three-letter ISO 4217 currency code used as the conversion base (e.g., `USD`, `EUR`). |

### Operation examples

Fetch latest rates relative to USD:

```json
{
  "key": "your_api_key_here",
  "base": "USD"
}
Fetch latest rates relative to EUR:

JSON
{
  "key": "your_api_key_here",
  "base": "EUR"
}
Inputs & Outputs
Inputs
Input	Type	Description
input	any	Trigger input data passed from the upstream node during workflow execution.
Outputs
Output	Type	Description
success	object	Response object containing rate tables, timestamps, and currency details upon successful API lookup.
error	Error	Thrown when the API key is missing, network request fails, or invalid response status is returned.
Example output
success payload:

JSON
{
  "success": true,
  "base_currency": "USD",
  "rates": {
    "USD": 1,
    "EUR": 0.92,
    "GBP": 0.78,
    "JPY": 155.42,
    "CAD": 1.36
  },
  "time_last_update": {
    "unix": 1723420801,
    "utc": "Mon, 12 Aug 2026 00:00:01 +0000"
  },
  "time_next_update": {
    "unix": 1723507201,
    "utc": "Tue, 13 Aug 2026 00:00:01 +0000"
  },
  "documentation": "[https://www.exchangerate-api.com/docs](https://www.exchangerate-api.com/docs)",
  "terms_of_use": "[https://www.exchangerate-api.com/terms](https://www.exchangerate-api.com/terms)",
  "note": "Rates are relative to the base currency"
}
Notes
An active API key from ExchangeRate-API is required to execute this node.

Input currency strings are automatically converted to uppercase when hitting the API (e.g., eur is evaluated as EUR).

If no base parameter is supplied, USD will be automatically assigned.

give a full downloadble one

Fetch latest rates relative to EUR:

JSON
{
  "key": "your_api_key_here",
  "base": "EUR"
}
Inputs & Outputs
Inputs
Input	Type	Description
input	any	Trigger input data passed from the upstream node during workflow execution.
Outputs
Output	Type	Description
success	object	Response object containing rate tables, timestamps, and currency details upon successful API lookup.
error	Error	Thrown when the API key is missing, network request fails, or invalid response status is returned.
Example output
success payload:

JSON
{
  "success": true,
  "base_currency": "USD",
  "rates": {
    "USD": 1,
    "EUR": 0.92,
    "GBP": 0.78,
    "JPY": 155.42,
    "CAD": 1.36
  },
  "time_last_update": {
    "unix": 1723420801,
    "utc": "Mon, 12 Aug 2026 00:00:01 +0000"
  },
  "time_next_update": {
    "unix": 1723507201,
    "utc": "Tue, 13 Aug 2026 00:00:01 +0000"
  },
  "documentation": "[https://www.exchangerate-api.com/docs](https://www.exchangerate-api.com/docs)",
  "terms_of_use": "[https://www.exchangerate-api.com/terms](https://www.exchangerate-api.com/terms)",
  "note": "Rates are relative to the base currency"
}
Notes
An active API key from ExchangeRate-API is required to execute this node.

Input currency strings are automatically converted to uppercase when hitting the API (e.g., eur is evaluated as EUR).

If no base parameter is supplied, USD will be automatically assigned.
"""

file_path = "exchange-rate-api.md"
with open(file_path, "w", encoding="utf-8") as f:
f.write(md_content)

print(f"File created successfully: {file_path}")


```text?code_stdout&code_event_index=1
File created successfully: exchange-rate-api.md

Your Markdown document file is ready:

Icône MD
exchange-rate-api
 MD 
Ouvert
You can download the full exchange-rate-api.md file using the link above.




Gemini est une IA et peut se tromper.

---
node_id: "exchange-rate-api"
title: "ExchangeRate API"
description: "Fetch real-time foreign exchange rates for any base currency using ExchangeRate-API."
category: "Finance & Market"
subcategory: "Currency & FX"
version: "1.0.0"
language: "en"
last_updated: "2026-08-12"
author: "Fusion Team"
tags:
  - exchange rate
  - currency
  - forex
  - finance
  - api
related_nodes:
  - currency-converter
  - crypto-price-tracker
---

<!-- SECTION:header -->
# ExchangeRate API

> **Category:** Finance & Market | **Subcategory:** Currency & FX | **Type:** Action Node

Fetch latest foreign exchange rates relative to a specified base currency directly from the ExchangeRate-API endpoint.

<!-- /SECTION:header -->

---

## Overview

The **ExchangeRate API** node retrieves up-to-date conversion rates for world currencies using your ExchangeRate-API key.

- Queries the official v6 API endpoint (`/latest/{base}`).
- Supports dynamic base currency specification (defaults to `USD`).
- Standardizes API metadata, including update timestamps, rate mappings, and terms of use links.

### Features

- Fetch real-time foreign exchange conversion rates.
- Flexible base currency selection (e.g., `USD`, `EUR`, `GBP`).
- Standardized payload response structure including UNIX and UTC timestamps for cache tracking.

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `key` | `string` | ✅ Yes | *None* | Your ExchangeRate-API key. |
| `base` | `string` | No | `"USD"` | Three-letter ISO 4217 currency code used as the conversion base (e.g., `USD`, `EUR`). |

### Operation examples

Fetch latest rates relative to USD:

```json
{
  "key": "your_api_key_here",
  "base": "USD"
}
```

Fetch latest rates relative to EUR:

```json
{
  "key": "your_api_key_here",
  "base": "EUR"
}
```

---

## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Trigger input data passed from the upstream node during workflow execution. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Response object containing rate tables, timestamps, and currency details upon successful API lookup. |
| `error` | `Error` | Thrown when the API key is missing, network request fails, or invalid response status is returned. |

#### Example output

`success` payload:

```json
{
  "success": true,
  "base_currency": "USD",
  "rates": {
    "USD": 1,
    "EUR": 0.92,
    "GBP": 0.78,
    "JPY": 155.42,
    "CAD": 1.36
  },
  "time_last_update": {
    "unix": 1723420801,
    "utc": "Mon, 12 Aug 2026 00:00:01 +0000"
  },
  "time_next_update": {
    "unix": 1723507201,
    "utc": "Tue, 13 Aug 2026 00:00:01 +0000"
  },
  "documentation": "https://www.exchangerate-api.com/docs",
  "terms_of_use": "https://www.exchangerate-api.com/terms",
  "note": "Rates are relative to the base currency"
}
```

---

## Notes

- An active API key from [ExchangeRate-API](https://www.exchangerate-api.com/) is required to execute this node.
- Input currency strings are automatically converted to uppercase when hitting the API (e.g., `eur` is evaluated as `EUR`).
- If no `base` parameter is supplied, `USD` will be automatically assigned.