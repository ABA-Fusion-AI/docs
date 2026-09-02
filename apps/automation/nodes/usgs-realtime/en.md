---
node_id: "usgs-realtime"
title: "USGS Real-time Feed"
description: "Get real-time earthquake feed from USGS. Static file updated every minute."
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
  - realtime
  - environment
  - geojson
related_nodes:
  - usgs-earthquakes
  - emsc-seismic
  - ingv-seismic
  - gdacs
---

<!-- SECTION: header -->

# USGS Real-time Feed

> **Category:** Web Search & Information | **Type:** Action Node

Get real-time earthquake feed data from USGS using predefined magnitude and time-period feeds.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->

## Overview

The **USGS Real-time Feed** node retrieves real-time earthquake data from the USGS Earthquake Hazards Program.

It uses the official USGS GeoJSON summary feeds, which are static feeds updated regularly by USGS.

No authentication or API key is required.

### Key Features

- Retrieves real-time earthquake data from USGS.
- Supports multiple earthquake magnitude thresholds.
- Supports hourly, daily, weekly, and monthly feeds.
- Uses USGS GeoJSON summary feeds.
- Returns earthquake metadata and event features.
- Returns the total number of events in the feed.
- Exposes the feed generation timestamp through `last_updated`.
- Automatically falls back to supported values when invalid parameters are provided.
- Requires no API key or authentication.

### Processing Flow

```text
Input
  ↓
Resolve magnitude and period
  ↓
Validate magnitude
  ↓
Validate period
  ↓
Build USGS GeoJSON feed URL
  ↓
Request real-time earthquake feed
  ↓
Parse GeoJSON response
  ↓
Return metadata and earthquake events
```

### Use Cases

- Monitoring recent earthquake activity.
- Retrieving earthquakes from the past hour.
- Monitoring daily seismic activity.
- Tracking earthquakes above a specific magnitude.
- Building earthquake alert workflows.
- Collecting weekly or monthly seismic data.
- Sending earthquake data to downstream processing nodes.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->

## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `magnitude` | `string` | No | `2.5` | Earthquake magnitude feed to retrieve. |
| `period` | `string` | No | `day` | Time period covered by the feed. |

### Magnitude

Select the earthquake magnitude feed.

Supported values:

| Value | Description |
|-------|-------------|
| `1.0` | Earthquakes with magnitude 1.0 or greater. |
| `2.5` | Earthquakes with magnitude 2.5 or greater. |
| `4.5` | Earthquakes with magnitude 4.5 or greater. |
| `all` | All earthquakes available in the selected feed period. |

Default:

```text
2.5
```

If an unsupported value is provided, the node automatically falls back to:

```text
2.5
```

For example:

```text
magnitude: 3.0
```

is resolved as:

```text
magnitude: 2.5
```

### Period

Select the time period covered by the earthquake feed.

Supported values:

| Value | Description |
|-------|-------------|
| `hour` | Earthquakes from the past hour. |
| `day` | Earthquakes from the past day. |
| `week` | Earthquakes from the past week. |
| `month` | Earthquakes from the past month. |

Default:

```text
day
```

If an unsupported value is provided, the node automatically falls back to:

```text
day
```

For example:

```text
period: year
```

is resolved as:

```text
period: day
```

### Empty Values

Empty values also use the defaults.

```text
magnitude: empty → 2.5
period: empty    → day
```

### Feed URL

The node constructs the USGS feed URL using:

```text
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/{magnitude}_{period}.geojson
```

For example:

```text
magnitude: 4.5
period: day
```

uses:

```text
https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/4.5_day.geojson
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->

## Inputs & Outputs

### Inputs

The node accepts incoming workflow data through its input connection.

The earthquake feed itself is selected using the configured `magnitude` and `period` parameters.

### Output

The node returns an object containing the resolved feed configuration, USGS metadata, earthquake features, event count, and feed generation timestamp.

Example:

```json
{
  "success": true,
  "magnitude": "4.5",
  "period": "day",
  "metadata": {},
  "features": [],
  "total_events": 0,
  "last_updated": 1788348510000
}
```

### Output Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | `boolean` | Indicates that the USGS request completed successfully. |
| `magnitude` | `string` | Magnitude value actually used for the feed request. |
| `period` | `string` | Period value actually used for the feed request. |
| `metadata` | `object` | Metadata returned by the USGS GeoJSON feed. |
| `features` | `array` | Earthquake events returned by USGS. |
| `total_events` | `number` | Number of earthquake events contained in `features`. |
| `last_updated` | `number` | Feed generation timestamp from `metadata.generated`. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->

## Examples

### Example 1: Daily Magnitude 2.5+ Feed

**Configuration**

```text
magnitude: 2.5
period: day
```

The node requests:

```text
2.5_day.geojson
```

and returns earthquakes with magnitude 2.5 or greater from the daily USGS feed.

### Example 2: Hourly Feed

**Configuration**

```text
magnitude: 2.5
period: hour
```

The node requests:

```text
2.5_hour.geojson
```

and returns the corresponding hourly earthquake feed.

### Example 3: All Daily Earthquakes

**Configuration**

```text
magnitude: all
period: day
```

The node requests:

```text
all_day.geojson
```

and returns all earthquakes available in the daily feed.

### Example 4: Invalid Magnitude Fallback

**Configuration**

```text
magnitude: 3.0
period: day
```

Because `3.0` is not a supported magnitude feed, the node resolves the configuration to:

```text
magnitude: 2.5
period: day
```

and requests:

```text
2.5_day.geojson
```

### Example 5: Invalid Period Fallback

**Configuration**

```text
magnitude: 2.5
period: year
```

Because `year` is not supported, the node resolves the configuration to:

```text
magnitude: 2.5
period: day
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: USGS Real-time Feed Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->

## Troubleshooting

### Magnitude Falls Back to 2.5

**Cause:** The configured magnitude is not one of the supported values.

Supported values are:

```text
1.0
2.5
4.5
all
```

**Solution:** Use one of the supported values when a specific magnitude feed is required.

### Period Falls Back to Day

**Cause:** The configured period is not one of the supported values.

Supported values are:

```text
hour
day
week
month
```

**Solution:** Use one of the supported period values.

### Empty Parameters Use Defaults

If `magnitude` or `period` is empty, the node applies its runtime defaults:

```text
magnitude → 2.5
period    → day
```

This is expected behavior.

### No Earthquake Events Are Returned

A valid feed may contain zero earthquake events depending on the selected magnitude and period.

Check `total_events` and the returned `features` array.

### USGS Feed Request Fails

When the USGS endpoint returns a non-successful HTTP status, the node creates an API error:

```text
USGS Real-time Feed API error: <status>
```

The request handler wraps the error as:

```text
USGS Real-time Feed request failed: USGS Real-time Feed API error: <status>
```

### Event Count Changes Between Executions

USGS feeds contain real-time earthquake information and are updated regularly.

The number of events, feed metadata, and `last_updated` value can therefore change between workflow executions.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->

## Related

- **USGS Earthquakes** — Query USGS earthquake events using date, magnitude, format, and limit filters.
- **EMSC Seismic** — Access seismic event information from EMSC.
- **INGV Seismic** — Work with seismic information from INGV.
- **GDACS** — Access disaster alert and coordination information.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-09-02 | Initial documentation for the USGS Real-time Feed node. |

<!-- /SECTION: changelog -->