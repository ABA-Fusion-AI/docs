---
node_id: "wttr"
title: "Wttr.in Weather"
description: "Get current conditions and weather forecasts for a city using the public wttr.in service."
category: "Web Search & Information"
subcategory: "Environment"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:
  - weather
  - forecast
  - wttr
  - climate
  - environment
  - city
  - public-api
related_nodes:
  - openweathermap
  - weather-search
  - global-solar-atlas
  - log
---

<!-- SECTION: header -->
# Wttr.in Weather

> **Category:** Web Search & Information | **Subcategory:** Environment | **Type:** Action Node

Get current weather conditions and forecast information for a city using the public wttr.in weather service.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Wttr.in Weather** node queries wttr.in for a city and returns machine-readable weather data. The service can provide current conditions, multi-day forecasts, location details, astronomy information, and hourly observations.

### Key Features

- **City Weather:** Look up weather using a city name such as `Casablanca`
- **Current Conditions:** Retrieve temperature, feels-like temperature, humidity, wind, visibility, and weather description when available
- **Forecast Data:** Return daily and hourly forecast information
- **JSON Response:** Use wttr.in's machine-readable JSON format for downstream workflow processing
- **Dynamic Input:** Use a configured `city` or provide a city through the `input` port
- **No Credentials Required:** The `wttr` node configuration contains no API key, access token, authorization header, or secret

### Use Cases

- Send weather-aware notifications
- Add local conditions to travel, logistics, or field-service workflows
- Trigger operational decisions based on forecast conditions
- Enrich dashboards and reports with current weather
- Combine weather information with environmental or solar-resource data

> Weather data is informational and may be delayed, incomplete, or unavailable. Do not use it as the sole basis for safety-critical decisions.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `city` | `string` | Yes | — | City or location to query, for example `Casablanca` |

If `city` is not configured, the node can read a direct string from the incoming `input` value. An incoming object can provide a `city` property.

### API Behavior

The node requests the JSON representation of wttr.in weather data:

```text
GET https://wttr.in/{city}?format=j1
```

The `city` value should be URL-encoded when it contains spaces or special characters. No API key or authentication token is required.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | A city name or an object containing a `city` field. Used when the configured `city` parameter is empty. |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Weather response returned by wttr.in for the requested city |

Typical JSON response fields include:

```json
{
  "current_condition": [
    {
      "temp_C": "22",
      "temp_F": "72",
      "FeelsLikeC": "22",
      "humidity": "45",
      "weatherDesc": [{ "value": "Partly cloudy" }],
      "windspeedKmph": "12"
    }
  ],
  "nearest_area": [
    {
      "areaName": [{ "value": "Casablanca" }],
      "country": [{ "value": "Morocco" }]
    }
  ],
  "weather": []
}
```

The exact fields and forecast entries depend on the requested location and the upstream response.

### Error Output

Missing or invalid cities, lookup failures, timeouts, and malformed upstream responses are routed to `error`.

```json
{
  "success": false,
  "error": "Weather lookup failed",
  "city": "Unknown City"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic City Lookup

```json
{
  "city": "Casablanca"
}
```

### City with Spaces

```json
{
  "city": "New York"
}
```

The node handles URL construction for the city value.

### Dynamic City from a Previous Node

Pass a city directly through `input`:

```text
Rabat
```

Or pass a named object:

```json
{
  "city": "Marrakesh"
}
```

### Extract Current Conditions

Use a Function node after the lookup to select a compact weather object:

```js
const current = input.current_condition?.[0] || {};
return {
  temperatureC: Number(current.temp_C),
  feelsLikeC: Number(current.FeelsLikeC),
  humidity: Number(current.humidity),
  description: current.weatherDesc?.[0]?.value || null
};
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example-workflow.json
title: Get weather data for a city
```

### Common Patterns

- **Basic lookup:** Manual Trigger → Wttr.in Weather → Log
- **Weather alert:** Schedule → Wttr.in Weather → Function → Notification
- **Logistics planning:** Location record → Wttr.in Weather → Conditional branch
- **Environmental context:** Wttr.in Weather → Global Solar Atlas → Report

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### City is required

**Cause:** Neither `city` nor a usable incoming `input` value was provided.

**Solution:** Configure a city name, pass a string to `input`, or pass an object with a `city` field.

### Location not found or ambiguous

**Cause:** The city name is misspelled, incomplete, or matches multiple locations.

**Solution:** Include a country or region when needed, such as `Paris, France` or `Springfield, Illinois`.

### Missing forecast fields

**Cause:** The upstream response varies by location and may omit optional fields.

**Solution:** Check that arrays such as `current_condition` and `weather` contain an item before reading nested values.

### Request failed or timed out

**Cause:** wttr.in is unavailable, rate-limited, or unreachable from the workflow runtime.

**Solution:** Retry with backoff, avoid sending bursts of requests, and inspect the `error` output.

### Unexpected temperature or numeric values

**Cause:** wttr.in commonly returns weather measurements as strings, and some fields may be empty.

**Solution:** Convert values explicitly in downstream logic and handle missing or non-numeric fields safely.

### Authentication question

**Cause:** The node appears to require an API credential.

**Solution:** No credential is configured or required for the public wttr.in service.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [OpenWeatherMap](./openweathermap.md) — Retrieve weather data using OpenWeatherMap
- [Weather Search](./weather-search.md) — Query weather from multiple providers
- [Global Solar Atlas](./global-solar-atlas.md) — Retrieve solar and irradiation data
- [Log](./log.md) — Inspect weather responses

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-27 | Initial documentation |

<!-- /SECTION: changelog -->
