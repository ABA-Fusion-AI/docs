---
node_id: "rest-countries"
title: "REST Countries"
description: "Get country information using the REST Countries v5 API."
category: "Web Search & Information"
subcategory: "Reference"
version: "1.0.0"
language: "en"
last_updated: "2026-08-28"
author: "Fusion Team"
tags:
  - rest-countries
  - country
  - geography
  - currency
  - flags
  - population
  - iso-codes
  - geospatial
related_nodes:
  - geo-names
  - ip-geolocation
  - currency-converter
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# REST Countries

> **Category:** Web Search & Information | **Subcategory:** Reference | **Type:** Action Node

Retrieve comprehensive country metadata, geographic coordinates, capitals, currencies, languages, population statistics, calling codes, and flag vectors using the **REST Countries v5 API**.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **REST Countries** node connects workflows to the official REST Countries v5 API (`https://api.restcountries.com/countries/v5`), enabling rapid retrieval of structured demographic, economic, and geopolitical details for countries worldwide.

The node automatically encodes country search queries, authenticates via headers, and normalizes the v5 API response into an accessible, clean JSON format.

### Key Features

- **Comprehensive Country Profile:** Extracts official and common names, ISO codes (`cca2`, `cca3`, `ccn3`, `cioc`), capitals, regions, and UN membership.
- **Economic & Communication Data:** Provides currency names/symbols and international dialing calling codes.
- **Flags & Vector Assets:** Supplies flag emoji characters as well as direct CDN URLs for PNG and SVG flag images.
- **Geographic & Map Links:** Returns latitude/longitude coordinates, land area in km², border country codes, and direct links to Google Maps and OpenStreetMap.
- **Flexible Dynamic Input:** Accepts country queries directly from node configuration or dynamically from incoming workflow payloads (`input`).

### Use Cases

- **E-Commerce Checkout & Address Enrichment:** Automatically populate currency, international phone prefixes, and timezone information based on customer country.
- **Geographic Validation & Normalization:** Convert user-entered country names into standardized ISO 3166-1 alpha-2 or alpha-3 codes (`MA`, `MAR`, `FR`, `FRA`).
- **AI Agent Context Grounding:** Provide LLMs and AI agents with verified facts about countries, population, and spoken languages.
- **User Interface Customization:** Retrieve flag SVGs, official native translations, and localized country names.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `apiKey` | `string` | ✅ Yes | — | Your REST Countries v5 API key obtained from [restcountries.com](https://restcountries.com). |
| `name` | `string` | Conditional | — | The common or official name of the country to search (e.g., `Morocco`, `France`, `Japan`). If omitted, incoming data from the previous node is used. |

---

### Parameter Details

#### `apiKey`
Your REST Countries v5 secret API key.
- Register and obtain your key at [restcountries.com](https://restcountries.com).
- Pass it directly in the parameter field or reference it securely via expressions (e.g., `{{secrets.REST_COUNTRIES_API_KEY}}`).

#### `name`
The country name to look up (case-insensitive).
- Supports common names (e.g. `Morocco`, `Germany`, `Saudi Arabia`).
- If left empty, the node automatically uses the string value passed to its input port (`input`).

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data. If the `name` configuration parameter is empty, `input` (or `String(input)`) is used as the country name. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when country details are successfully fetched. Returns an object containing the array of matched country results. |
| `error` | `Error` | Emitted when the API key is missing, country is not found, or an HTTP error is returned by the API. |

---

### Output Data Structure

```json
{
  "success": true,
  "name": "Morocco",
  "results": [
    {
      "name": {
        "common": "Morocco",
        "official": "Kingdom of Morocco",
        "nativeName": {
          "ara": {
            "official": "المملكة المغربية",
            "common": "المغرب"
          },
          "ber": {
            "official": "ⵜⴰⴳⵍⴷⵉⵜ ⵏ ⵓⵎⵔⵔⵓⴽ",
            "common": "ⵍⵎⵖⵔⵉⴱ"
          }
        }
      },
      "cca2": "MA",
      "cca3": "MAR",
      "ccn3": "504",
      "cioc": "MAR",
      "independent": true,
      "status": "officially-assigned",
      "unMember": true,
      "currencies": {
        "MAD": {
          "name": "Moroccan dirham",
          "symbol": "د.م."
        }
      },
      "capital": [
        "Rabat"
      ],
      "region": "Africa",
      "subregion": "Northern Africa",
      "languages": {
        "ara": "Arabic",
        "ber": "Berber"
      },
      "latlng": [
        32.0,
        -5.0
      ],
      "landlocked": false,
      "borders": [
        "DZA",
        "ESH",
        "ESP"
      ],
      "area": 446550,
      "flag": "🇲🇦",
      "flags": {
        "png": "https://flagcdn.com/w320/ma.png",
        "svg": "https://flagcdn.com/ma.svg"
      },
      "population": 36910558,
      "timezones": [
        "UTC+01:00"
      ],
      "continents": [
        "Africa"
      ],
      "calling_codes": {
        "root": "+2",
        "suffixes": [
          "12"
        ]
      },
      "links": {
        "googleMaps": "https://goo.gl/maps/7PusycSQnfPZfqWV8",
        "openStreetMaps": "https://www.openstreetmap.org/relation/3630439"
      }
    }
  ],
  "total_results": 1
}
```

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether the query executed successfully |
| `name` | `string` | The search term queried |
| `total_results` | `number` | Total number of matched country objects in the `results` array |
| `results[].name` | `object` | Common, official, and native names |
| `results[].cca2` | `string` | ISO 3166-1 alpha-2 two-letter country code |
| `results[].cca3` | `string` | ISO 3166-1 alpha-3 three-letter country code |
| `results[].ccn3` | `string` | ISO 3166-1 numeric three-digit code |
| `results[].cioc` | `string` | International Olympic Committee code |
| `results[].currencies` | `object` | Currency codes, names, and symbols |
| `results[].capital` | `array` | Capital cities |
| `results[].region` | `string` | Macro-geographical region (e.g., `Africa`, `Europe`, `Asia`) |
| `results[].subregion` | `string` | Sub-geographical region (e.g., `Northern Africa`) |
| `results[].languages` | `object` | Languages spoken in the country |
| `results[].latlng` | `array` | Latitude and longitude geographic coordinates `[lat, lng]` |
| `results[].area` | `number` | Total land area in square kilometers |
| `results[].flag` | `string` | Flag emoji character |
| `results[].flags` | `object` | URLs to PNG and SVG flag graphics |
| `results[].population` | `number` | Estimated country population |
| `results[].timezones` | `array` | List of UTC timezone offsets |
| `results[].calling_codes` | `object` | International dialing root prefix and suffixes |
| `results[].links` | `object` | Direct links to Google Maps and OpenStreetMap |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Static Lookup for a Country

Retrieve complete details for `Morocco`.

**Parameter Configuration:**

```text
ApiKey: YOUR_REST_COUNTRIES_API_KEY
Name:   Morocco
```

---

### Example 2: Dynamic Country Resolution from Webhook or Form

Receive user registration data and enrich the country profile dynamically.

**Workflow Pattern:**

```text
Webhook / Form Trigger
  → Function (extract country from payload: "Japan")
  → REST Countries (name: {{Function.countryName}}, apiKey: {{secrets.API_KEY}})
  → Function (extract ISO codes and currency)
  → Log
```

---

### Example 3: International Phone & Currency Validator

Validate international checkout orders by looking up currency symbols and dialing codes before initiating payment.

**Workflow Pattern:**

```text
Order Created Webhook
  → REST Countries (name: {{input.shipping_country}})
  → Function (verify currency matches results[0].currencies)
  → Payment Gateway Node
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Lookup country information using REST Countries API
```

### Common Patterns

- **Geographic Normalization:** User Input → REST Countries → Extract `cca2` & `cca3` → Save to Database.
- **Flag & Info Display:** Frontend Webhook → REST Countries → Return flag SVG & Dialing codes.
- **RAG & AI Enrichment:** User Query → REST Countries → Prompt Builder → AI Chat.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Error: `REST Countries API Key is required for v5 API...`

**Cause:** The `apiKey` parameter was left blank.

**Solution:** Provide a valid API key obtained from [restcountries.com](https://restcountries.com) in the `apiKey` parameter.

#### Error: `Country name is required`

**Cause:** The `name` parameter was omitted in the configuration, and no string value was provided in the incoming workflow data.

**Solution:** Specify a country name in the node UI or ensure the upstream node emits a non-empty string.

#### Error: `REST Countries request failed: REST Countries API error: 404`

**Cause:** The country name was misspelled or not recognized by the REST Countries v5 database.

**Solution:** Check the spelling of the country name (e.g. use standard English names like `Germany` instead of `Deutschland` or `United States` instead of abbreviations).

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `REST Countries API Key is required...` | Missing API key | Set `apiKey` parameter with key from restcountries.com |
| `Country name is required` | Missing search query | Provide country name in config or input port |
| `REST Countries API error: 401 / 403` | Invalid or expired API key | Check API key on restcountries.com dashboard |
| `REST Countries API error: 404` | Country not found | Verify country name spelling |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [GeoNames](../geo-names/en.md) — Search worldwide geographical places and postal codes
- [IP Geolocation](../ip-geolocation/en.md) — Resolve IP addresses to country and coordinates
- [Currency Converter](../currency-converter/en.md) — Perform foreign exchange rate calculations
- [HTTP Request](../http-request/en.md) — Make custom API calls
- [Function](../function/en.md) — Transform and parse country results
- [Log](../log/en.md) — Inspect output structures in workflow console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-28 | Initial release |

<!-- /SECTION: changelog -->
