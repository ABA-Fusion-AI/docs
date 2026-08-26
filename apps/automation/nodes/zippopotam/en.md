---
node_id: "zippopotam"
title: "Zippopotam.us"
description: "Get location and geographic information based on zip or postal codes across 60+ countries."
category: "Location & Mapping"
subcategory: "Geocoding & Postal Codes"
version: "1.0.0"
language: "en"
last_updated: "2026-08-26"
author: "Fusion Team"
tags:
  - zippopotam
  - zipcode
  - postal-code
  - geocoding
  - location
  - address
  - places
  - coordinates
  - latitude
  - longitude
  - maps
related_nodes:
  - http-request
  - ip-geolocation
  - function
  - webhook
  - slack
  - discord-bot-send
---

<!-- SECTION: header -->
# Zippopotam.us

> **Category:** Location & Mapping | **Subcategory:** Geocoding & Postal Codes | **Type:** Action Node

Retrieve detailed geographic and location information (city, state, region, latitude, and longitude) from postal codes and country codes worldwide using the free [Zippopotam.us API](http://www.zippopotam.us/).

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Zippopotam.us** node allows automated workflows to convert postal or ZIP codes into rich geographic metadata across more than 60 supported countries. It provides instant access to location names, state/province administrative divisions, ISO country abbreviations, and precise GPS coordinates (latitude and longitude).

Because the Zippopotam.us API is completely free and requires **no API key or registration**, this node is ideal for address verification pipelines, shipping calculation workflows, localized content delivery, CRM address enrichment, and form autofill systems.

### Key Features

- **Global Coverage:** Supports postal and ZIP code lookup for 60+ countries (including United States, Canada, United Kingdom, France, Germany, Spain, Italy, Australia, and more).
- **Precise Geocoordinates:** Returns latitude and longitude coordinates for mapping, distance calculation, and routing workflows.
- **Zero Configuration / Keyless:** No API keys, subscriptions, or authentication credentials required.
- **Case-Insensitive:** Automatically normalizes country codes (e.g. `US`, `us`, `Us`).
- **Dynamic Expression Support:** Easily accept dynamic country codes and postal codes from upstream form triggers, webhooks, or CRM integrations.
- **Structured Array Output:** Handles multi-location postal codes seamlessly through a structured `places` array.

### Common Use Cases

- **Checkout & Form Autofill:** Automatically resolve City and State when a user enters their ZIP code on an e-commerce checkout form or lead capture page.
- **Delivery & Shipping Routing:** Determine geographic regions and calculate logistics zones based on customer postal codes.
- **Localized Weather & Notifications:** Fetch latitude and longitude from a postal code to trigger local weather alerts or regional marketing campaigns.
- **CRM Data Enrichment:** Cleanse and standardize customer address data by populating missing city or state fields automatically.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

Configure the Zippopotam.us node by supplying the target country code and postal/ZIP code.

![Zippopotam Node Configuration](icon.svg)

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|:--------:|:-------:|-------------|
| `country` | `string` | ✅ Yes | — | Two-letter ISO 3166-1 alpha-2 country code (e.g. `us`, `fr`, `de`, `es`, `ca`, `it`, `gb`). Case-insensitive. |
| `zip` | `string` | ✅ Yes | — | Postal or ZIP code to search for (e.g. `90210`, `75001`, `10115`, `08001`, `M5V`, `00184`). |

---

### Detailed Parameter Descriptions

#### `country` (Required)
The standard two-letter country code for the destination country.
- Examples: `us` (United States), `fr` (France), `de` (Germany), `es` (Spain), `ca` (Canada), `it` (Italy), `gb` (Great Britain).
- Case-insensitive: Both uppercase (`US`, `FR`) and lowercase (`us`, `fr`) are accepted.
- Supports expressions: e.g. `{{$json.country}}` or `{{outputs.Webhook.body.countryCode}}`.

#### `zip` (Required)
The postal code or ZIP code to query.
- Examples: `90210` (Beverly Hills), `10001` (New York), `75001` (Paris), `10115` (Berlin), `M5V` (Toronto).
- Supports expressions: e.g. `{{$json.postalCode}}` or `{{outputs.Function.zip}}`.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow trigger or data payload that initiates the lookup. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when the postal code lookup succeeds. Contains geographic and place metadata. |
| `error` | `Error` | Emitted when the postal code is not found (404) or when required parameters are missing. |

---

### Output Data Structure Example

When querying `country: "US"` and `zip: "90210"`:

```json
{
  "success": true,
  "country": "United States",
  "country_abbreviation": "US",
  "post_code": "90210",
  "places": [
    {
      "place_name": "Beverly Hills",
      "longitude": "-118.4065",
      "latitude": "34.0901",
      "state": "California",
      "state_abbreviation": "CA"
    }
  ],
  "total_places": 1
}
```

---

### Output Field Reference

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates whether the postal code query was successfully resolved (`true`). |
| `country` | `string` | Full name of the queried country (e.g. `"United States"`, `"France"`, `"Germany"`). |
| `country_abbreviation` | `string` | Official two-letter ISO country code (e.g. `"US"`, `"FR"`, `"DE"`). |
| `post_code` | `string` | The postal code that was queried (e.g. `"90210"`, `"75001"`). |
| `places` | `array` | Array of locations associated with this postal code. |
| `places[].place_name` | `string` | City, district, or town name (e.g. `"Beverly Hills"`, `"Paris 01 Louvre"`). |
| `places[].state` | `string` | State, province, or region name (e.g. `"California"`, `"Île-de-France"`, `"Ontario"`). |
| `places[].state_abbreviation` | `string` | State or administrative code (e.g. `"CA"`, `"NY"`, `"ON"`, `"BE"`). |
| `places[].latitude` | `string` | Geographic latitude in decimal degrees (e.g. `"34.0901"`). |
| `places[].longitude` | `string` | Geographic longitude in decimal degrees (e.g. `"-118.4065"`). |
| `total_places` | `number` | Total count of places returned in the `places` array. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: United States ZIP Code (Beverly Hills)

Look up geographic data for Beverly Hills, California:

**Configuration:**
- **Country:** `us`
- **Zip:** `90210`

**Output:**
```json
{
  "success": true,
  "country": "United States",
  "country_abbreviation": "US",
  "post_code": "90210",
  "places": [
    {
      "place_name": "Beverly Hills",
      "longitude": "-118.4065",
      "latitude": "34.0901",
      "state": "California",
      "state_abbreviation": "CA"
    }
  ],
  "total_places": 1
}
```

---

### Example 2: France Postal Code (Paris)

Retrieve district and coordinates for Paris, France:

**Configuration:**
- **Country:** `fr`
- **Zip:** `75001`

---

### Example 3: Germany Postal Code (Berlin)

Look up city and administrative region for Berlin, Germany:

**Configuration:**
- **Country:** `de`
- **Zip:** `10115`

---

### Example 4: Spain Postal Code (Barcelona)

Retrieve municipality metadata for Barcelona, Spain:

**Configuration:**
- **Country:** `es`
- **Zip:** `08001`

---

### Example 5: Canada Postal Code (Toronto)

Look up location information for a Canadian postal code area:

**Configuration:**
- **Country:** `ca`
- **Zip:** `M5V`

---

### Example 6: Dynamic Postal Code Resolution from Upstream Node

Receive address data dynamically from an upstream webhook or function:

**Configuration:**
- **Country:** `{{$json.country}}`
- **Zip:** `{{$json.zip}}`

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Lookup geographic location and coordinates using Zippopotam.us
```

### Common Architecture Patterns

- **Address Autofill Pipeline:** Form Submission Trigger → Zippopotam.us (`country: input.country`, `zip: input.postalCode`) → Update CRM Record (Set `city: places[0].place_name`, `state: places[0].state`).
- **Geo-Distance & Routing:** Webhook (Order Created) → Zippopotam.us → Function (Calculate distance to nearest fulfillment warehouse via latitude/longitude) → Assign Carrier.
- **Local Weather Integration:** User Chat Trigger → Zippopotam.us → Weather API (Pass `latitude` and `longitude`) → Reply with Local Forecast.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### `Zippopotam.us request failed: Postal code not found`
- **Cause:** The postal code does not exist in the specified country's postal registry, or the country code is unsupported.
- **Solution:** Verify that the two-letter country code is valid (e.g. `us`, `ca`, `fr`, `de`, `es`, `it`). Ensure the postal code format matches the country's postal standard.

#### `Country code and zip code are required`
- **Cause:** One or both parameters (`country`, `zip`) were left blank or evaluated to an empty string.
- **Solution:** Provide both the two-letter country code and the postal code in the node configuration or expression.

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Postal code not found` | The postal code does not exist in the database (HTTP 404) | Double check postal code spelling and country code |
| `Country code and zip code are required` | Required parameter is missing | Ensure both `country` and `zip` parameters are supplied |
| `Zippopotam.us API error: 500` | Temporary upstream API server issue | Retry the workflow request |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [IP Geolocation](../ip-geolocation/en.md) — Identify user location and timezone from IP address
- [HTTP Request](../http-request/en.md) — Make custom API calls to external geocoding providers
- [Function](../function/en.md) — Transform and manipulate address coordinates
- [Webhook](../webhook/en.md) — Receive incoming postal code requests from web forms

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-26 | Initial release of Zippopotam.us Geocoding Action Node |

<!-- /SECTION: changelog -->
