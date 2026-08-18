---
node_id: "geo-names"
title: "GeoNames"
description: "Search for geographical places, cities, and locations using the GeoNames API."
category: "Web Search & Information"
subcategory: "Maps & Geospatial"
version: "1.0.0"
language: "en"
last_updated: "2026-08-18"
author: "Fusion Team"
tags:
  - geonames
  - geography
  - locations
  - cities
  - places
  - geocoding
  - geospatial
related_nodes:
  - google-maps
  - open-street-map
  - data-gouv-fr
  - ip-geolocation
---

<!-- SECTION: header -->
# GeoNames

> **Category:** Web Search & Information | **Type:** Action Node

Search for geographical places, cities, and locations using the GeoNames API for location enrichment, geocoding, and geospatial workflows.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GeoNames** node queries the GeoNames API to search for geographical places, cities, regions, and landmarks by name. It is useful for workflows that need to identify or validate geographical locations, enrich address data with coordinates, or build location-aware applications.

### Key Features

- **Place Search:** Search for cities, places, and geographical features by name
- **Coordinate Retrieval:** Get latitude, longitude, and elevation for locations
- **Administrative Information:** Retrieve country, state, and region data
- **Population Data:** Access population statistics where available
- **Structured Output:** Return machine-readable JSON with location details
- **Flexible Queries:** Support multiple search parameters and filters

### Typical Use Cases

- Search for cities or places by name
- Validate geographical locations
- Retrieve coordinates for mapping applications
- Enrich customer or business records with location data
- Build location lookup workflows
- Support geospatial analysis and mapping
- Integrate location data into dashboards and reports

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `q` | `string` | ✅ Yes | — | Search query for the geographical place or city name |
| `username` | `string` | ✅ Yes | — | GeoNames API username (required for authentication) |
| `featureClass` | `string` | ❌ No | — | Feature class filter (e.g., `A` for administrative regions, `P` for populated places) |
| `featureCode` | `string` | ❌ No | — | Specific feature code filter (e.g., `PPLC` for capital cities) |
| `countryCode` | `string` | ❌ No | — | ISO country code to limit search to a specific country |
| `maxRows` | `number` | ❌ No | `10` | Maximum number of results to return |
| `lang` | `string` | ❌ No | `en` | Language code for returned results |
| `type` | `enum` | ❌ No | `ALL` | Result type filter: `ALL`, `EQUALS`, or `CONTAINS` |

### Feature Class Reference

| Code | Description |
|------|-------------|
| `A` | Administrative regions |
| `H` | Water bodies |
| `L` | Landscape |
| `P` | Populated places |
| `R` | Road/railway |
| `S` | Sites (man-made structures) |
| `T` | Terrain features |
| `U` | Undersea features |
| `V` | Forests and vegetation |

### Example

```text
q: "Casablanca"
username: "your-geonames-username"
maxRows: 10
lang: "en"
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `string` or `object` | Location name to search, or an object with search parameters like `q`, `countryCode`, or other filters |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | GeoNames API response with matching places and location data |
| `error` | `object` | Error details when the request fails or validation fails |

### Success Response Example

```json
{
  "geonames": [
    {
      "adminCode1": "09",
      "lng": -7.9898,
      "geonameId": 2521265,
      "toponymName": "Casablanca",
      "countryId": 2988725,
      "fcCode": "PPLA",
      "name": "Casablanca",
      "fclName": "city, village",
      "adminCodes1": {
        "ISO3166_2": "09"
      },
      "countryCode": "MA",
      "lat": 33.5731,
      "fcode": "PPLA",
      "continentCode": "AF",
      "adminName1": "Casablanca",
      "countryName": "Morocco",
      "fclName": "city, village",
      "population": 3359818
    }
  ],
  "totalResultsCount": 1
}
```

### Error Response Example

```json
{
  "success": false,
  "error": "GeoNames API request failed or no results found",
  "query": "Casablanca"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example: Search for a City

```text
q: "Paris"
username: "your-geonames-username"
maxRows: 5
lang: "en"
```

**Result:**

```json
{
  "geonames": [
    {
      "name": "Paris",
      "lat": 48.8566,
      "lng": 2.3522,
      "countryCode": "FR",
      "countryName": "France",
      "population": 2161000
    }
  ]
}
```

### Example: Search by Country

```text
q: "Madrid"
username: "your-geonames-username"
countryCode: "ES"
maxRows: 1
```

### Example: Location Enrichment Workflow

Use the node to enrich customer or business records with geographical coordinates and location metadata, then pass results to mapping or analysis steps.

<!-- /SECTION: examples -->

<!-- SECTION: security -->
## Security

Store GeoNames API credentials in Fusion's credential system. Do not place your username or API key directly in workflow parameters or exported examples.
<!-- /SECTION: security -->
