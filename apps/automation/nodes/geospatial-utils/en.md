---
node_id: "geospatial-utils"
title: "Geospatial Utils"
description: "Geospatial distance & location utilities for logistics. Calculates distances, bearings, midpoints, route optimization, and more."
category: "business-commerce"
subcategory: "logistics-supply-chain"
version: "1.0.0"
language: "en"
last_updated: "2026-08-31"
author: "Fusion Team"
tags:
  - geospatial
  - location
  - distance
  - coordinates
  - logistics
  - routing
  - bearing
  - geography
related_nodes:
  - inventory-logistics
  - mrp-engine
  - quality-maintenance
---

<!-- SECTION: header -->
# Geospatial Utils

> **Category:** Business & Commerce | **Type:** Action Node

Calculate geographic distances, bearings, midpoints, destinations, radius checks, closest points, route distances, bounding boxes, and estimated travel times.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **Geospatial Utils** node provides geospatial distance and location utilities for logistics and location-based workflows.

It performs geographic calculations locally from latitude and longitude coordinates. The node supports distance calculations, bearings, midpoints, destination coordinates, radius checks, closest-point searches, route distances, bounding boxes, and travel-time estimates.

The operation is selected through the `operation` parameter. Each operation exposes its own configuration fields.

Supported operations:

- Calculate Distance
- Calculate Bearing
- Calculate Midpoint
- Calculate Destination
- Check Within Radius
- Find Closest Point
- Calculate Route Distance
- Calculate Bounding Box
- Estimate Travel Time

### Key Features

- Calculate geographic distance between two coordinates.
- Return distance in kilometers, miles, meters, or nautical miles.
- Calculate compass bearing between two locations.
- Calculate the geographic midpoint between coordinates.
- Calculate a destination from a starting point, bearing, and distance.
- Check whether a point is inside a specified radius.
- Find the closest location from a JSON list of points.
- Calculate total distance through multiple waypoints.
- Return distance and bearing information for individual route segments.
- Calculate a bounding box around multiple coordinates.
- Estimate travel time from distance and average speed.
- Validate latitude and longitude ranges.
- Perform calculations locally without external API credentials.

### Processing Flow

```text
Select operation
        ↓
Load operation-specific parameters
        ↓
Validate coordinates and input data
        ↓
Perform geospatial calculation
        ↓
Build result
        ↓
Return calculated data
```

### Use Cases

- Logistics and delivery workflows.
- Distance calculations between locations.
- Route analysis.
- Warehouse and distribution planning.
- Geographic proximity checks.
- Finding the closest service point.
- Location-based automation.
- Fleet and transportation calculations.
- Geographic boundary calculations.
- Travel-time estimation.

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Operation

The `operation` parameter determines which geospatial calculation is performed.

Supported values:

```text
Calculate Distance
Calculate Bearing
Calculate Midpoint
Calculate Destination
Check Within Radius
Find Closest Point
Calculate Route Distance
Calculate Bounding Box
Estimate Travel Time
```

The default operation is:

```text
Calculate Distance
```

---

### Calculate Distance

Calculates the geographic distance between two coordinates using the Haversine formula.

Select:

```text
operation: Calculate Distance
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lat1` | `number` | Yes | Latitude of the starting point. |
| `lon1` | `number` | Yes | Longitude of the starting point. |
| `lat2` | `number` | Yes | Latitude of the destination point. |
| `lon2` | `number` | Yes | Longitude of the destination point. |
| `unit` | `string` | No | Output unit: `km`, `mi`, `m`, or `nm`. Default is `km`. |

Example:

```text
operation: Calculate Distance
lat1: 34.0209
lon1: -6.8416
lat2: 33.5731
lon2: -7.5898
unit: km
```

Example output:

```json
{
  "distance": 85.20096475445087,
  "unit": "km",
  "allUnits": {
    "km": 85.20096475445087,
    "mi": 52.94468991726118,
    "m": 85200.96475445086,
    "nm": 46.004857325919026
  },
  "coordinates": {
    "from": {
      "lat": 34.0209,
      "lon": -6.8416
    },
    "to": {
      "lat": 33.5731,
      "lon": -7.5898
    }
  }
}
```

The result includes the selected distance unit together with equivalent values in all supported units.

---

### Calculate Bearing

Calculates the initial bearing from one geographic point to another.

Select:

```text
operation: Calculate Bearing
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lat1` | `number` | Yes | Latitude of the starting point. |
| `lon1` | `number` | Yes | Longitude of the starting point. |
| `lat2` | `number` | Yes | Latitude of the destination point. |
| `lon2` | `number` | Yes | Longitude of the destination point. |

Example:

```text
operation: Calculate Bearing
lat1: 34.0209
lon1: -6.8416
lat2: 33.5731
lon2: -7.5898
```

Example output:

```json
{
  "degrees": 234.44679788216578,
  "direction": "SW",
  "radians": 4.0918685438014615
}
```

The bearing is normalized between `0` and `360` degrees.

The `direction` field uses one of 16 compass directions:

```text
N
NNE
NE
ENE
E
ESE
SE
SSE
S
SSW
SW
WSW
W
WNW
NW
NNW
```

---

### Calculate Midpoint

Calculates the geographic midpoint between two coordinates.

Select:

```text
operation: Calculate Midpoint
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lat1` | `number` | Yes | Latitude of the first point. |
| `lon1` | `number` | Yes | Longitude of the first point. |
| `lat2` | `number` | Yes | Latitude of the second point. |
| `lon2` | `number` | Yes | Longitude of the second point. |

Example:

```text
operation: Calculate Midpoint
lat1: 34.0209
lon1: -6.8416
lat2: 33.5731
lon2: -7.5898
```

Example output:

```json
{
  "lat": 33.797564552666394,
  "lon": -7.2166785681797165,
  "formatted": "33.797565, -7.216679"
}
```

The `formatted` field returns the calculated latitude and longitude with six decimal places.

---

### Calculate Destination

Calculates a destination coordinate from a starting coordinate, bearing, and distance.

Select:

```text
operation: Calculate Destination
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `startLat` | `number` | Yes | Starting latitude. |
| `startLon` | `number` | Yes | Starting longitude. |
| `bearing` | `number` | Yes | Direction of travel in degrees. |
| `distance` | `number` | Yes | Distance to travel. |
| `distanceUnit` | `string` | No | Distance unit: `km`, `mi`, `m`, or `nm`. Default is `km`. |

Example:

```text
operation: Calculate Destination
startLat: 34.0209
startLon: -6.8416
bearing: 180
distance: 100
distanceUnit: km
```

Example output:

```json
{
  "lat": 33.12157839408127,
  "lon": -6.8416,
  "formatted": "33.121578, -6.841600"
}
```

The destination is calculated along the Earth's surface using the provided bearing and distance.

---

### Check Within Radius

Checks whether a geographic point is located within a specified radius of a center point.

Select:

```text
operation: Check Within Radius
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `centerLat` | `number` | Yes | Latitude of the radius center. |
| `centerLon` | `number` | Yes | Longitude of the radius center. |
| `pointLat` | `number` | Yes | Latitude of the point to check. |
| `pointLon` | `number` | Yes | Longitude of the point to check. |
| `radiusKm` | `number` | Yes | Radius in kilometers. |

Example:

```text
operation: Check Within Radius
centerLat: 34
centerLon: -7
pointLat: 34
pointLon: -8
radiusKm: 100
```

Example output:

```json
{
  "isWithin": true,
  "distance": 92.18440618930562,
  "radius": 100,
  "unit": "km"
}
```

`isWithin` is `true` when the calculated distance is less than or equal to `radiusKm`.

---

### Find Closest Point

Finds the closest point to an origin from a JSON array of geographic coordinates.

Select:

```text
operation: Find Closest Point
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `originLat` | `number` | Yes | Latitude of the origin. |
| `originLon` | `number` | Yes | Longitude of the origin. |
| `points` | `string` | Yes | JSON array containing candidate points with `lat` and `lon` properties. |

Example:

```text
operation: Find Closest Point
originLat: 34
originLon: -7
points: [{"lat":34,"lon":-8,"name":"A"},{"lat":35,"lon":-7,"name":"B"},{"lat":34,"lon":-7.5,"name":"C"}]
```

Example output:

```json
{
  "point": {
    "lat": 34,
    "lon": -7.5,
    "name": "C"
  },
  "index": 2,
  "distance": 46.092340299030845,
  "unit": "km"
}
```

Additional properties in each point object, such as `name`, are preserved in the returned closest point.

---

### Calculate Route Distance

Calculates the total geographic distance through a sequence of waypoints.

Select:

```text
operation: Calculate Route Distance
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `waypoints` | `string` | Yes | JSON array containing at least two points with `lat` and `lon` properties. |
| `routeUnit` | `string` | No | Distance unit: `km`, `mi`, `m`, or `nm`. Default is `km`. |

Example:

```text
operation: Calculate Route Distance
waypoints: [{"lat":34,"lon":-7,"name":"A"},{"lat":34,"lon":-8,"name":"B"},{"lat":35,"lon":-8,"name":"C"}]
routeUnit: km
```

Example output structure:

```json
{
  "totalDistance": 203.37933283386434,
  "unit": "km",
  "segments": [
    {
      "from": {
        "lat": 34,
        "lon": -7,
        "name": "A"
      },
      "to": {
        "lat": 34,
        "lon": -8,
        "name": "B"
      },
      "distance": 92.18440618930562,
      "bearing": {
      "degrees": 270.2796,
      "direction": "W"
      }
    }
  ],
  "waypointCount": 3
}
```

The `segments` array contains one entry for every pair of consecutive waypoints.

Each segment includes:

```text
from
to
distance
bearing
```

The tested three-waypoint route produced two route segments.

---

### Calculate Bounding Box

Calculates the smallest axis-aligned geographic bounding box containing all supplied points.

Select:

```text
operation: Calculate Bounding Box
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `boundingBoxPoints` | `string` | Yes | JSON array of points containing `lat` and `lon` properties. |

Example:

```text
operation: Calculate Bounding Box
boundingBoxPoints: [{"lat":34,"lon":-7},{"lat":35,"lon":-8},{"lat":33,"lon":-6}]
```

Example output:

```json
{
  "north": 35,
  "south": 33,
  "east": -6,
  "west": -8,
  "center": {
    "lat": 34,
    "lon": -7
  }
}
```

The returned `center` is calculated from the minimum and maximum latitude and longitude values.

---

### Estimate Travel Time

Estimates travel time from a distance in kilometers and an average speed in kilometers per hour.

Select:

```text
operation: Estimate Travel Time
```

Parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `distanceKm` | `number` | Yes | Travel distance in kilometers. |
| `avgSpeedKmh` | `number` | Yes | Average travel speed in kilometers per hour. Must be greater than `0`. |

Example:

```text
operation: Estimate Travel Time
distanceKm: 120
avgSpeedKmh: 80
```

Example output:

```json
{
  "hours": 1,
  "minutes": 30,
  "totalMinutes": 90,
  "totalHours": 1.5,
  "formatted": "1h 30m"
}
```

The node calculates:

```text
travel time = distanceKm / avgSpeedKmh
```

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

The node uses operation-specific parameters.

| Operation | Required Parameters |
|-----------|---------------------|
| `Calculate Distance` | `lat1`, `lon1`, `lat2`, `lon2` |
| `Calculate Bearing` | `lat1`, `lon1`, `lat2`, `lon2` |
| `Calculate Midpoint` | `lat1`, `lon1`, `lat2`, `lon2` |
| `Calculate Destination` | `startLat`, `startLon`, `bearing`, `distance` |
| `Check Within Radius` | `centerLat`, `centerLon`, `pointLat`, `pointLon`, `radiusKm` |
| `Find Closest Point` | `originLat`, `originLon`, `points` |
| `Calculate Route Distance` | `waypoints` |
| `Calculate Bounding Box` | `boundingBoxPoints` |
| `Estimate Travel Time` | `distanceKm`, `avgSpeedKmh` |

Optional parameters depend on the selected operation.

Coordinate-based operations expect:

```text
Latitude:  -90 to 90
Longitude: -180 to 180
```

### Outputs

The output depends on the selected operation.

Common output fields include:

| Field | Description |
|-------|-------------|
| `distance` | Calculated geographic distance. |
| `unit` | Unit associated with a distance result. |
| `allUnits` | Distance converted to all supported units. |
| `coordinates` | Starting and destination coordinates. |
| `degrees` | Bearing in degrees. |
| `direction` | Compass direction. |
| `radians` | Bearing in radians. |
| `lat` | Calculated latitude. |
| `lon` | Calculated longitude. |
| `formatted` | Human-readable formatted result. |
| `isWithin` | Indicates whether a point is within a radius. |
| `point` | Closest point found. |
| `index` | Index of the closest point. |
| `totalDistance` | Total distance through route waypoints. |
| `segments` | Individual route segments. |
| `waypointCount` | Number of waypoints in a route. |
| `north` | Northern boundary of a bounding box. |
| `south` | Southern boundary of a bounding box. |
| `east` | Eastern boundary of a bounding box. |
| `west` | Western boundary of a bounding box. |
| `hours` | Whole-hour component of estimated travel time. |
| `minutes` | Minute component of estimated travel time. |
| `totalMinutes` | Total estimated travel time in minutes. |
| `totalHours` | Total estimated travel time in hours. |

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Example 1: Calculate Distance

**Configuration**

```text
operation: Calculate Distance
lat1: 34.0209
lon1: -6.8416
lat2: 33.5731
lon2: -7.5898
unit: km
```

**Output**

```text
distance: 85.20096475445087
unit: km
```

### Example 2: Calculate Bearing

**Configuration**

```text
operation: Calculate Bearing
lat1: 34.0209
lon1: -6.8416
lat2: 33.5731
lon2: -7.5898
```

**Output**

```text
degrees: 234.44679788216578
direction: SW
radians: 4.0918685438014615
```

### Example 3: Calculate Midpoint

**Configuration**

```text
operation: Calculate Midpoint
lat1: 34.0209
lon1: -6.8416
lat2: 33.5731
lon2: -7.5898
```

**Output**

```text
lat: 33.797564552666394
lon: -7.2166785681797165
formatted: 33.797565, -7.216679
```

### Example 4: Calculate Destination

**Configuration**

```text
operation: Calculate Destination
startLat: 34.0209
startLon: -6.8416
bearing: 180
distance: 100
distanceUnit: km
```

**Output**

```text
lat: 33.12157839408127
lon: -6.8416
formatted: 33.121578, -6.841600
```

### Example 5: Check Within Radius

**Configuration**

```text
operation: Check Within Radius
centerLat: 34
centerLon: -7
pointLat: 34
pointLon: -8
radiusKm: 100
```

**Output**

```text
isWithin: true
distance: 92.18440618930562
radius: 100
unit: km
```

### Example 6: Find Closest Point

**Configuration**

```text
operation: Find Closest Point
originLat: 34
originLon: -7
points: [{"lat":34,"lon":-8,"name":"A"},{"lat":35,"lon":-7,"name":"B"},{"lat":34,"lon":-7.5,"name":"C"}]
```

The closest point is:

```text
name: C
index: 2
distance: 46.092340299030845 km
```

### Example 7: Calculate Route Distance

**Configuration**

```text
operation: Calculate Route Distance
waypoints: [{"lat":34,"lon":-7,"name":"A"},{"lat":34,"lon":-8,"name":"B"},{"lat":35,"lon":-8,"name":"C"}]
routeUnit: km
```

**Result**

```text
totalDistance: 203.37933283386434
unit: km
waypointCount: 3
segments: 2
```

### Example 8: Calculate Bounding Box

**Configuration**

```text
operation: Calculate Bounding Box
boundingBoxPoints: [{"lat":34,"lon":-7},{"lat":35,"lon":-8},{"lat":33,"lon":-6}]
```

**Output**

```json
{
  "north": 35,
  "south": 33,
  "east": -6,
  "west": -8,
  "center": {
    "lat": 34,
    "lon": -7
  }
}
```

### Example 9: Estimate Travel Time

**Configuration**

```text
operation: Estimate Travel Time
distanceKm: 120
avgSpeedKmh: 80
```

**Output**

```json
{
  "hours": 1,
  "minutes": 30,
  "totalMinutes": 90,
  "totalHours": 1.5,
  "formatted": "1h 30m"
}
```

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Geospatial Utils Example
```

<!-- /SECTION: examples -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Coordinates Are Required

Coordinate-based operations require all coordinates used by the selected calculation.

For example, `Calculate Distance`, `Calculate Bearing`, and `Calculate Midpoint` require:

```text
lat1
lon1
lat2
lon2
```

Missing required values cause the node to return a `Geospatial Utils error`.

### Invalid Latitude

Latitude must be between:

```text
-90 and 90
```

Values outside this range cause:

```text
Latitude must be between -90 and 90
```

### Invalid Longitude

Longitude must be between:

```text
-180 and 180
```

Values outside this range cause:

```text
Longitude must be between -180 and 180
```

### Invalid Points JSON

`Find Closest Point` expects `points` to contain valid JSON representing a non-empty array.

Example:

```json
[
  {
    "lat": 34,
    "lon": -7
  },
  {
    "lat": 35,
    "lon": -8
  }
]
```

Invalid JSON produces an error prefixed with:

```text
Invalid JSON in points:
```

### Invalid Waypoints JSON

`Calculate Route Distance` expects `waypoints` to contain a JSON array with at least two points.

Each waypoint must contain numeric:

```text
lat
lon
```

Invalid JSON produces an error prefixed with:

```text
Invalid JSON in waypoints:
```

### Invalid Bounding Box Points

`Calculate Bounding Box` expects a non-empty JSON array containing points with numeric latitude and longitude values.

Invalid JSON produces an error prefixed with:

```text
Invalid JSON in boundingBoxPoints:
```

### Average Speed Must Be Positive

`Estimate Travel Time` requires `avgSpeedKmh` to be greater than `0`.

An invalid speed produces:

```text
Average speed must be greater than 0
```

### Check Within Radius Fields

`Check Within Radius` uses:

```text
centerLat
centerLon
pointLat
pointLon
radiusKm
```

These values are required by the calculation.

During testing, the UI also displayed `lat1` and `lon1` for this operation even though the calculation does not use those fields.

### Decimal Coordinate Input

During testing, some numeric fields in the `Check Within Radius` UI rejected decimal values through the browser's numeric input validation.

The underlying geospatial calculation supports numeric decimal coordinates. Integer coordinates were used to validate the operation successfully.

### Geospatial Utils Error

Errors thrown during processing are returned with the prefix:

```text
Geospatial Utils error:
```

Check the operation-specific parameters and coordinate values when this error appears.

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- **Inventory Logistics** — Manage inventory and logistics-oriented workflow data.
- **MRP Engine** — Perform material requirements planning calculations for supply-chain workflows.
- **Quality Maintenance** — Perform quality assurance and maintenance calculations for logistics and operational workflows.

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-31 | Initial documentation for the Geospatial Utils node. |

<!-- /SECTION: changelog -->