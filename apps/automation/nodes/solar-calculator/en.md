---
node_id: "solar-calculator"
title: "Solar Calculator"
description: "Calculate sunrise, sunset, and solar noon times for any location and date"
category: "Mathematical & Statistical Analysis"
subcategory: "Calculators & Models"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - solar
  - sun
  - sunrise
  - sunset
  - solar-noon
  - astronomy
  - daylight
  - twilight
  - coordinates
  - geolocation
  - calculator
related_nodes:
  - ip-geolocation
  - open-meteo-climate
  - weather-search
  - date-calculator
  - function
  - log
---

<!-- SECTION: header -->
# Solar Calculator

> **Category:** Mathematical & Statistical Analysis | **Subcategory:** Calculators & Models | **Type:** Action Node

Calculate astronomical solar events—including **sunrise**, **sunset**, **solar noon**, **daylight duration**, and **twilight phases**—for any geographic location on Earth and any given date.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Solar Calculator** node performs precise astronomical calculations based on standard solar position algorithms. Given geographic coordinates (`latitude` and `longitude`) and an optional target `date`, it determines the exact solar cycle timings, day length, and twilight boundaries in UTC / ISO 8601 format.

This node is ideal for automating outdoor smart lighting, agricultural irrigation schedules, solar panel efficiency modeling, photography golden hour planning, maritime navigation, and dynamic scheduled routines linked to natural day/night cycles.

### Key Features

- **Accurate Solar Event Timings:** Computes exact timestamps for sunrise, sunset, and solar noon (meridian transit).
- **Comprehensive Twilight Phases:** Calculates civil, nautical, and astronomical twilight start and end times.
- **Day Length & Duration Metrics:** Provides total daylight hours and duration in seconds and formatted time strings (`HH:MM:SS`).
- **Global Coordinate Support:** Works for any location worldwide across Northern and Southern hemispheres (`-90°` to `+90°` latitude, `-180°` to `+180°` longitude).
- **Polar Day & Night Handling:** Gracefully handles high-latitude polar conditions (midnight sun and polar night).
- **Dynamic Expression Ready:** Bind geographic coordinates and dates directly from upstream geolocation, weather, or CRM nodes using `{{outputs.NodeName.success.field}}`.

### Calculation Flow

```text
Geographic Coordinates (latitude, longitude) + Target Date (YYYY-MM-DD)
                                  ↓
                  Compute Solar Declination & Equation of Time
                                  ↓
        ┌─────────────────────────┼─────────────────────────┐
        ↓                         ↓                         ↓
Astronomical Events:        Daylight Duration:       Twilight Phases:
- Sunrise                   - Total Seconds          - Civil (6° below)
- Solar Noon                - Formatted (HH:MM:SS)   - Nautical (12° below)
- Sunset                    - Day / Night Ratio      - Astronomical (18° below)
        └─────────────────────────┼─────────────────────────┘
                                  ↓
               Structured JSON Output (ISO 8601 UTC Timestamps)
```

### Use Cases

- **Smart Home & IoT Automation:** Automatically trigger street lights, outdoor garden illumination, or window blinds at civil twilight or sunset without static clock schedules.
- **Solar Energy & Photovoltaic Modeling:** Estimate active sunlight generation windows and peak irradiance hours for solar installations.
- **Smart Agriculture & Irrigation:** Schedule greenhouse shading and drip irrigation around solar noon to minimize water evaporation.
- **Photography & Drone Operations:** Plan outdoor shoots and golden/blue hour sessions by calculating exact twilight intervals.
- **Maritime & Aviation Operations:** Ensure operations comply with daylight flight rules (VFR) and nautical visibility windows.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `latitude` | `number` | ✅ Yes | — | Latitude coordinate in decimal degrees. Values range from `-90.0` (South Pole) to `+90.0` (North Pole). |
| `longitude` | `number` | ✅ Yes | — | Longitude coordinate in decimal degrees. Values range from `-180.0` (West) to `+180.0` (East). |
| `date` | `string` | ❌ No | *(Current Date)* | The target date for calculations in `YYYY-MM-DD` format (e.g., `2026-06-21`). Defaults to the current date if omitted. |

---

### Parameter Details

#### `latitude`
The North-South angular position of the target location in decimal degrees.
- **Type:** `number`
- **Required:** Yes
- **Valid Range:** `-90.0` to `90.0`
- **Positive Values:** Northern Hemisphere (e.g., `33.5731` for Casablanca, `48.8566` for Paris, `40.7128` for New York)
- **Negative Values:** Southern Hemisphere (e.g., `-33.8688` for Sydney, `-22.9068` for Rio de Janeiro)
- **Example:** `33.5731`, `-33.8688`

#### `longitude`
The East-West angular position of the target location in decimal degrees relative to the Prime Meridian.
- **Type:** `number`
- **Required:** Yes
- **Valid Range:** `-180.0` to `180.0`
- **Positive Values:** Eastern Hemisphere (e.g., `151.2093` for Sydney, `2.3522` for Paris, `139.6917` for Tokyo)
- **Negative Values:** Western Hemisphere (e.g., `-7.5898` for Casablanca, `-74.0060` for New York, `-122.4194` for San Francisco)
- **Example:** `151.2093`, `-7.5898`

#### `date`
The specific calendar date for which solar times are calculated.
- **Type:** `string`
- **Format:** `YYYY-MM-DD` (ISO 8601 calendar date)
- **Default:** Current system date at execution time
- **Example Values:** `"2026-06-21"`, `"2026-12-21"`, `"2026-03-20"`

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Upstream workflow execution payload. Can supply dynamic coordinates and dates via expressions (e.g., `{{outputs.IPGeolocation.success.latitude}}`). |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Emitted when calculations succeed. Contains ISO 8601 timestamps for all solar events and day length metrics. |
| `error` | `Error` | Emitted if coordinates are out of bounds or the date format is invalid. |

---

### Output Data Structure

When calculating for Sydney, Australia (`latitude: -33.8688`, `longitude: 151.2093`, `date: "2026-06-21"`):

```json
{
  "date": "2026-06-21",
  "latitude": -33.8688,
  "longitude": 151.2093,
  "sunrise": "2026-06-20T20:59:42.000Z",
  "sunset": "2026-06-21T06:54:18.000Z",
  "solarNoon": "2026-06-21T01:57:00.000Z",
  "dayLength": {
    "seconds": 35676,
    "hours": 9.91,
    "formatted": "09:54:36"
  },
  "twilight": {
    "civil": {
      "begin": "2026-06-20T20:32:15.000Z",
      "end": "2026-06-21T07:21:45.000Z"
    },
    "nautical": {
      "begin": "2026-06-20T20:00:48.000Z",
      "end": "2026-06-21T07:53:12.000Z"
    },
    "astronomical": {
      "begin": "2026-06-20T19:30:10.000Z",
      "end": "2026-06-21T08:23:50.000Z"
    }
  }
}
```

---

### Output Fields Reference

| Field | Type | Description |
|-------|------|-------------|
| `date` | `string` | The target calculation date (`YYYY-MM-DD`). |
| `latitude` | `number` | Input latitude evaluated in the calculation. |
| `longitude` | `number` | Input longitude evaluated in the calculation. |
| `sunrise` | `string` | UTC ISO 8601 timestamp when the upper edge of the Sun appears on the horizon. |
| `sunset` | `string` | UTC ISO 8601 timestamp when the upper edge of the Sun disappears below the horizon. |
| `solarNoon` | `string` | UTC ISO 8601 timestamp when the Sun reaches its highest elevation in the sky (transit). |
| `dayLength.seconds` | `number` | Total daylight duration between sunrise and sunset in seconds. |
| `dayLength.hours` | `number` | Total daylight duration in fractional hours. |
| `dayLength.formatted` | `string` | Daylight duration formatted as `HH:MM:SS`. |
| `twilight.civil` | `object` | Civil twilight interval (Sun between 0° and 6° below the horizon; natural light is sufficient for outdoor activities). |
| `twilight.nautical` | `object` | Nautical twilight interval (Sun between 6° and 12° below the horizon; horizon is still visible at sea). |
| `twilight.astronomical` | `object` | Astronomical twilight interval (Sun between 12° and 18° below the horizon; sky is fully dark for astronomy). |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Summer Solstice Calculation (Sydney)

Calculate shortest day of the year in the Southern Hemisphere.

**Parameter Configuration:**

```text
Latitude: -33.8688
Longitude: 151.2093
Date: 2026-06-21
```

**Results:**
- **Sunrise:** `20:59:42 UTC` (~`06:59` AEST)
- **Solar Noon:** `01:57:00 UTC` (~`11:57` AEST)
- **Sunset:** `06:54:18 UTC` (~`16:54` AEST)
- **Day Length:** `09:54:36` (~9.9 hours of daylight)

---

### Example 2: Dynamic Geolocation Solar Automation

Trigger outdoor lighting dynamically based on the user's detected IP address or site coordinates.

**Workflow Pipeline:**

```text
Manual Trigger
  → IP Geolocation (Fetch current site latitude & longitude)
  → Solar Calculator (latitude: {{outputs.IPGeolocation.success.latitude}}, longitude: {{outputs.IPGeolocation.success.longitude}})
  → Function (Extract civil twilight end time)
  → MQTT / Home Assistant Action (Schedule outdoor lights ON)
  → Log (Display schedule)
```

---

### Example 3: Agricultural Irrigation Window Optimization

Determine solar noon to schedule greenhouse misting during peak heat.

**Workflow Pipeline:**

```text
Cron Trigger (Daily at 05:00 AM)
  → Solar Calculator (latitude: 33.5731, longitude: -7.5898)
  → Function (Set irrigation timer for solarNoon + 1 hour)
  → HTTP Request (Trigger smart valve controller)
  → Log (Confirm automated scheduling)
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Solar Calculator Location Example Workflow
```

### How it flows

1. **Manual Trigger:** Starts the calculation workflow.
2. **Solar Calculator:** Takes coordinates (`latitude: -33.8688`, `longitude: 151.2093`, `date: "2026-06-21"`) and computes the exact sunrise, solar noon, and sunset timestamps.
3. **Log Node:** Displays the structured astronomical event times in the execution console.

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: best-practices -->
## Best Practices

1. **UTC vs Local Timezones:** The node always returns timestamps in **UTC (Zulu time)**. Use standard timezone conversion nodes (such as `format-date` or `convert-timezone`) if displaying times to end-users in local clock time.
2. **Atmospheric Refraction:** Timestamps incorporate standard atmospheric refraction corrections (`34 arcminutes`) for standard sea-level horizon visibility.
3. **Polar Latitude Handling:** At latitudes near the poles (`|lat| > 66.5°`), check for continuous daylight (*midnight sun*) or continuous darkness (*polar night*) during summer/winter solstices where sunrise or sunset may not occur.
4. **Coordinate Accuracy:** For standard municipal scheduling, 4 decimal places (e.g. `33.5731`) provides accuracy within ~11 meters, which is more than sufficient for solar angle calculations.

<!-- /SECTION: best-practices -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Latitude or Longitude Out of Range
- **Symptom:** Workflow fails with validation error.
- **Cause:** Passing latitudes outside `[-90, 90]` or longitudes outside `[-180, 180]` (e.g., swapping latitude and longitude values).
- **Solution:** Ensure `latitude` is between `-90` and `90` and `longitude` is between `-180` and `180`.

#### Incorrect Date Format
- **Symptom:** Node returns `NaN` or invalid timestamp strings.
- **Cause:** Supplying dates formatted as `DD/MM/YYYY` or text like `"yesterday"`.
- **Solution:** Always supply the standard `YYYY-MM-DD` ISO format (e.g. `"2026-06-21"`).

---

### Error Reference

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `Latitude must be between -90 and 90` | Latitude value exceeds boundary | Verify decimal degree coordinate |
| `Longitude must be between -180 and 180` | Longitude value exceeds boundary | Verify decimal degree coordinate |
| `Invalid date format` | Date string does not match `YYYY-MM-DD` | Supply date in standard ISO `YYYY-MM-DD` format |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related Nodes

- [IP Geolocation](../ip-geolocation/en.md) — Lookup geographic coordinates from IP addresses
- [Open-Meteo Climate](../open-meteo-climate/en.md) — Retrieve solar radiation and climate metrics
- [Weather Search](../weather-search/en.md) — Access real-time weather and cloud coverage conditions
- [Date Calculator](../date-calculator/en.md) — Perform date arithmetic and duration conversions
- [Format Date](../format-date/en.md) — Convert UTC timestamps to localized time formats
- [Log](../log/en.md) — Print astronomical outputs to the execution console

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-18 | Initial release with support for sunrise, sunset, solar noon, daylight duration, and civil/nautical/astronomical twilight calculations |

<!-- /SECTION: changelog -->
