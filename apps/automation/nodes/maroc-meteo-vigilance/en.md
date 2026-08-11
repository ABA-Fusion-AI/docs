---
node_id: "maroc-meteo-vigilance"
title: "Maroc Meteo: Vigilance Map"
description: "Fetch weather vigilance data from Maroc Météo with option for GeoJSON or simplified JSON output."
category: "utilities"
subcategory: "weather"
version: "1.0.0"
language: "en"
last_updated: "2026-08-10"
author: "Fusion Team"
tags:
  - morocco
  - weather
  - vigilance
  - geojson
  - meteo
related_nodes:
  - http-request
  - geojson
  - log
  - function
---

<!-- SECTION: header -->
# Maroc Meteo: Vigilance Map

> **Category:** Utilities | **Type:** Action Node

Fetch weather vigilance data from Maroc Météo (Direction de la Météorologie Nationale) with support for GeoJSON or simplified JSON output.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Maroc Meteo: Vigilance Map** node retrieves official Moroccan weather vigilance data published by the Direction de la Météorologie Nationale (DMN).

It returns the current (or date-specific) vigilance bulletin, indicating alert levels and affected regions across Morocco. The output can be delivered as a full **GeoJSON** document for geospatial processing or as a **simplified JSON** object for lightweight automation.

### Key Features

- **Official Data Source:** Pulls vigilance data directly from Maroc Météo / DMN
- **Date Selection:** Fetch the bulletin for the current day or a specific past date
- **Dual Output Format:** Choose between GeoJSON (for mapping) or simplified JSON (for alerts and notifications)
- **Regional Coverage:** Covers all Moroccan regions with per-region alert levels
- **Alert Levels:** Returns standardized vigilance levels (green, yellow, orange, red)

### Use Cases

- Monitor weather alerts and trigger notifications (SMS, email, Teams)
- Feed vigilance data into a mapping dashboard
- Automate regional emergency responses based on vigilance level
- Archive daily vigilance bulletins for historical analysis
- Integrate weather risk data into logistics or field operations workflows

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type     | Required | Default   | Description |
|-----------|----------|----------|-----------|-------------|
| `date`    | `string` | No       | Today     | Target date for the vigilance bulletin in `D-M-YYYY` format (e.g., `7-8-2026`). Leave empty to fetch the current day's bulletin. |
| `format`  | `enum`   | Yes      | `geojson` | Output format for the vigilance data. Accepted values: `geojson` or `json`. |

### Format Options

| Value     | Description |
|-----------|-------------|
| `geojson` | Returns a GeoJSON `FeatureCollection` where each feature represents a Moroccan region with its vigilance level and metadata as properties. Compatible with mapping tools and the GeoJSON Tools node. |
| `json`    | Returns a simplified JSON array containing region names and their corresponding alert levels. Suitable for notification workflows and conditional logic. |

### Date Format

The `date` parameter uses the format `D-M-YYYY`:

```text
7-8-2026   → August 7, 2026
15-1-2026  → January 15, 2026
```

Leave the field empty to retrieve the **current day's** bulletin automatically.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input   | Type  | Description |
|---------|-------|-------------|
| `input` | `any` | Incoming workflow data passed from the preceding node. |

### Outputs

| Output    | Type     | Description |
|-----------|----------|-------------|
| `success` | `object` | The vigilance data in the selected format (`geojson` or `json`). |
| `error`   | `Error`  | Emitted when the request fails, the date is invalid, or the data source is unavailable. |

### GeoJSON Output Structure

When `format` is set to `geojson`, the output is a GeoJSON `FeatureCollection`:

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": ["..."]
      },
      "properties": {
        "region": "Souss-Massa",
        "level": "orange",
        "phenomena": "Thunderstorms",
        "date": "2026-08-07"
      }
    }
  ]
}
```

### Simplified JSON Output Structure

When `format` is set to `json`, the output contains a simplified representation:

```json
[
  {
    "region": "Souss-Massa",
    "level": "orange",
    "phenomena": "Thunderstorms",
    "date": "2026-08-07"
  },
  {
    "region": "Marrakech-Safi",
    "level": "yellow",
    "phenomena": "Wind",
    "date": "2026-08-07"
  }
]
```

### Vigilance Levels

| Level    | Meaning |
|----------|---------|
| `green`  | No particular vigilance required |
| `yellow` | Be attentive — potentially dangerous weather |
| `orange` | Be very vigilant — dangerous weather expected |
| `red`    | Absolute vigilance required — very dangerous weather |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Fetch Today's Vigilance Map (GeoJSON)

Retrieve the current day's bulletin as GeoJSON to render on a map.

**Configuration:**

```text
Date:   (empty — defaults to today)
Format: geojson
```

**Output:** A GeoJSON `FeatureCollection` with one feature per region, each carrying the vigilance level and phenomena as properties.

---

### Example 2: Fetch a Specific Date (Simplified JSON)

Retrieve the bulletin for August 7, 2026 as simplified JSON.

**Configuration:**

```text
Date:   7-8-2026
Format: json
```

**Output:**

```json
[
  {
    "region": "Souss-Massa",
    "level": "orange",
    "phenomena": "Thunderstorms",
    "date": "2026-08-07"
  }
]
```

---

### Example 3: Alert Notification Workflow

Trigger a Teams or email alert when any region reaches `orange` or `red` vigilance.

**Workflow pattern:**

```text
Manual Trigger
  → Maroc Meteo: Vigilance Map (format: json)
  → Function (filter regions where level is "orange" or "red")
  → Microsoft Teams / Email Send
```

---

### Example 4: Map Visualization

Render a live vigilance map using GeoJSON output.

**Workflow pattern:**

```text
Cron Trigger (every morning)
  → Maroc Meteo: Vigilance Map (format: geojson)
  → HTTP Request (POST to map rendering API)
  → Log
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Fetch Morocco weather vigilance data
```

### Common Patterns

- **Daily Briefing:** Cron Trigger → Maroc Meteo: Vigilance Map → Email Send
- **Alert Automation:** Maroc Meteo: Vigilance Map → Function (filter high levels) → Notification
- **Geospatial Processing:** Maroc Meteo: Vigilance Map (geojson) → GeoJSON Tools → Store
- **Historical Archive:** Cron Trigger → Maroc Meteo: Vigilance Map → Database Insert
- **Dashboard Feed:** Manual Trigger → Maroc Meteo: Vigilance Map → HTTP Request (dashboard API)

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### No data returned / empty response

**Cause:** The requested date may be in the future or the bulletin has not yet been published.

**Solution:** Use a past date or leave the `date` field empty to fetch today's bulletin.

#### Invalid date format

**Cause:** The `date` parameter was provided in an unsupported format (e.g., `2026-08-07` or `08/07/2026`).

**Solution:** Use the `D-M-YYYY` format without leading zeros for day and month:

```text
Correct:   7-8-2026
Incorrect: 07-08-2026
Incorrect: 2026-08-07
```

#### Network error / data source unavailable

**Cause:** The Maroc Météo data endpoint is temporarily unreachable.

**Solution:** Retry after a short delay. Add a retry mechanism using the `delay` node or workflow retry settings.

#### GeoJSON output not rendering correctly in mapping tools

**Cause:** The mapping tool may not support `MultiPolygon` geometries or requires a specific coordinate system.

**Solution:** Pre-process the output using the **GeoJSON Tools** node to simplify or transform the geometry as needed.

### Error Reference

| Error | Cause | Solution |
|-------|-------|----------|
| Network timeout | DMN endpoint unreachable | Retry or add error-path fallback |
| Invalid date | Unsupported `date` format | Use `D-M-YYYY` format |
| No bulletin found | Future date or unpublished data | Use today's date or a past date |
| Parse error | Unexpected response structure | Check the DMN data source for changes |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [GeoJSON Tools](../geojson/en.md) — Process, validate, and transform the GeoJSON output
- [HTTP Request](../http-request/en.md) — Make custom requests to Maroc Météo or other weather APIs
- [Function](../function/en.md) — Filter regions by vigilance level
- [Log](../log/en.md) — Inspect the raw vigilance output

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date       | Changes         |
|---------|------------|-----------------|
| 1.0.0   | 2026-08-10 | Initial release |

<!-- /SECTION: changelog -->
