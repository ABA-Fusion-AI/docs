---
node_id: "weather-search"
title: "Weather Search"
description: "Get current weather data from multiple sources (OpenWeatherMap, WeatherAPI, Open-Meteo)"
category: "Weather"
version: "1.0.0"
language: "en"
last_updated: "2026-08-27"
author: "Fusion Team"
tags:

- weather
- openweathermap
- weatherapi
- open-meteo
- forecast
- climate
- api

related_nodes:
- function
- if
- http-request

---

# Weather Search

> **Category:** weather-nodes | **Type:** Action Node

Get **current weather data** from any of three configurable providers — **OpenWeatherMap**, **WeatherAPI.com**, or **Open-Meteo** — normalized into a single common output shape.

The **Weather Search** node lets the caller pick a `source`, accepts location either as a city name (optionally with country) or as latitude/longitude coordinates, and returns a provider-agnostic `WeatherData` object regardless of which source was used.

### Supported Features

- Three selectable weather data sources, one requiring no API key (Open-Meteo)
- Location input by city name (+ optional ISO country code) or by coordinates
- Normalized output shape across all three providers
- Randomized `User-Agent` rotation and randomized request delays (anti-throttling/bot-detection friendliness)
- Automatic retry with exponential backoff on retryable errors (timeouts, DNS failures, 5xx, 429)
- Detailed, status-code-specific error messages
- WMO weather-code-to-description mapping for Open-Meteo (which returns numeric codes, not text)

---

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
| --------- | ---- | -------- | ------- | ----------- |
| `source` | `enum` | ✅ Yes | — | Weather data source: `openweathermap`, `weatherapi`, or `openmeteo`. |
| `city` | `string` | ✅ Yes (if no coordinates) | — | City name to look up. |
| `country` | `string` | ❌ No | — | ISO 3166-1 alpha-2 country code (exactly 2 characters), used alongside `city` to disambiguate. |
| `latitude` | `number` | ✅ Yes (if no city, and always for `openmeteo`) | — | Latitude, between -90 and 90. |
| `longitude` | `number` | ✅ Yes (if no city, and always for `openmeteo`) | — | Longitude, between -180 and 180. |
| `apiKey` | `string` | ✅ Yes (for `openweathermap`, `weatherapi`) | — | API key for the selected source. Not required for `openmeteo`. |

---

## Source Comparison

| Source | Requires API Key | Location Input | Notes |
| ------ | ------------------ | ---------------- | ----- |
| `openweathermap` | ✅ Yes | City (+country) or lat/lon | Comprehensive current-weather data. |
| `weatherapi` | ✅ Yes | City (+country) or lat/lon | Real-time weather via WeatherAPI.com. |
| `openmeteo` | ❌ No | **Coordinates only** — `city`/`country` are not sent to the API | Free, no key required; provides fewer current-condition fields (no humidity/pressure/feels-like in the basic `current_weather` response used here). |

**Important:** `openmeteo` requires `latitude`/`longitude` specifically — a `city` name alone is **not** resolved to coordinates for this source, since Open-Meteo's `current_weather` endpoint takes coordinates only. If only `city` is provided with `source: "openmeteo"`, the node throws.

---

## Location Resolution

The node accepts location in either form, validated **before** checking which source-specific fields are actually required:

```text
hasCoordinates = latitude !== undefined && longitude !== undefined
hasCity = city is set and non-blank
```

At least one of `hasCoordinates` or `hasCity` must be true, or the node throws immediately — before even checking the API key. Note that this initial check does **not** account for Open-Meteo's coordinates-only requirement; that is enforced separately inside `buildOpenMeteoUrl`.

---

## Request Resilience

Every request goes through a shared `fetchWeather` helper with the following behavior:

- **Pre-request delay**: a random 300–1000ms delay before the first attempt (throttling/bot-detection avoidance).
- **Post-success delay**: a random 100–400ms delay after a successful response.
- **Randomized `User-Agent`**: chosen randomly per request from a small pool of realistic browser strings.
- **Timeout**: 30 seconds by default (`DEFAULT_CONFIG.timeout`).
- **Retries**: up to 3 attempts by default (`DEFAULT_CONFIG.maxRetries`), with exponential backoff (`retryDelay * 2^attempt`, starting at 1000ms).

### Retryable vs Non-Retryable Errors

| Condition | Retried? |
| --------- | -------- |
| Timeout (`ECONNABORTED` or message contains "timeout") | ✅ Yes |
| DNS failure (`ENOTFOUND`) / connection refused (`ECONNREFUSED`) | ✅ Yes |
| HTTP 5xx | ✅ Yes |
| HTTP 429 (rate limited) | ✅ Yes |
| HTTP 4xx other than 429 (400, 401, 403, 404) | ❌ No — thrown immediately |

---

## Status-Code-Specific Error Messages

| HTTP Status | Message |
| ----------- | ------- |
| 400 | `Bad Request: Invalid location or parameters` |
| 401 | `Unauthorized: Invalid or missing API key` |
| 403 | `Forbidden: API key invalid or quota exceeded` |
| 404 | `Not Found: Location not found` |
| 429 | `Rate Limited: Too many requests. Retrying...` |
| 500 / 502 / 503 | `Server Error: Weather service is experiencing issues` |
| Other | `HTTP <status>: <statusText>` |

---

## Inputs & Outputs

### Inputs

The node does not require workflow input. All configuration is provided through the node configuration.

### Outputs

A normalized `WeatherData` object, the same shape regardless of `source`, plus a `source_description` field:

| Output | Type | Description |
| ------ | ---- | ----------- |
| `source` | `string` | Which provider produced this data (`"openweathermap"`, `"weatherapi"`, or `"openmeteo"`). |
| `location.name` | `string` | Location name. For `openmeteo`, this is echoed from the `city` config value (or `"Unknown"` if not provided), **not** resolved by the API. |
| `location.country` | `string` | Country. Same `openmeteo` caveat as `location.name`. |
| `location.latitude` | `number` | Latitude. |
| `location.longitude` | `number` | Longitude. |
| `current.temperature` | `number` | Current temperature, in Celsius. |
| `current.feels_like` | `number` | Feels-like temperature. For `openmeteo`, this is just a copy of `temperature` — the API doesn't provide a true feels-like value in this response. |
| `current.humidity` | `number` | Relative humidity (%). Always `0` for `openmeteo` — not available in the basic `current_weather` response used here. |
| `current.pressure` | `number` | Atmospheric pressure. Always `0` for `openmeteo`, same reason. |
| `current.wind_speed` | `number` | Wind speed. |
| `current.wind_direction` | `number` | Wind direction in degrees. |
| `current.description` | `string` | Text weather description. For `openmeteo`, mapped from a numeric WMO weather code. |
| `current.icon` | `string \| undefined` | Icon code/URL, when available. Always `undefined` for `openmeteo`. |
| `timestamp` | `string` | ISO timestamp (or provider-local time string for `weatherapi`). |
| `source_description` | `string` | Human-readable description of the source, from the node's internal source registry. |

---

## Output Example

### `openweathermap`

```json
{
  "source": "openweathermap",
  "location": { "name": "Rabat", "country": "MA", "latitude": 34.02, "longitude": -6.83 },
  "current": {
    "temperature": 27.4,
    "feels_like": 27.1,
    "humidity": 58,
    "pressure": 1014,
    "wind_speed": 3.6,
    "wind_direction": 210,
    "description": "clear sky",
    "icon": "01d"
  },
  "timestamp": "2026-08-27T12:00:00.000Z",
  "source_description": "OpenWeatherMap - Comprehensive weather data with API key"
}
```

### `openmeteo` (reduced field set)

```json
{
  "source": "openmeteo",
  "location": { "name": "Unknown", "country": "Unknown", "latitude": 34.02, "longitude": -6.83 },
  "current": {
    "temperature": 26.8,
    "feels_like": 26.8,
    "humidity": 0,
    "pressure": 0,
    "wind_speed": 12.4,
    "wind_direction": 200,
    "description": "Clear sky",
    "icon": null
  },
  "timestamp": "2026-08-27T12:00",
  "source_description": "Open-Meteo - Free weather API, no API key required"
}
```

---

## Configuration Examples

### OpenWeatherMap by City

```json
{
  "source": "openweathermap",
  "city": "Rabat",
  "country": "MA",
  "apiKey": "your-openweathermap-key"
}
```

### WeatherAPI by Coordinates

```json
{
  "source": "weatherapi",
  "latitude": 34.02,
  "longitude": -6.83,
  "apiKey": "your-weatherapi-key"
}
```

### Open-Meteo (No API Key, Coordinates Required)

```json
{
  "source": "openmeteo",
  "latitude": 34.02,
  "longitude": -6.83
}
```

---

## Workflow Integration

### Sample Workflow: Weather → Function

```json
{
  "nodes": [
    {
      "id": "weather-search",
      "type": "weather-search",
      "config": {
        "source": "openmeteo",
        "latitude": 34.02,
        "longitude": -6.83
      }
    },
    {
      "id": "format-weather-message",
      "type": "function"
    }
  ]
}
```

### Sample Workflow: Weather → If → Notification

```json
{
  "nodes": [
    {
      "id": "weather-search",
      "type": "weather-search",
      "config": {
        "source": "openweathermap",
        "city": "Casablanca",
        "country": "MA",
        "apiKey": "your-openweathermap-key"
      }
    },
    {
      "id": "check-rain",
      "type": "if"
    },
    {
      "id": "notify-team",
      "type": "notification"
    }
  ]
}
```

### Sample Workflow: Schedule → Weather → Database

```json
{
  "nodes": [
    {
      "id": "weather-search",
      "type": "weather-search",
      "config": {
        "source": "openmeteo",
        "latitude": 34.02,
        "longitude": -6.83
      }
    },
    {
      "id": "log-weather",
      "type": "database"
    }
  ]
}
```

### Common Patterns

- Schedule (hourly) → Weather Search → Database — historical weather logging
- Weather Search → If (temperature/condition threshold) → Notification — weather alerting
- Weather Search → Function → Dashboard/report rendering

---

## Error Handling

### Missing Location

```text
Either city name or latitude/longitude coordinates are required
```

Raised when neither a non-blank `city` nor both `latitude` and `longitude` are provided.

### Missing API Key

```text
<source> requires an API key. Please provide one in the apiKey parameter.
```

Raised when the selected `source` requires a key (`openweathermap`, `weatherapi`) and `apiKey` is empty.

### Source-Specific Build Errors

```text
Either city or lat/lon coordinates are required
```

Raised inside `buildOpenWeatherMapUrl`/`buildWeatherAPIUrl` if, unexpectedly, neither location form is present at URL-build time (defense-in-depth beyond the earlier check).

```text
Open-Meteo requires latitude and longitude coordinates
```

Raised inside `buildOpenMeteoUrl` when `latitude`/`longitude` are missing — this is the practical enforcement of the "coordinates only" rule for `openmeteo`, since the earlier general check would otherwise accept a `city`-only config.

### Status-Specific Errors

See [Status-Code-Specific Error Messages](#status-code-specific-error-messages) above — these propagate up wrapped as:

```text
Failed to fetch weather from <source>: <message>
```

### Retry Exhaustion

```text
Failed to fetch weather from <source>: <last retryable error message>
```

Raised after all retry attempts (default 3) are exhausted for a retryable error (timeout, DNS/connection failure, 5xx, or 429).

---

## Troubleshooting

### "Either city name or latitude/longitude coordinates are required"

**Cause**

Neither `city` nor both `latitude`/`longitude` were provided.

**Solution**

Provide at least one location form. For `openmeteo`, provide coordinates specifically — a city name alone will fail later at URL-build time.

---

### "Failed to fetch weather from openmeteo: Open-Meteo requires latitude and longitude coordinates"

**Cause**

`source` is `openmeteo` but only `city` (no coordinates) was provided — Open-Meteo's `current_weather` endpoint used by this node does not accept a city name.

**Solution**

Provide `latitude`/`longitude` explicitly, or geocode the city name to coordinates first (e.g. via a separate geocoding API call in an `HTTP Request` node) before using `openmeteo`.

---

### "<source> requires an API key. ..."

**Cause**

`source` is `openweathermap` or `weatherapi` and `apiKey` is empty.

**Solution**

Provide a valid API key for the selected source, or switch to `openmeteo` if no key is available.

---

### "Failed to fetch weather from <source>: Unauthorized: Invalid or missing API key"

**Cause**

The provided `apiKey` was rejected by the provider — invalid, revoked, or not yet activated (some providers take time to activate new keys).

**Solution**

Verify the key on the provider's dashboard; for OpenWeatherMap specifically, note that new keys can take up to a couple of hours to activate.

---

### "Failed to fetch weather from <source>: Not Found: Location not found"

**Cause**

The `city` name (with or without `country`) did not match any known location for the selected provider.

**Solution**

Try a more specific city name, add the `country` code to disambiguate, or switch to coordinate-based lookup.

---

### `humidity`/`pressure` Are Always `0` for Open-Meteo

**Cause**

This is expected — the node only requests Open-Meteo's basic `current_weather=true` parameter, which does not include humidity or pressure. A richer Open-Meteo request (with `hourly`/`current` parameter variants) would be needed to get these fields, which this node does not currently request.

**Solution**

If humidity/pressure are required, use `openweathermap` or `weatherapi` instead, both of which include these fields.

---

### Requests Feel Slow

**Cause**

By design, the node adds randomized delays (300–1000ms before the request, 100–400ms after success) as a deliberate anti-throttling measure, on top of any retry backoff.

**Solution**

This is expected behavior for this node and not a fault; if latency is unacceptable for a given use case, a direct `HTTP Request` node without these delays could be used instead (at the cost of losing the built-in retry/backoff handling).

---

## Security

The node performs outbound HTTPS requests to the selected provider's public API (`api.openweathermap.org`, `api.weatherapi.com`, or `api.open-meteo.com`).

API keys (`apiKey`) are sent as URL query parameters, as required by each provider's API.

The node rotates a small, fixed set of realistic browser `User-Agent` strings and adds randomized delays between requests — this is aimed at avoiding basic bot-detection/throttling on the provider side, not at anonymizing the request in any stronger sense.

---

## Notes

The node normalizes output across three fundamentally different provider APIs — some fields are inherently unavailable for a given source (`humidity`/`pressure`/`icon` for `openmeteo`) and are filled with `0`/`undefined` rather than omitted, to keep the output shape consistent.

The node does not:

- Geocode a city name into coordinates for `openmeteo` — coordinates must be supplied directly for that source
- Support forecast data (only `current` conditions) despite the OpenWeatherMap/WeatherAPI/Open-Meteo APIs each supporting forecasts
- Cache weather results between calls
- Retry on non-retryable 4xx errors (400, 401, 403, 404) — these fail immediately
- Support additional Open-Meteo parameters (e.g. hourly data, additional current-condition fields) beyond the basic `current_weather=true` request

---

## Changelog

| Version | Date | Changes |
| ------- | ---- | ------- |
| 1.0.0 | 2026-08-27 | Initial release |