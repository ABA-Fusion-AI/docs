---
node_id: "usgs-earthquakes"
title: "USGS Earthquakes"
description: "Query earthquake data from the USGS Earthquake API."
category: "web-search-information"
subcategory: "environment"
version: "1.0.0"
language: "en"
last_updated: "2026-09-02"
author: "Fusion Team"
tags:
  - usgs
  - earthquakes
  - seismic
  - environment
  - geojson
  - api
related_nodes:
  - usgs-realtime
  - emsc-seismic
  - ingv-seismic
  - gdacs
---

<!-- SECTION: header -->

# USGS Earthquakes

> **Category:** Web Search & Information | **Type:** Action Node

Query earthquake event data from the USGS Earthquake API using date, magnitude, format, and result limit filters.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **USGS Earthquakes** node queries earthquake event data from the USGS Earthquake API.

It allows workflows to retrieve seismic events using optional date and magnitude filters.

No authentication or API key is required.

### Key Features

- Queries earthquake events from the USGS Earthquake API.
- Supports optional start and end date filters.
- Supports minimum magnitude filtering.
- Supports maximum magnitude filtering.
- Uses `geojson` as the default response format.
- Supports configurable result limits.
- Limits API requests to a maximum of `20000` events.
- Returns USGS metadata and earthquake features.
- Returns the total number of events contained in the response.
- Requires no API key or authentication.

### Processing Flow

```text
Input
  ↓
Resolve configured parameters
  ↓
Build USGS query parameters
  ↓
Apply result limit
  ↓
Send request to USGS Earthquake API
  ↓
Parse JSON response
  ↓
Return metadata and earthquake features
```

### Use Cases

- Monitoring earthquake activity.
- Retrieving seismic events for a specific period.
- Filtering earthquakes by magnitude.
- Building earthquake monitoring workflows.
- Collecting seismic data for analysis.
- Integrating USGS earthquake data with downstream workflow nodes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `starttime` | `string` | No | Empty | Start date or timestamp for the earthquake query. |
| `endtime` | `string` | No | Empty | End date or timestamp for the earthquake query. |
| `minmagnitude` | `number` | No | `0` | Minimum earthquake magnitude. Values greater than `0` are included in the API request. |
| `maxmagnitude` | `number` | No | `0` | Maximum earthquake magnitude. Values greater than `0` are included in the API request. |
| `format` | `string` | No | `geojson` | Response format requested from the USGS API. |
| `limit` | `number` | No | `100` | Maximum number of earthquake events requested. |

### Starttime

Specify the start date or timestamp for the query.

Example:

```text
2026-08-01
```

If `starttime` is empty, it is not included in the API request.

### Endtime

Specify the end date or timestamp for the query.

Example:

```text
2026-08-02
```

If `endtime` is empty, it is not included in the API request.

### Minmagnitude

Specify the minimum earthquake magnitude.

Example:

```text
5
```

The parameter is included in the API request only when its value is greater than `0`.

When configured as `0`, `minmagnitude` is omitted from the request.

### Maxmagnitude

Specify the maximum earthquake magnitude.

Example:

```text
3
```

The parameter is included in the API request only when its value is greater than `0`.

When configured as `0`, `maxmagnitude` is omitted from the request.

### Format

Specify the response format requested from the USGS API.

Default:

```text
geojson
```

The current node implementation parses the response using JSON parsing, so `geojson` is the appropriate format for normal usage.

### Limit

Specify the maximum number of earthquake events requested.

Default:

```text
100
```

The request limit is constrained between `1` and `20000`.

The node applies the following behavior:

- `0` falls back to `100`.
- Values between `1` and `20000` are sent unchanged.
- Values greater than `20000` are sent to the API as `20000`.

The `query.limit` value returned by the node contains the configured value rather than the clamped request value.

For example, with:

```text
limit: 20001
```

the API request uses:

```text
limit=20000
```

while the output contains:

```text
query.limit: 20001
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The node accepts incoming workflow data through its input connection.

The earthquake API query is controlled by the configured node parameters.

### API Request

The node sends a request to the USGS Earthquake API.

Empty date values are omitted from the request.

Magnitude filters are included only when their configured values are greater than `0`.

### Output

The node returns an object containing the query information, USGS response metadata, earthquake features, and event count.

Example:

```json
{
  "success": true,
  "query": {
    "starttime": "2026-08-01",
    "endtime": "2026-08-02",
    "minmagnitude": 5,
    "maxmagnitude": 0,
    "format": "geojson",
    "limit": 10
  },
  "metadata": {},
  "features": [],
  "total_events": 0
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates that the API query completed successfully. |
| `query` | `object` | Contains the configured query values. |
| `metadata` | `object` | Metadata returned by the USGS API. |
| `features` | `array` | Earthquake events returned by the USGS API. |
| `total_events` | `number` | Number of events contained in `features`. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Example 1: Query Earthquakes by Minimum Magnitude

**Configuration**

```text
starttime: 2026-08-01
endtime: 2026-08-02
minmagnitude: 5
maxmagnitude: 0
format: geojson
limit: 10
```

The generated request includes:

```text
format=geojson
limit=10
starttime=2026-08-01
endtime=2026-08-02
minmagnitude=5
```

Because `maxmagnitude` is `0`, it is not included in the API request.

### Example 2: Query Earthquakes by Maximum Magnitude

**Configuration**

```text
starttime: 2026-08-01
endtime: 2026-08-02
minmagnitude: 0
maxmagnitude: 3
format: geojson
limit: 10
```

In this case, `minmagnitude` is omitted and `maxmagnitude=3` is included in the request.

### Example 3: Query Without a Date Range

**Configuration**

```text
starttime:
endtime:
minmagnitude: 5
maxmagnitude: 0
format: geojson
limit: 5
```

Because `starttime` and `endtime` are empty, neither parameter is included in the API request.

### Example 4: Limit Fallback

With:

```text
limit: 0
```

the node uses:

```text
limit: 100
```

because the runtime configuration applies the default fallback value.

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: USGS Earthquakes Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Limit Returns 100 When Configured as 0

**Cause:** The runtime configuration uses `100` as the fallback when `limit` is `0`.

**Solution:** Use a positive value such as `1` if a smaller result set is required.

### Limit Greater Than 20000

**Cause:** The node constrains the limit sent to the USGS API to a maximum of `20000`.

For example:

```text
Configured limit: 20001
API request limit: 20000
Output query.limit: 20001
```

This behavior is expected.

### Starttime or Endtime Is Missing From the Request

**Cause:** Empty `starttime` and `endtime` values are intentionally omitted.

**Solution:** Provide a date or timestamp when a date filter is required.

### Magnitude Filter Is Missing From the Request

**Cause:** `minmagnitude` and `maxmagnitude` are included only when their values are greater than `0`.

A value of `0` means the corresponding filter is omitted.

### Reversed Date Range Does Not Return an Error

The node does not validate that `starttime` occurs before `endtime`.

During testing, the USGS API accepted a reversed date range and returned an HTTP `200` response.

### Response Cannot Be Parsed

The node parses the API response as JSON.

Use:

```text
format: geojson
```

for normal operation.

### USGS API Request Fails

When the API returns a non-successful HTTP status, the node wraps the API error using:

```text
USGS Earthquakes query failed: USGS Earthquakes API error: <status>
```

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- **USGS Realtime** — Access realtime USGS earthquake information.
- **EMSC Seismic** — Access seismic event information from EMSC.
- **INGV Seismic** — Work with seismic information from INGV.
- **GDACS** — Access disaster alert and coordination information.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation for the USGS Earthquakes node. |

<!-- /SECTION: changelog -->