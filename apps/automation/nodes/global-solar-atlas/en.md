---
node_id: "global-solar-atlas"
title: "Global Solar Atlas"
description: "Get solar energy potential and irradiation data for any location using the Global Solar Atlas API."
category: "web-search-information"
subcategory: "environment"
version: "1.0.0"
language: "en"
last_updated: "2026-08-24"
author: "Fusion Team"
tags:
  - solar-energy
  - renewable-energy
  - environment
  - photovoltaic
  - pvout
  - ghi
  - dni
related_nodes:
  - solar-calculator
  - sunrise-sunset
---

<!-- SECTION: header -->
# Global Solar Atlas

> **Category:** Web Search & Information | **Subcategory:** Environment | **Type:** Action Node

Retrieve solar energy potential and irradiation data for any geographic location using the Global Solar Atlas API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Global Solar Atlas** node queries solar resource data for a location identified by latitude and longitude.

It can provide photovoltaic potential and solar irradiation indicators such as PVOUT, GHI, and DNI, making the node useful for renewable-energy analysis, site assessment, and environmental workflows.

### Key Features

- **Location-Based Lookup:** Query solar data using latitude and longitude coordinates.
- **Photovoltaic Potential:** Retrieve PVOUT and related solar-energy indicators.
- **Solar Irradiation Data:** Access GHI, DNI, and other values returned by the service.
- **Worldwide Coverage:** Request data for locations around the world supported by Global Solar Atlas.
- **No Credential in Example:** The supplied workflow contains no API token, secret, or authentication parameter.
- **Workflow Ready:** Connect results to log, function, calculation, or reporting nodes.

### Use Cases

- Assess solar potential for a proposed site
- Compare renewable-energy opportunities by location
- Support photovoltaic installation planning
- Enrich environmental and geographic workflows
- Build solar resource reports
- Combine solar data with sunrise, weather, or electricity information

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `lat` | `number` | Yes | — | Latitude of the location in decimal degrees. Valid range is `-90` to `90`. |
| `lng` | `number` | Yes | — | Longitude of the location in decimal degrees. Valid range is `-180` to `180`. |

### Coordinate Examples

Example coordinates for a location in Morocco:

```json
{
  "lat": 33,
  "lng": -7
}
```

Coordinates must use decimal degrees. Positive latitude values represent locations north of the Equator, while negative values represent locations south of it. Positive longitude values represent locations east of the Prime Meridian, while negative values represent locations west of it.

The example workflow does not configure an API key, bearer token, password, or other secret. No credential field is exposed by the node configuration shown in the repository.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | unknown | Incoming workflow data. The configured `lat` and `lng` values determine the requested location. |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | object | Solar resource data returned for the requested coordinates. |
| `error` | object | Error details when coordinates are invalid, the request fails, or the service returns an error. |

### Successful Response

The response contains solar resource information for the requested location. Depending on the service response, fields can include photovoltaic power output (`PVOUT`), global horizontal irradiation (`GHI`), direct normal irradiation (`DNI`), and other solar-energy indicators.

Example response shape:

```json
{
  "lat": 33,
  "lng": -7,
  "PVOUT": 1760,
  "GHI": 1900,
  "DNI": 2200
}
```

The exact field names, units, and additional properties depend on the current Global Solar Atlas API response.

### Error Output

Errors are routed through the `error` output. They can result from coordinates outside the valid geographic ranges, missing values, network failures, service limits, or an unavailable API response.

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Query Solar Potential

```json
{
  "lat": 33,
  "lng": -7
}
```

### Example: Query a Southern Hemisphere Location

```json
{
  "lat": -33.8688,
  "lng": 151.2093
}
```

The node does not require an API token or secret in the configuration represented by the example workflow.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Retrieve solar potential for a location and inspect the result
```

### Common Patterns

- **Solar Assessment:** Manual Trigger → Global Solar Atlas → Log
- **Location Comparison:** Global Solar Atlas → Function → Log
- **Renewable Report:** Global Solar Atlas → Function → Document or Database
- **Environmental Analysis:** Global Solar Atlas → Solar Calculator → Log

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### Missing coordinates

**Cause:** `lat` or `lng` was not provided.

**Solution:** Configure both coordinates as decimal numbers.

#### Invalid latitude

**Cause:** The latitude is outside the range `-90` to `90` or is not numeric.

**Solution:** Use a valid decimal latitude.

#### Invalid longitude

**Cause:** The longitude is outside the range `-180` to `180` or is not numeric.

**Solution:** Use a valid decimal longitude.

#### Unexpected or incomplete response

**Cause:** Solar resource availability and returned metadata can vary by location and service response.

**Solution:** Check the requested coordinates and handle missing indicators in downstream nodes.

#### Request failed

**Cause:** The Global Solar Atlas service may be unavailable, rate-limited, or unreachable.

**Solution:** Check connectivity and retry the workflow later. The example does not use an API token or secret.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [Solar Calculator](../solar-calculator/en.md) - Calculate solar-position and solar-energy values
- [Sunrise Sunset](../sunrise-sunset/en.md) - Retrieve sunrise and sunset information

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-24 | Initial documentation |

<!-- /SECTION: changelog -->
