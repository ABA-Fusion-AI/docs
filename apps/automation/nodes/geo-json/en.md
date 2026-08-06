---
node_id: "geojson"
title: "GeoJSON Tools"
description: "Analyze, transform, and measure GeoJSON data"
category: "utilities"
subcategory: "data"
version: "1.0.0"
language: "en"
last_updated: "2026-08-06"
author: "Fusion Team"
tags:
  - utility
  - geojson
  - geospatial
  - transform
  - validation
related_nodes:
  - http-request
  - function
  - log
---

<!-- SECTION: header -->
# GeoJSON Tools

> **Category:** Utilities | **Type:** Action Node

Analyze, transform, validate, and measure GeoJSON data using common geospatial operations.

<!-- /SECTION: header -->

---

<!-- SECTION: overview -->
## Overview

The **GeoJSON Tools** node performs geospatial analysis and transformation operations on GeoJSON data.

It accepts GeoJSON as serialized JSON strings and supports validation, measurements, geometry transformations, set operations, and spatial predicates.

### Key Features

- **Validation:** Check whether a GeoJSON document contains a valid `type` property
- **Bounding Box:** Calculate minimum and maximum coordinates
- **Centroid:** Calculate the center point of a geometry
- **Measurements:** Calculate area, length, and distance
- **Geometry Transformation:** Buffer, simplify, generate convex hulls, and create circles
- **Set Operations:** Union, intersect, or subtract polygon geometries
- **Spatial Predicate:** Check whether a point is inside a polygon
- **Multiple Units:** Use kilometers, miles, meters, or degrees where supported

### Supported Operations

| Operation | Description |
|-----------|-------------|
| `validate` | Perform basic GeoJSON validation |
| `bounding_box` | Return `[minX, minY, maxX, maxY]` |
| `centroid` | Calculate the centroid of a geometry |
| `area` | Calculate polygon area |
| `length` | Calculate the length of line geometries |
| `distance` | Calculate the distance between the centroids of two GeoJSON inputs |
| `buffer` | Create a buffer around a geometry |
| `simplify` | Reduce geometry complexity |
| `convex_hull` | Generate the smallest convex polygon containing the input |
| `circle` | Generate a circular polygon from the input centroid |
| `union` | Merge two polygon geometries |
| `intersection` | Return the overlapping area of two polygon geometries |
| `difference` | Subtract the second polygon from the first |
| `boolean_point_in_polygon` | Check whether a point is inside a polygon |

### Use Cases

- Validate GeoJSON before storage or processing
- Calculate map boundaries
- Measure routes and polygon areas
- Generate buffers around locations
- Simplify complex geographic shapes
- Merge or compare polygon geometries
- Check whether a location belongs to a defined area
- Process geospatial API responses

<!-- /SECTION: overview -->

---

<!-- SECTION: configuration -->
## Configuration

### Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `operation` | `enum` | ✅ Yes | `bounding_box` | Geospatial operation to execute |
| `geojson` | `string` | ✅ Yes | — | Primary GeoJSON document serialized as a JSON string |
| `geojson2` | `string` | Conditional | — | Secondary GeoJSON document required by binary operations |
| `radius` | `number` | Conditional | `1` | Radius used by `buffer` and `circle` |
| `tolerance` | `number` | Conditional | `0.01` | Simplification tolerance used by `simplify` |
| `units` | `enum` | Conditional | `kilometers` | Unit used by measurement and radius-based operations |

### Supported Units

The `units` parameter accepts:

```text
kilometers
miles
meters
degrees
```

It is displayed for these operations:

- `area`
- `length`
- `distance`
- `buffer`
- `circle`

### GeoJSON Input Format

The `geojson` and `geojson2` parameters must contain valid GeoJSON serialized as JSON strings.

Example:

```json
{"type":"Point","coordinates":[-7.5898,33.5731]}
```

Coordinates must follow the GeoJSON coordinate order:

```text
[longitude, latitude]
```

For example, Casablanca can be represented approximately as:

```json
{"type":"Point","coordinates":[-7.5898,33.5731]}
```

### Secondary Input Requirements

The `geojson2` parameter is required for:

- `distance`
- `union`
- `intersection`
- `difference`
- `boolean_point_in_polygon`

### Operation-Specific Requirements

#### Distance

Both inputs can be any valid GeoJSON supported by centroid calculation. The node calculates the centroid of each input before measuring the distance.

#### Union

Both inputs should be polygon-based GeoJSON objects such as `Polygon` or `MultiPolygon` features.

#### Intersection

Both inputs should be overlapping polygon-based GeoJSON objects.

#### Difference

Both inputs should be polygon-based GeoJSON objects. The second geometry is subtracted from the first.

#### Boolean Point in Polygon

The expected inputs are:

```text
geojson  = Point
geojson2 = Polygon or MultiPolygon
```

#### Circle

The circle center is calculated from the centroid of `geojson`.

#### Area

The underlying area is calculated in square meters.

When `units` is:

- `kilometers`, the result is converted to square kilometers
- `miles`, the result is converted to square miles
- `meters` or `degrees`, the raw square-meter value is returned

<!-- /SECTION: configuration -->

---

<!-- SECTION: inputs-outputs -->
## Inputs & Outputs

### Inputs

| Input | Type | Description |
|-------|------|-------------|
| `input` | `any` | Incoming workflow data |

The current implementation reads its values from the configured parameters.

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| `success` | `any` | Result returned by the selected geospatial operation |
| `error` | `Error` | Emitted when parsing or execution fails |

The output type depends on the operation.

Examples include:

- coordinate arrays
- numbers
- booleans
- GeoJSON features
- validation result objects
- `null` when an operation produces no geometry

<!-- /SECTION: inputs-outputs -->

---

<!-- SECTION: examples -->
## Examples

### Basic Example: Calculate a Bounding Box

Calculate the bounding box of a point.

**Configuration:**

```text
Operation: bounding_box
GeoJSON: {"type":"Point","coordinates":[-7.5898,33.5731]}
```

**Output:**

```json
[
  -7.5898,
  33.5731,
  -7.5898,
  33.5731
]
```

---

### Example: Validate GeoJSON

**Configuration:**

```text
Operation: validate
GeoJSON: {"type":"Point","coordinates":[-7.5898,33.5731]}
```

**Output:**

```json
{
  "valid": true,
  "type": "Point"
}
```

If the `type` property is missing:

```json
{
  "valid": false,
  "reason": "Missing 'type' property"
}
```

---

### Example: Calculate a Centroid

**Configuration:**

```text
Operation: centroid
```

**GeoJSON:**

```json
{"type":"Polygon","coordinates":[[[-7.60,33.57],[-7.58,33.57],[-7.58,33.59],[-7.60,33.59],[-7.60,33.57]]]}
```

The output is a GeoJSON `Point` feature representing the centroid.

---

### Example: Calculate Polygon Area

**Configuration:**

```text
Operation: area
Units: kilometers
```

**GeoJSON:**

```json
{"type":"Polygon","coordinates":[[[-7.60,33.57],[-7.58,33.57],[-7.58,33.59],[-7.60,33.59],[-7.60,33.57]]]}
```

The result is returned in square kilometers.

---

### Example: Calculate Line Length

**Configuration:**

```text
Operation: length
Units: kilometers
```

**GeoJSON:**

```json
{"type":"LineString","coordinates":[[-7.60,33.57],[-7.58,33.59]]}
```

The result is returned as a numeric distance.

---

### Example: Calculate Distance

**Configuration:**

```text
Operation: distance
Units: kilometers
```

**GeoJSON:**

```json
{"type":"Point","coordinates":[-7.5898,33.5731]}
```

**GeoJSON 2:**

```json
{"type":"Point","coordinates":[-6.8498,34.0209]}
```

The node calculates the distance between the centroids of both inputs.

---

### Example: Create a Buffer

**Configuration:**

```text
Operation: buffer
Radius: 1
Units: kilometers
```

**GeoJSON:**

```json
{"type":"Point","coordinates":[-7.5898,33.5731]}
```

The output is a polygon feature representing the buffer area.

---

### Example: Simplify Geometry

**Configuration:**

```text
Operation: simplify
Tolerance: 0.01
```

**GeoJSON:**

```json
{"type":"LineString","coordinates":[[-7.60,33.57],[-7.595,33.575],[-7.59,33.58],[-7.585,33.585],[-7.58,33.59]]}
```

A higher tolerance produces a simpler geometry with fewer coordinates.

---

### Example: Generate a Circle

**Configuration:**

```text
Operation: circle
Radius: 2
Units: kilometers
```

**GeoJSON:**

```json
{"type":"Point","coordinates":[-7.5898,33.5731]}
```

The output is a polygon feature representing a circle centered on the input centroid.

---

### Example: Union

Use two overlapping polygon features.

**GeoJSON:**

```json
{"type":"Feature","properties":{"name":"Zone A"},"geometry":{"type":"Polygon","coordinates":[[[-7.60,33.57],[-7.58,33.57],[-7.58,33.59],[-7.60,33.59],[-7.60,33.57]]]}}
```

**GeoJSON 2:**

```json
{"type":"Feature","properties":{"name":"Zone B"},"geometry":{"type":"Polygon","coordinates":[[[-7.59,33.58],[-7.57,33.58],[-7.57,33.60],[-7.59,33.60],[-7.59,33.58]]]}}
```

**Configuration:**

```text
Operation: union
```

The output is a merged `Polygon` or `MultiPolygon` feature.

---

### Example: Intersection

Use the same two overlapping polygon features.

**Configuration:**

```text
Operation: intersection
```

The output is the overlapping area shared by both polygons.

If the polygons do not overlap, the result may be `null`.

---

### Example: Difference

Use the same two polygon features.

**Configuration:**

```text
Operation: difference
```

The output is the part of the first polygon that remains after subtracting the second polygon.

---

### Example: Point in Polygon

**GeoJSON:**

```json
{"type":"Feature","properties":{},"geometry":{"type":"Point","coordinates":[-7.59,33.58]}}
```

**GeoJSON 2:**

```json
{"type":"Feature","properties":{},"geometry":{"type":"Polygon","coordinates":[[[-7.60,33.57],[-7.58,33.57],[-7.58,33.59],[-7.60,33.59],[-7.60,33.57]]]}}
```

**Configuration:**

```text
Operation: boolean_point_in_polygon
```

**Output:**

```json
true
```

<!-- /SECTION: examples -->

---

<!-- SECTION: workflow-example -->
## Workflow Integration

### Example Workflow

```fusion-workflow
src: example.workflow.json
title: Calculate a GeoJSON bounding box
```

### Sample Workflow: Bounding Box Calculation

```json
{
  "nodes": [
    {
      "id": "trigger",
      "type": "manual-trigger"
    },
    {
      "id": "geojson",
      "type": "geojson",
      "config": {
        "operation": "bounding_box",
        "geojson": "{\"type\":\"Point\",\"coordinates\":[-7.5898,33.5731]}"
      }
    },
    {
      "id": "log",
      "type": "log"
    }
  ]
}
```

### Common Patterns

- **Validation:** Manual Trigger → GeoJSON Tools → Log
- **API Processing:** HTTP Request → GeoJSON Tools → Function
- **Geospatial Storage:** Webhook → GeoJSON Tools → Database
- **Polygon Analysis:** Load Geometry → GeoJSON Tools → Store Result
- **Location Validation:** Point Input → GeoJSON Tools → Conditional Node

<!-- /SECTION: workflow-example -->

---

<!-- SECTION: troubleshooting -->
## Troubleshooting

### Common Issues

#### ValidationAggregateError: Invalid string

**Cause:** The `geojson` or `geojson2` parameter received an object instead of a string.

**Solution:** Serialize the GeoJSON as a JSON string.

Correct:

```json
{"type":"Point","coordinates":[-7.5898,33.5731]}
```

When stored inside workflow JSON parameters:

```json
"{\"type\":\"Point\",\"coordinates\":[-7.5898,33.5731]}"
```

#### Invalid JSON in geojson

**Cause:** The provided string is not valid JSON.

**Solution:** Check quotes, commas, brackets, and braces.

#### Invalid JSON in geojson2

**Cause:** The secondary GeoJSON string is malformed.

**Solution:** Validate `geojson2` before execution.

#### Operation requires geojson2 input

**Cause:** A binary operation was selected without a secondary input.

**Solution:** Provide `geojson2` for:

- `distance`
- `union`
- `intersection`
- `difference`
- `boolean_point_in_polygon`

#### Cannot read properties of undefined (reading 'type')

**Cause:** A set operation received an unsupported or incomplete geometry, or one of the inputs was missing.

**Solution:** For `union`, `intersection`, and `difference`, provide two valid overlapping `Polygon` or `MultiPolygon` features.

#### Invalid coordinate order

**Cause:** Latitude and longitude were reversed.

**Solution:** Use:

```text
[longitude, latitude]
```

#### Union, intersection, or difference returns an error

**Cause:** One or both inputs are not polygon-based features.

**Solution:** Use valid `Polygon` or `MultiPolygon` GeoJSON features and ensure each polygon ring is closed.

A closed polygon ring repeats the first coordinate as its final coordinate.

### Error Codes

| Error | Cause | Solution |
|-------|-------|----------|
| `ValidationAggregateError` | Invalid parameter type | Pass GeoJSON as a string |
| `Invalid JSON in geojson` | Malformed primary input | Correct the JSON syntax |
| `Invalid JSON in geojson2` | Malformed secondary input | Correct the JSON syntax |
| `Operation requires geojson2 input` | Missing secondary geometry | Provide `geojson2` |
| `TypeError` | Unsupported or incomplete geometry | Verify geometry types and structure |
| `Unknown operation` | Unsupported operation value | Select a supported operation |

<!-- /SECTION: troubleshooting -->

---

<!-- SECTION: related -->
## Related

- [HTTP Request](./http-request.md) - Retrieve GeoJSON from an API
- [Function](./function.md) - Transform GeoJSON data
- [Log](./log.md) - Inspect GeoJSON operation results

<!-- /SECTION: related -->

---

<!-- SECTION: changelog -->
## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-06 | Initial release |

<!-- /SECTION: changelog -->