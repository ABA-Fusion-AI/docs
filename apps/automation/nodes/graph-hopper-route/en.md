---
node_id: "graph-hopper-route"
title: "GraphHopper Route"
description: "Calculate a route between coordinate points using the GraphHopper Routing API."
category: "Web Search & Information"
subcategory: "Maps & Geospatial"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - graphhopper
  - routing
  - maps
  - geospatial
  - directions
  - coordinates
related_nodes:
  - geospatial-utils
  - geojson
  - http-request
---

<!-- SECTION: header -->
# GraphHopper Route

> **Category:** Web Search & Information | **Subcategory:** Maps & Geospatial | **Type:** Action Node

Calculate a route between two or more geographic coordinate points using the GraphHopper Routing API.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GraphHopper Route** node requests a route between coordinate points and returns routing information such as distance, travel time, geometry, and path details when provided by the API. It is suitable for driving, walking, cycling, and other supported routing profiles.

### Key Features

- **Point-to-Point Routing:** Calculate a route between two or more coordinates
- **Multiple Profiles:** Use the routing profile supported by the GraphHopper account and endpoint
- **Route Details:** Receive distance, duration, geometry, and path information
- **Coordinate Support:** Pass coordinate pairs such as `latitude,longitude`
- **Secure Authentication:** Use a GraphHopper API key through Fusion’s secret system
- **Workflow Integration:** Send route results to maps, logistics, reporting, or geospatial nodes
- **Error Routing:** Route invalid requests, API failures, and network errors to the error output

### Use Cases

- Calculate directions between delivery locations
- Estimate route distance and travel time
- Build logistics and fleet workflows
- Enrich records with route information
- Support map visualization and geospatial analysis
- Compare routes between cities, facilities, or service locations

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `key` | `string` | Yes | — | GraphHopper API key used to authenticate the request |
| `points` | `array` | Yes | — | Ordered list of coordinate strings defining the route stops |

### Coordinate Format

Each point in the example uses a `latitude,longitude` string:

```text
33.5731,-7.5898
34.0209,-6.8416
```

The points are processed in the order provided. At least an origin and destination are required; additional points can represent intermediate stops when supported by the route endpoint.

### API Request

The node uses the GraphHopper Routing API route endpoint:

```text
GET https://graphhopper.com/api/1/route
```

The API key is sent as the `key` query parameter together with the route points and other request options supported by the runtime.

### API Key Authentication

Store the key in Fusion’s secret system and reference it dynamically:

```json
{
  "key": "{{secrets.graphHopperApiKey}}"
}
```

Never commit a real GraphHopper API key in a workflow file. The included example contains a literal key-shaped value and should be treated as exposed; revoke or rotate it and replace it with a secret reference.

### Routing Profiles

GraphHopper supports profiles such as car, bike, and foot depending on the API configuration. The current node example exposes only `key` and `points`; use the runtime’s supported options if a profile can be configured.

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Input

| Input | Type | Description |
|-------|------|-------------|
| `input` | `object` | Optional dynamic input containing `key` and `points` overrides |

### Success Output

| Output | Type | Description |
|--------|------|-------------|
| `success` | `object` | GraphHopper routing response containing route paths and navigation metrics |

### Success Output Example

```json
{
  "paths": [
    {
      "distance": 87000.5,
      "time": 5400000,
      "ascend": 120.4,
      "descend": 98.2,
      "points_encoded": true,
      "points": "encoded-route-geometry"
    }
  ],
  "info": {
    "took": 42
  }
}
```

Distance is normally expressed in meters and time in milliseconds. The exact response fields depend on the GraphHopper API options and runtime normalization.

### Error Output

Invalid coordinates, missing keys, unsupported routes, API errors, rate limits, and network failures are routed to the error output.

```json
{
  "success": false,
  "error": "GraphHopper route request failed"
}
```

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Calculate a Route Between Two Points

```json
{
  "key": "{{secrets.graphHopperApiKey}}",
  "points": [
    "33.5731,-7.5898",
    "34.0209,-6.8416"
  ]
}
```

### Calculate a Multi-Stop Route

Provide the stops in the order they should be visited:

```json
{
  "key": "{{secrets.graphHopperApiKey}}",
  "points": [
    "33.5731,-7.5898",
    "33.9716,-6.8498",
    "34.0209,-6.8416"
  ]
}
```

### Dynamic Route from Input

A previous node can provide the key and coordinate list dynamically:

```json
{
  "points": [
    "33.5731,-7.5898",
    "34.0209,-6.8416"
  ]
}
```

Keep the API key configured as a secret even when route points come from incoming data.

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Calculate a route between coordinate points
```

### Common Patterns

- **Basic Routing:** Manual Trigger → GraphHopper Route → Log
- **Delivery Planning:** Order Data → GraphHopper Route → Database
- **Travel Estimate:** Location Input → GraphHopper Route → Notification
- **Map Visualization:** GraphHopper Route → GeoJSON or Geospatial Processing
- **Fleet Workflow:** Delivery Stops → GraphHopper Route → Dashboard

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### API key is missing

**Cause:** The `key` parameter is empty or the referenced Fusion secret is not configured.

**Solution:** Add a valid GraphHopper API key to the secret system and reference it with `{{secrets.graphHopperApiKey}}`.

#### Unauthorized request

**Cause:** The API key is invalid, expired, revoked, or does not have access to the requested service.

**Solution:** Verify the key in GraphHopper and confirm that it is passed as the `key` query parameter.

#### Invalid coordinates

**Cause:** A point is malformed, missing latitude or longitude, or uses an unsupported coordinate format.

**Solution:** Use valid coordinate strings such as `33.5731,-7.5898` and provide at least two points.

#### No route found

**Cause:** The points are unreachable for the selected profile or the route cannot be calculated across the requested locations.

**Solution:** Check the coordinates, use a compatible routing profile, and retry with reachable points.

#### Incorrect route distance or duration

**Cause:** The route profile, map data, or coordinate order does not match the intended journey.

**Solution:** Confirm latitude/longitude order, verify the stop sequence, and select the appropriate profile.

#### Rate limit or quota exceeded

**Cause:** The workflow exceeded the GraphHopper API plan’s request or credit limit.

**Solution:** Reduce request frequency, add a delay, or review the GraphHopper account plan and usage.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `HTTP 401` | Missing or invalid API key | Configure a valid secret-backed key |
| `HTTP 403` | Account or plan restriction | Check GraphHopper account permissions and quota |
| `HTTP 400` | Invalid coordinates or request options | Verify points and supported parameters |
| `HTTP 404` | Incorrect endpoint or unavailable route | Verify the API endpoint and coordinates |
| `HTTP 429` | Rate limit exceeded | Reduce request frequency and retry later |
| `HTTP 5xx` | GraphHopper service failure | Retry after a short delay |
| `Network error` | Connection failure | Check connectivity and API availability |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Geospatial Utils** - Transform and analyze geospatial data
- **GeoJSON** - Work with GeoJSON structures and geometry
- [HTTP Request](../http-request/en.md) - Send generic HTTP requests

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation |

<!-- /SECTION: changelog -->
