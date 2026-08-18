---
node_id: "emsc-seismic"
title: "EMSC Seismic Portal"
description: "Query earthquake and seismic activity data from the European-Mediterranean Seismological Centre (EMSC) API."
category: "Web Search & Information"
subcategory: "Environment"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - emsc
  - earthquake
  - seismic
  - environment
  - geology
  - natural-disaster
related_nodes:
  - usgs-earthquakes
  - ingv-seismic
  - open-data-search
---

<!-- SECTION: header -->
# EMSC Seismic Portal

> **Category:** Web Search & Information | **Type:** Action Node

Query earthquake data from the European-Mediterranean Seismological Centre (EMSC) for monitoring, analysis, and environmental alert workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **EMSC Seismic Portal** node retrieves seismic activity records and earthquake metadata from the EMSC API. It is useful for monitoring recent events, analyzing seismic trends, and enriching environmental or operational workflows with real-time event intelligence.

### Key Features

- **Earthquake Search:** Look up recent or historical seismic events
- **Regional Filtering:** Query by area, coordinates, or time window
- **Magnitude Filtering:** Focus on events above a specified magnitude threshold
- **Structured Output:** Return machine-readable JSON for analysis or storage
- **Operational Monitoring:** Use seismic data in alerting and monitoring pipelines

### Typical Use Cases

- Track earthquake activity in a region
- Build dashboards for natural hazard monitoring
- Enrich operational systems with seismic event data
- Support research or environmental observation workflows
- Monitor significant seismic events for alerting and reporting

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `query` | `string` | ❌ No | — | Keyword or text-based region search |
| `startTime` | `string` | ❌ No | — | Start date/time for the earthquake search window |
| `endTime` | `string` | ❌ No | — | End date/time for the earthquake search window |
| `minMagnitude` | `number` | ❌ No | — | Minimum earthquake magnitude to include |
| `maxMagnitude` | `number` | ❌ No | — | Maximum earthquake magnitude to include |
| `limit` | `number` | ❌ No | `10` | Maximum number of results to return |
| `latitude` | `number` | ❌ No | — | Latitude for localized earthquake queries |
| `longitude` | `number` | ❌ No | — | Longitude for localized earthquake queries |
| `radiusKm` | `number` | ❌ No | — | Search radius in kilometers around the provided coordinates |
| `filters` | `object` | ❌ No | — | Optional API filters for more specific result narrowing |

### Example

```text
startTime: "2026-08-01T00:00:00Z"
endTime: "2026-08-18T23:59:59Z"
minMagnitude: 4.5
limit: 20
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Search criteria such as a region, time range, or structured request data |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | Earthquake records returned by the EMSC API |
| `error` | `object` | Error data when the request fails or validation fails |

### Successful Response Example

```json
{
  "events": [
    {
      "id": "2026-08-17-1234",
      "time": "2026-08-17T10:42:18Z",
      "magnitude": 5.8,
      "location": "Aegean Sea",
      "depth": 18.2,
      "latitude": 38.1,
      "longitude": 23.4
    }
  ],
  "count": 1
}
```

### Error Response Example

```json
{
  "success": false,
  "error": "The EMSC request was invalid or no results were returned.",
  "query": "earthquakes"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Search Recent Earthquakes Above Magnitude 5

```text
startTime: "2026-08-01T00:00:00Z"
endTime: "2026-08-18T23:59:59Z"
minMagnitude: 5
limit: 10
```

**Result:**

```json
{
  "events": [
    {
      "time": "2026-08-17T10:42:18Z",
      "magnitude": 5.8,
      "location": "Aegean Sea"
    }
  ]
}
```

### Example: Regional Monitoring Workflow

Use the node to monitor seismic activity in a country, region, or risk zone and route results into alerts or dashboards.

<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store credentials in Fusion's credential system. Do not place secrets directly in workflow parameters or exported examples.
<!-- /SECTION: security -->
